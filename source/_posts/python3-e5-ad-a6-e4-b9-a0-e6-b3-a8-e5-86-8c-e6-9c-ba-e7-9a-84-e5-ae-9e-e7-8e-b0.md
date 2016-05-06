---
title: python3学习——注册机的实现
tags:
  - keygen
  - python
  - rsa
  - 加密
id: 26
categories:
  - python
date: 2014-12-10 14:04:21
---

## 前言

这个注册机本来是用java写的的，但是在完成后收到了新的需求————添加gui。作为一个被java gui坑过一次的人，我绝不愿意再被坑第二次，而我手头又没装vs，所以便选择据说开发效率极高的python。
一边学一边做，开发过程虽说磕磕碰碰但是最后也算是顺利完成了，贴到网上，既是分享也是备忘。

整个项目主要用到了tkinter,pyDes,rsa,multiprocessing等库。tkinter用于实现gui。pyDes用于对key文件加密。multiprocessing用于实现多线程素性测试以提高rsaKey的生成效率。而最关键的则是rsa库了，但是事实上最后的版本中是没有引入rsa库的，原因是rsa库在打包过程中出现了问题，所以我参考rsa库和网上的一篇论文实现了一个简单myrsa库。

## 模块

### main函数

一个相当简陋的对话框带着四个同样简陋的按钮。
生成授权：接受用户输入的授权信息包括机器码，授权开始结束日期等
选择密钥：选择授权时所使用的key
创建密钥：随机生成rsaKey
导出公钥：导出rsa算法中的e和n，这两个参数需要写死在被保护的程序的验证模块中
[python]
if __name__ == '__main__':
    root = Tk()
    Button(root, text='生成授权', relief=RAISED, command=register).pack()
    Button(root, text='选择密钥', relief=RAISED, command=select_key).pack()
    Button(root, text='创建密钥', relief=RAISED, command=create_key).pack()
    Button(root, text='导出公钥', relief=RAISED, command=get_pubkey).pack()
    root.mainloop()
[/python]

### 创建密钥

首先接受用户输入的密钥位数和密钥别名，然后调用myrsa库中的newkeys生成密钥，myrsa会根据当前机器的cpu核心数量开启相应数量的线程进行计算。「此处没有对length最大最小值进行约束，存在安全隐患」
密钥生成完毕后会调用save_key进行保存。程序将在key文件下下根据用户输入的别名生成一个文件夹，然后用文件头部的_des_key对密钥对加密，最后保存为pub.key和pri.key。加入des算法是因为注册机与key文件分离，为了防止key文件流出导致第三方任意授权所以对key进行加密。加入des后必须有key文件保存时所用的_des_key否则无法读取key文件。
此外，程序还会向同目录下生成一个xml文件保存key的别名、位数和kid，其中kid可写入被保护程序，用于判断授权文件所使用的key。
[python]
def create_key():
    length = simpledialog.askinteger(root, &quot;请输入密钥长度&quot;)
    remark = simpledialog.askstring(root, &quot;输入密钥名称&quot;)
    (pubkey, prikey) = newkeys(length)
    config = configparser.ConfigParser()
    config.add_section('info')
    config.set('info', 'length', str(length))
    save_key(pubkey, prikey, config, remark)
    messagebox.showinfo(&quot;&quot;, &quot;密钥生成完毕&quot;)

def save_key(pubkey, prikey, config, remark):
    pub = pubkey.save_pkcs1()
    pri = prikey.save_pkcs1()
    encrypter = des(&quot;DESCRYPT&quot;, CBC, _des_key, pad=None, padmode=PAD_PKCS5)

    time_stamp = int(time.time() * 1000)
    if os.path.exists('key/' + remark):
        i = 1
        while os.path.exists('key/' + remark + '_' + str(i)):
            i += 1
        remark = remark + '_' + str(i)
    config.set('info', 'kid', str(time_stamp))
    e_pub = encrypter.encrypt(pub)
    e_pri = encrypter.encrypt(pri)

    file_prefix = 'key/' + remark
    os.makedirs(file_prefix)
    with open(file_prefix + '/info.ini', 'w') as f:
        config.write(f)
    with open(file_prefix + '/pub.key', 'wb') as f:
        f.write(e_pub)
    with open(file_prefix + '/pri.key', 'wb') as f:
        f.write(e_pri)
[/python]

### 选择密钥

之前的控制台版可以列出所有密钥进行选择，gui版本只做了根据用户输入寻找key文件夹然后写入配置文件。
[python]
def select_key():
    key_dir = filedialog.askdirectory()
    if None == key_dir:
        return
    config = configparser.ConfigParser()
    config.read('config.ini')
    if not config.has_section('info'):
        config.add_section('info')
    config.set('info', 'current_key', key_dir)
    with open('config.ini', 'w') as f:
        config.write(f)
[/python]

### 导出公钥

这里和上面的选择密钥原本在控制台版中都属于密钥管理的功能，为了适应gui做了弱化，会读取当前选择的key然后将kid,n,e写入指定的配置文件。
[python]
def get_pubkey():
    key_dir = filedialog.askdirectory()
    (pubkey, prikey, config) = load_key(key_dir)
    name = filedialog.asksaveasfilename(defaultextension='.ini', initialfile='pubkey',
                                        initialdir='', filetypes=[('公钥', '.ini')])
    pubkey_cfg = configparser.ConfigParser()
    pubkey_cfg.add_section('pubkey')
    pubkey_cfg.set('pubkey', 'kid', config.get('info', 'kid'))
    pubkey_cfg.set('pubkey', 'n', str(getattr(pubkey, 'n')))
    pubkey_cfg.set('pubkey', 'e', str(getattr(pubkey, 'e')))
    with open(name, 'w') as f:
        pubkey_cfg.write(f)
[/python]

### 生成授权

首先会读取配置文件获得当前的key，然后接受用户输入的授权信息并拼接成json串，最后调用_encrypt加密。加密完后会进行解密与原信息比对如果无法通过校验会提示密钥可能失效。
最后新建一个二进制文件在前8位写入kid，然后写入加密后的授权信息
[python]
def register():
    if not os.path.exists('config.ini'):
        select_key()
        print(1)
    config = configparser.ConfigParser()
    config.read('config.ini')
    try:
        current_key = config.get('info', 'current_key')
        print(current_key)
        load_key(current_key)
        print(3)
    except Exception:
        select_key()
        current_key = config.get('info', 'current_key')

    code = simpledialog.askstring(root, '请输入机器码')
    begin_str = simpledialog.askstring(root, '请输入授权开始日期(yyyy-MM-dd)')
    end_str = simpledialog.askstring(root, '请输入授权结束日期(yyyy-MM-dd)')
    count = simpledialog.askstring(root, '请输入客户端数量')
    name = filedialog.asksaveasfilename(defaultextension='.license', initialdir='', initialfile=str(int(time.time())),
                                        filetypes=[('授权文件', '.license')])
    reg_info = {}
    if not None == code:
        reg_info['code'] = code
    if not None == begin_str:
        begin_date = time.strptime(begin_str, &quot;%Y-%m-%d&quot;)
        reg_info['beginDate'] = int(time.mktime(begin_date) * 1000)
    if not None == end_str:
        end_date = time.strptime(end_str, &quot;%Y-%m-%d&quot;)
        reg_info['endDate'] = int(time.mktime(end_date) * 1000)
    if not None == count:
        reg_info['count'] = count
    json = JSONEncoder().encode(reg_info)

    (pubkey, prikey, config) = load_key(current_key)
    e = getattr(prikey, 'e')
    d = getattr(prikey, 'd')
    setattr(prikey, 'e', d)
    setattr(prikey, 'd', e)
    setattr(pubkey, 'e', d)
    e_json = _encrypt(json.encode(), pubkey)
    d_json = _decrypt(e_json, prikey)

    if not json.encode() == d_json:
        print(&quot;校验失败，密钥可能损坏。请确认license是否有效&quot;)

    with open(name, 'wb') as f:
        f.write(struct.pack('&gt;q', int(config.get('info', 'kid'))))
        f.write(e_json)
[/python]

### 验证部分

关于被保护的程序中的验证部分这里以java举例，需要将导出公钥中得到的e,d,kid以常量形式写入程序，如果对不同版本需要隔离只需要修改常量即可。
在验证部分会读取指定文件，然后读取前八位与常量kid比较如果不同则认为授权与软件版本不匹配，如果相同则利用常量n和e生成公钥对剩余部分进行解密。解密后得到授权信息开始对机器码等参数进行验证并返回结果。
[java]
private static final String modulus = &quot;***&quot;;
private static final String publicExponent = &quot;65537&quot;;
private static final long kid = ***L;

try (DataInputStream in = new DataInputStream(new BufferedInputStream(
	new FileInputStream(file)))) {

	Long licenseKid = in.readLong();
	if (kid != licenseKid) {
		return false;
	}
	byte[] license = new byte[(int) file.length() - 8];
	in.read(license);

	RSAPublicKey pubKey = (RSAPublicKey) KeyFactory.getInstance(&quot;RSA&quot;)
		.generatePublic(
			new RSAPublicKeySpec(new BigInteger(modulus),
				new BigInteger(publicExponent)));

	String licenseJsonString = RSAUtil.decryptByPublicKey(
		RSAUtil.bcd2Str(license), pubKey);
	JSONObject jsonObject = (JSONObject) new JSONParser()
		.parse(licenseJsonString);

	String code = (String) jsonObject.get(&quot;code&quot;);
	if (null != code &amp;&amp; !getSigarSequence().equals(code)) {
		return false;
	}
	Long beginDate = (Long) jsonObject.get(&quot;beginDate&quot;);
	if (null != beginDate &amp;&amp; beginDate &gt; new Date().getTime()) {
		return false;
	}
	Long endDate = (Long) jsonObject.get(&quot;endDate&quot;);
	if (null != endDate &amp;&amp; endDate &lt; new Date().getTime()) {
		return false;
	}
	Object o = jsonObject.get(&quot;count&quot;);
	Integer count = o instanceof String ? Integer.valueOf((String) o)
		: (Integer) o;
	if (null != count) {
		//TODO 设置客户端数量
	}
	return true;
} catch (Exception e) {
	e.printStackTrace();
	return false;
}
[/java]

## 总结

总的来说虽然在学习python方面花了不少时间，但是python的开发效率的确很高，对同类元素的高度整合使得python开发的过程中用于很少为了「哪一种方案更好」这种事而浪费时间。虽然执行效率偏低，但是这对于一些轻量级的工具应用而言，省下来的开发时间远比程序执行所花费的时间来的多，的确是开发各种小工具的首选。

另外，上面提到过rsa无法打包的问题，附件中已经用我自己实现的库代替了rsa，源代码下载后可以使用cxfreeze打包为exe文件。

[python3实现注册机源码](http://www.woodensail.tk/archives/26/reg)
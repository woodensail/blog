---
title: java8 lambda表达式体验
tags:
  - java
id: 124
categories:
  - java
date: 2015-04-16 21:16:25
---

java8在去年就发布了，其中包含了接口的默认实现，重复注解等许多新特性。其中，最令人关心的莫过于支持lambda表达式了。由于项目的历史问题，我没能在第一时间尝试java8所带来的lambda表达式。
今天群里面有人提到需要把一个数组中所有只出现了一次的字符串去除，剩下的输出。这个过程用js或是python都能够很容易的完成，并且代码非常简短，但是如果用java的话就会变得比较复杂。所以我尝试了用lambda来进行简化。
下面代码是首先将数组转化为stream后分组，然后通过values取得分组完毕的字符串。再转化为stream后通过filter筛去只出现过一次的字符串。此时得到的应该是一个双层的list，于是foreach两次对每个字符串使用println，打印输出。
[java]
String[] strings = {&quot;a&quot;, &quot;a&quot;, &quot;b&quot;, &quot;c&quot;, &quot;d&quot;, &quot;d&quot;, &quot;d&quot;};
Stream.of(strings).collect(Collectors.groupingBy((x) -&gt; x)).values().stream().filter((x) -&gt; x.size() != 1).forEach((x) -&gt; x.forEach(System.out::println));
[/java]
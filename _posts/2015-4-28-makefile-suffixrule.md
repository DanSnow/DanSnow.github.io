---
layout: post
title: "Makefile 的 suffix rule"
date: 2015-4-28 1:53
categories: program make
---

suffix rule 是 Makefile 的一種規則寫法  
似乎是挺老的一種寫法的 可是在一些比較舊的 make 版本上 新的萬用字元無法使用  
只能用 suffix rule 來簡化 Makefile 了

```make
.java.class: # Define how to make .class from .java
  javac $<

foo.class: foo.java # And you can simply write this to compile foo.class

target: foo.class # Or when you use foo.class, make will automatic compile foo.class from foo.java
```

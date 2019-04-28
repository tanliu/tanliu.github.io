---
layout: post
title: ClassLoader加载机制
date: 2019-03-19
---
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/9D629FB380FF9BB2833082B54DBB5E15.jpg)
1.类装载器
  - 启动类装载器
    - 装载核心lib
  - 扩展类装载器
    - jdk/lib/ext目录下
  - 系统类装载器
    - classpath
  - 自定义类装载器
  
2.装载器过程
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/10B3E8BEBD95BECCD1C5053E77EBBCFE.jpg)
- 验证：字节码等验证
- 准备：为变量分配内存，为类变量初始化(private static int size = 12的语句，初始化size=0)
- 解释：接口，方法，变更等解释
- 初始化（size=12）


3。双亲委派
- 把class委派给父亲。直到启动类装载器。验证是否是自己的职责
- 安全性：职责分明后。指定的类指定加载器加载。

4.类初始化的顺序



---
layout:     post
title:      "php的反射类"
subtitle:   "功能强大的反射类"
date:       2018-12-23 19:00:00
author:     "憧憬"
header-img: "img/post-bg-alibaba.jpg"
header-mask: 0.3
catalog:    true
tags:
  - php
---

## 反射

反射机制被各种语言广泛使用,主要是用来动态地获取系统中类,实例对象,方法等语言构建的信息,通过反射API函数可以实现对这些语言构建信息的动态获取和动态操作等.

反射API还提供了获取函数,类,和方法等语言构建中的文档注释,下面介绍一个实例

```
class Person {
 /**
  * For the sake of demonstration, we"re setting this private
  */
 private $_allowDynamicAttributes = false;

 /**
  * type=primary_autoincrement
  */
 protected $id = 0;

 /**
  * type=varchar length=255 null
  */
 protected $name;

 /**
  * type=text null
  */
 protected $biography;

 public function getId() {
  return $this->id;
 }

 public function setId($v) {
  $this->id = $v;
 }

 public function getName() {
  return $this->name;
 }

 public function setName($v) {
  $this->name = $v;
 }

 public function getBiography() {
  return $this->biography;
 }

 public function setBiography($v) {
  $this->biography = $v;
 }
}

// 参数是类的名称   
$person = new ReflectionClass('Person');

反射之后就能获取到该类的一些相关信息,例如:
常量 Contants
属性 Property Names
方法 Method Names
静态属性 Static Properties
命名空间 Namespace
Person类是否为final或者abstract


// 这里就相当于获取该类的实例对象
$class = $person->newInstanceArgs();

$class->setName('Test');

$class->getName();  // Test
```

依赖注入的实现,就是运用了反射API,可以简单说明一下

```
// 取类的构造函数，返回的是ReflectionMethod对象
$constructor = $reflector->getConstructor();

如果构造函数存在,返回一个ReflectionMethod对象,相当于获取构造函数的反射类,当构造函数不存在,会返回一个NULL
对于不存在的,我们可以直接去实例化,就不需要去解决构造函数中的依赖问题

// 取构造函数的参数，这是一个对象数组
$parameters = $constructor->getParameters();

然后去解决构造函数中依赖参数问题,进而实现依赖注入
```

依赖注入这个内容还是比较多,这里主要是做一个反射的记录,对于一些反射的使用,可以去查看一些框架的源代码
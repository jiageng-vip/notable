---
tags: [Oop]
title: 依赖倒置原则(DIP)
created: '2022-08-31T10:26:21.134Z'
modified: '2022-08-31T10:37:56.049Z'
---

# 依赖倒置原则(DIP)

依赖倒置简单地讲就是将依赖关系倒置为依赖接口，具体概念如下：
- 上层模块不应该依赖于下层模块，它们共同依赖于一个抽象（父类不能依赖子类，它们都要依赖抽象类）。
- 抽象不能依赖于具体，具体应该依赖于抽象。

为什么要依赖接口？因为接口体现对问题的抽象，同时由于抽象一般是相对稳定的或者是相对变化不频繁的，而具体是易变的。因此，依赖抽象是实现代码扩展和运行期内绑定（多态）的基础：只要实现了该抽象类的子类，都可以被类的使用者使用。
```
<?php
interface employee
{
    public function working();
}

class teacher implements employee
{
    public function working(){
        echo 'teaching...';
    }
}

class coder implements employee
{
    public function working(){
        echo 'coding...';
    }
}

class workA
{
    public function work(){
        $teacher = new teacher;
        $teacher->working();
    }
}

class workB
{
    private $e;
    public function set(employee $e){
        $this->e = $e;
    }
    
    public function work(){
        $this->e->working();
    }
}

$worka = new workA;
$worka->work();
$workb = new workB;
$workb->set(new teacher());
$workb->work();
```

在workA中，work方法依赖于teacher实现；在workB中，work转而依赖于抽象，这样可以把需要的对象通过参数传入。
在workB中，teacher实例通过setter方法传入，从而实现了工厂模式。由于这样的是实现是硬编码的，为了实现代码的进一步扩展，把这个依赖关系写在配置文件里，指明workB需要一个teacher对象，专门由一个程序检测配置是否正确（如所依赖的类文件是否存在）以及加载配置中所依赖的实现，这个检测程序，就称为IOC容器。
IOC（Inversion Of Control）是依赖倒置原则（Dependence Inversion Principle, DIP）的同义词。依赖注入（DI）和依赖查找（DS）是IOC的两种实现。

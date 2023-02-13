---
tags: [Oop]
title: 接口隔离原则(ISP)
created: '2022-08-31T10:17:19.866Z'
modified: '2022-08-31T10:35:39.880Z'
---

# 接口隔离原则(ISP)

接口隔离原则（Interface Segregation Principle,ISP）表明客户端不应该被强迫实现一些不会使用的接口，应该把胖接口分组，用多个接口代替它，每个接口服务于一个子模块。简单地说，就是使用多个专门的接口比使用单个接口要好很多。
ISP主要观点：
1. 一个类对另外一个类的依赖性应当是建立在最小接口上的。
ISP可以达到不强迫客户（接口使用者）依赖于他们不用的方法，接口的实现类应该只呈现为单一职责的角色（遵守SRP原则）。
ISP可以降低客户之间的相互影响——当某个客户程序要求提供新的职责（需求变化）而迫使接口发生变化时，影响到其他客户程序的可能性会最小。
2. 客户端程序不应该依赖它不需要的接口方法（功能）。

ISP强调的是接口对客户端的承诺越少越好，并且要做到专一。
接口污染就是为接口添加不必要的职责。`“接口隔离”其实就是定制化服务设计的原则`。使用接口的多重继承实现对不同的接口的组合，从而对外提供组合功能——达到“按需提供服务”。

对于接口的污染，使用下面两种处理方式：
- 利用委托分离接口。
- 利用多继承分离接口。

委托模式中，有两个对象参与处理同一个请求，接受请求的对象将请求委托给另一个对象来处理，如策略模式、代理模式等都应用到了委托的概念。

接口隔离原则(Interface  Segregation Principle, ISP)的概念：使用多个专门的接口，而不使用单一的总接口，即客户端不应该依赖那些它不需要的接口。

在使用接口隔离原则时，我们需要注意控制接口的粒度，接口不能太小，如果太小会导致系统中接口泛滥，不利于维护；接口也不能太大，太大的接口将违背接口隔离原则，灵活性较差，使用起来很不方便。一般而言，接口中仅包含为某一类用户定制的方法即可，不应该强迫客户依赖于那些它们不用的方法。

#### 举个例子来说吧：有一个跟动物有关的接口，代码如下：
```
<?php

interface Animal{

  public function walk();

  public function speak();

}
```

#### 狗是这个接口的一个具体实现：
```
<?php

require_once "animal.php";

class Dog implements Animal
{

    public function walk()
    {
        echo "dogs can walk";
    }

    public function speak()
    {
        echo "dogs can speak";
    }

}
```

#### ok，现在我们想创建一个鱼类，它会游泳，怎么办呢？我们必须要修改接口，还会影响到dog类的实现，而fish也需要实现walk和speak方法，如下代码所示：
##### Animal接口类：
```
<?php

interface Animal{

  public function walk();

  public function speak();

  public function swim();

}
```

##### dog类：
```
<?php

require_once "animal.php";

class Dog implements Animal
{

    public function walk()
    {
        echo "dogs can walk";
    }

    public function speak()
    {
        echo "dogs can speak";
    }

    public function swim()
    {
    }

}
```

##### fish类：
```
<?php

require_once "animal.php";

class Fish implements Animal
{

    public function walk()
    {
    }

    public function speak()
    {
    }

    public function swim()
    {
        echo "fish can swim";
    }

}
```

#### 这时Animal接口类就呈现出了”胖“接口的特征了。所谓胖接口其实就是接口中定义了不是所有实现类都需要的方法，就像Animal接口类，有些动物是不会游泳的，有些动物是不会行走的，还有些动物是不会飞的。如果将这些方法都写在一个Animal接口类中，那么后期的扩展和维护简直就是一场灾难。

##### 那么，怎么解决以上问题呢？

##### 很简单，接口细化即可，将Animal接口类拆分成三个接口类：

##### animalCanWalk接口类：
```
<?php

interface animalCanSpeak{
  public function speak();
}
```

##### AnimalCanSwim接口类：
```
<?php

interface AnimalCanSwim{
  public function swim();
}
```

##### animalCanSpeak接口类：
```
<?php

interface animalCanSpeak{
  public function speak();
}
```

#### 定义好这几个接口类之后，dog和fish的实现就容易多了，
```
<?php

require_once "animalCanSpeak.php";

require_once "animalCanWalk.php";

class Dog implements animalCanSpeak,animalCanWalk{
    public function walk(){
        echo "dogs can walk";
    }

    public function speak(){
        echo "dogs can speak";
    }

}

<?php

require_once "animalCanSwim.php";

class Fish implements AnimalCanSwim{
    public function swim(){
        echo "fish can swim";
    }

}
```

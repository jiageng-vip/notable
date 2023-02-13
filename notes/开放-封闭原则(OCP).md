---
tags: [Oop]
title: 开放-封闭原则(OCP)
created: '2022-08-31T09:39:18.902Z'
modified: '2022-08-31T10:07:39.391Z'
---

# 开放-封闭原则(OCP)

随着软件系统的规模不断增大，软件系统的维护和修改的复杂性不断提高，这种困境促使法国工程院士Bertrand Meyer在1998年提出了“开放-封闭”（Open-Close Principle, OCP）原则，基本思想是：
Open(Open for extension)模块的行为必须是开放的、支持扩展的，而不是僵化的。
Closed(Closed for modification)在对模块的功能进行扩展时，不应该影响或大规模地影响已有的程序模块。

换句话说，也就是要求开发人员在不修改系统中现有功能代码（源代码或二进制代码）的前提下，实现对应用系统的软件功能的扩展。用一句话概括就是：`一个模块在扩展性方面应该是开放的而在更改性方面应该是封闭的`。

开放-封闭能够提高系统的可扩展性和可维护性，但这也是相对的。

#### 以播放器为例，先定义一个抽象的接口：
```
interface Process
{
    public function process();
}
```

#### 然后对此接口进行扩展，实现解码和输出的功能：
```
class playerEncode implements Proess
{
    public function process(){
        echo "encode\r\n";
    }
}    

class playerOutput implements Process
{
    public function process(){
        echo "output\r\n";
    }
}
```
对于播放器的各种功能，这里是开放的。只要你遵守约定，实现了process接口，就能给播放器添加新的功能模块。

#### 接下来为定义播放器的线程调度管理器，播放器一旦接收到通知（可以是外部单击行为，也可以是内部的notify行为），将回调实际的线程处理
```
class playProcess
{
    private $message = null;
    public function __construct(){
    }
    
    public function callback(Event $event){
        $this->message = $event->click();
        if($this->message instanceof Process){
            $this->message->process();
        }
    }
}
```

#### 具体的产品出来了，在这里定义一个MP4类，这个类是相对封闭的，其中定义事件的处理逻辑：
```
class MP4
{
    public function work(){
        $playProcess = new playProcess();
        $playProcess->callback(new Event('encode'));
        $playProcess->callback(new Event('output'));
    }
}
```

#### 最后为事件分拣的处理类，此类负责对事件进行分拣，判断用户或内部行为，以产生正确的“线程”，供播放器内置的线程管理器调度：
```
class Event
{
    private $m;
    
    public function __construct($me){
        $this->m = $me;
    }
    
    public function click(){
        switch($this->m){
            case 'encode':
                return new playerEncode();
                break;
            case 'output':
                return new playerOutput();
                break;    
        }
    }
}
```

#### 运行：
```
$mp4 = new MP4;
$mp4->work();
//打印结果
encode
output
```

如何遵守开放-封闭原则
实现开放-封闭的核心思想就是抽象编程的核心思想就是`对抽象编程`，而不是对具体编程，因为`抽象相对稳定`。让类依赖于固定的抽象，这样的修改就是封闭的；而通过面向对象的`继承和多态机制`，可以实现对`抽象体`的继承，通过覆写其方法来改变固有的行为，实现新的扩展方法，所以对于扩展就是开放的。
1. 在设计方面充分应用“抽象”和封装的思想。
一方面就是要在软件系统中找出各种可能的“可变因素”，并将之封装起来；另一方面，一种可变性因素不应当散落在多个不同代码模块中，而应当被封装到一个对象中。
2. 在系统功能编程实现方面应用`面向接口`编程。
当需求发生变化时，可以提供该接口新的实现类，以求适应变化。
面向接口编程要求功能类实现接口，对象声明为接口类型。再设计模式中，装饰模式比较明显地用到


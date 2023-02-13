---
tags: [Oop]
title: 单一职责原则(SRP)
created: '2022-08-31T09:09:50.922Z'
modified: '2022-08-31T09:30:47.083Z'
---

# 单一职责原则(SRP)

### 单一职责原则 Single Pesponsibility Principle, SRP

在MVC框架中，对于表单插入数据库字段过滤与安全检查应该是放在control层处理还是model层处理，这类问题都可以归到单一职责的范围。

单一职责有两个含义：
1. 避免相同的职责分散到不同的类中
2. 一个类承担太多职责

遵守SRP的好处：
1. 减少类之间的耦合
2. 提高类的复用性

在实际代码开发中的应用：工厂模式、命令模式、代理模式等。
工厂模式（Factory）允许在代码执行时实例化对象。之所以被称为工厂模式是因为它负责“生产”对象。以数据库为例，工厂需要的就是根据不同的参数，生成不同的实例化对象。它只负责生产对象，而不负责对象的具体内容。

#### 定义一个适配器接口：
```
<?php
interface DbAdapter 
{
    /**
    * 数据库连接
    * @param $config 数据库配置
    * @return resource
    */
    public function connect($config);
    
    /**
    * 执行数据库查询
    * @param string $query 数据库查询SQL字符串
    * @param mixed $handle 连接对象
    * @return resource
    */
    public function query($query, $handle);
}
?>

```

#### 定义MySQL数据库操作类：
```
<?php
class DbAdapterMysql implements DbAdapter
{
    private $_dbLink; //数据库连接字符串表示

    /**
    * 数据库连接函数
    * @param $config 数据库配置
    * @throws DbException
    * @return resource
    */
    public function connect($config)
    {
        if($this->_dbLink = @mysql_connect($config->host .
            (empty($config->port) ? '' : ':' . $config->port),
        $config->user, $config->password, true)) {
            if(@mysql_select_db($config->database, $this->_dbLink)){
                if($config->charset){
                    mysql_query("SET NAMES '{$config->charset}'", $this->_dbLink);
                }
                return $this->_dbLink;
            }
        }
        //数据库异常
        throw new DbException(@mysql_error($this->_dbLink));
    }
    
    /**
    * 执行数据库查询
    * @param string $query 数据库查询SQL字符串
    * @param mixed $handle 连接对象
    * @return resource
    */
    public function query($query, $handle)
    {
        if ($resource = @mysql_query($query, $handle)) {
            return $resource;
        }
    }
}
?>
```

#### SQLite数据库操作类：
```
<?php
class DbAdapterSqlite implements DbAdapter
{
    private $_dbLink;
    
    /**
    * 数据库连接函数
    * @param $config 数据库配置
    * @throws DbException
    * @return resource
    */
    public function connect($config)
    {
        if ($this->_dbLink = sqlite_open($config->file, 0666, $error)) {
            return $this->_dbLink;
        }
        
        throw new DbException($error);
    }
    
    /**
    * 执行数据库查询
    * @param string $query 数据库查询SQL字符串
    * @param mixed $handle 连接对象
    * @return resource
    */
    public function query($query, $handle)
    {
        if ($resource = @sqlite_query($query, $handle)) {
            return $resource;
        }
    }
}
?>
```

#### 定义一个工厂类，根据传入不同的参数生成需要的类：
```
<?php
    class sqlFactory
    {
        public static function factory($type)
        {
            if (include_once 'Drivers/' . $type . '.php') {
                $classname = 'DbAdapter' . $type;
                return new $classname;
            } else {
                throw new Exception('Driver not found');
            }
        }
    }
?>
```

#### 调用：
```
$db = sqlFactory::factory('MySQL');
$db = sqlFactory::factory('SQLite');

```

#### 命令模式分离“命令的请求者”和“命令的实现者”方面的职责。
```
<?php
/**
* 厨师类，命令接受者与执行者
*/
class cook 
{
    public function meal(){
        echo '番茄炒鸡蛋',PHP_EOL;
    }
    
    public function drink(){
        echo '紫菜蛋花汤',PHP_EOL;
    }
    
    public function ok(){
        echo '完毕',PHP_EOL;
    }
}

/**
* 命令接口
*/
interface Command{
    public function execute();
}
?>
```

#### 模拟服务员与厨师的过程：
```
<?php
class MealCommand implements Command
{
    private $cook;
    
    //绑定命令接受者
    public function __construct(cook $cook){
        $this->cook = $cook;
    }
    
    public function execute(){
        $this->cook->meal();//把消息传递给厨师，让厨师做饭
    }
}

class DrinkCommand implements Command
{
    private $cook;
    
    //绑定命令接受者
    public function __construct(cook $cook){
        $this->cook = $cook;
    }
    
    public function execute(){
        $this->cook->drink();
    }
}
?>
```

#### 模拟顾客与服务员的过程：
```
<?php
class cookControl
{
    private $mealCommand;
    private $drinkCommand;
    
    //将命令发送者绑定到命令接收器
    public function addCommand(Command $mealCommand, Command $drinkCommand){
        $this->mealCommand = $mealCommand;
        $this->drinkCommand = $drinkCommand;
    }
    
    public function callMeal(){
        $this->mealCommand->execute();
    }
    
    public function callDrink(){
        $this->drinkCommand->execute();
    }
}    
?>
```

#### 实现命令模式
```
$control = new cookControl;
$cook = new cook;
$mealCommand = new MealCommand($cook);
$drinkCommand = new DrinkCommand($cook);
$control->addCommand($mealCommand, $drinkCommand);
$control->callMeal();
$control->callDrink();
```

---
tags: [Laravel]
title: 基于Composer实现类和文件的自动加载
created: '2022-11-04T08:26:36.226Z'
modified: '2022-11-04T09:09:57.182Z'
---

# 基于Composer实现类和文件的自动加载

##### 以 Lumen 项目为例，在 bootstrap/app.php 中可以看到应用在一开始就引入了类加载器：
```
require_once __DIR__ . '/../vendor/autoload.php';
```

##### 对应的 `vendor/autoload.php` 文件由 Composer 在初始化的时候生成，代码很简单：
```
require_once __DIR__ . '/composer/autoload_real.php';

return ComposerAutoloaderInit0e76a85fe5139db13e153d53e25d4523::getLoader();
```
> 注：`ComposerAutoloaderInitXXX` 中的 `XXX` 表示用于标识不同项目的唯一标识符，比如我这里是 `0e76a85fe5139db13e153d53e25d4523`，为了方便描述，我们后续都用 XXX 来指代它。

##### 接下来我们来重点关注 vendor/composer/autoload_real.php 文件中 ComposerAutoloaderInitXXX::getLoader() 的实现代码：
```
    /**
     * @return \Composer\Autoload\ClassLoader
     */
    public static function getLoader()
    {
        if (null !== self::$loader) {
            return self::$loader;
        }

        require __DIR__ . '/platform_check.php';

        spl_autoload_register(array('ComposerAutoloaderInit0e76a85fe5139db13e153d53e25d4523', 'loadClassLoader'), true, true);
        self::$loader = $loader = new \Composer\Autoload\ClassLoader();
        spl_autoload_unregister(array('ComposerAutoloaderInit0e76a85fe5139db13e153d53e25d4523', 'loadClassLoader'));

        $useStaticLoader = PHP_VERSION_ID >= 50600 && !defined('HHVM_VERSION') && (!function_exists('zend_loader_file_encoded') || !zend_loader_file_encoded());
        if ($useStaticLoader) {
            require __DIR__ . '/autoload_static.php';

            call_user_func(\Composer\Autoload\ComposerStaticInit0e76a85fe5139db13e153d53e25d4523::getInitializer($loader));
        } else {
            $map = require __DIR__ . '/autoload_namespaces.php';
            foreach ($map as $namespace => $path) {
                $loader->set($namespace, $path);
            }

            $map = require __DIR__ . '/autoload_psr4.php';
            foreach ($map as $namespace => $path) {
                $loader->setPsr4($namespace, $path);
            }

            $classMap = require __DIR__ . '/autoload_classmap.php';
            if ($classMap) {
                $loader->addClassMap($classMap);
            }
        }

        $loader->register(true);

        if ($useStaticLoader) {
            $includeFiles = Composer\Autoload\ComposerStaticInit0e76a85fe5139db13e153d53e25d4523::$files;
        } else {
            $includeFiles = require __DIR__ . '/autoload_files.php';
        }
        foreach ($includeFiles as $fileIdentifier => $file) {
            composerRequire0e76a85fe5139db13e153d53e25d4523($fileIdentifier, $file);
        }

        return $loader;
    }
```

###### 在 `ComposerAutoloaderInitXXX` 类中，使用了一个私有的静态属性 $loader 用于持有 `Composer\Autoload\ClassLoader` 实例，在一次 HTTP 请求处理过程中，如果类加载器已经初始化过，则直接返回 $loader，以提高应用性能。否则的话才继续往下走。

###### 接下来，通过 PHP 内置函数 spl_autoload_register 注册类加载器，当应用代码中遇到未定义类时会通过 ComposerAutoloaderInitXXX 的 loadClassLoader 方法加载该类，然后我们看下一行代码：
```
self::$loader = $loader = new \Composer\Autoload\ClassLoader();
```
###### 这里会对静态属性 $loader 进行赋值，对应的值是 `\Composer\Autoload\ClassLoader` 类实例，这里就需要用到我们上一行注册的类加载器对 `\Composer\Autoload\ClassLoader` 类进行加载，通过查看源码我们可以得出 Composer 会从 `vendor/composer/ClassLoader.php` 中查找该类的结论。

###### 紧接着在 ComposerAutoloaderInitXXX::getLoader() 方法中会通过 spl_autoload_unregister 函数注销之前注册的类加载器。

###### 再往后就是 Composer 类加载机制的核心逻辑了，从 5.6 版本开始 PHP 支持静态初始化，所以做了区别处理，对于满足静态初始化条件的，引入 `vendor/composer/autoload_static.php` 文件，并调用 `\Composer\Autoload\ComposerStaticInitXXX::getInitializer($loader)` 方法设置 `$loader` 实例的 `prefixLengthsPsr4`、`prefixDirsPsr4`、`prefixesPsr0`、`classMap` 属性。否则的话，引入 `vendor/composer/autoload_namespaces.php` 文件，调用 `$loader->set($namespace, $path);` 方法设置 `prefixesPsr0` 属性，然后引入 `vendor/composer/autoload_namespaces.php` 文件，调用 `$loader->setPsr4($namespace, $path);` 方法设置 `prefixLengthsPsr4` 和 `prefixDirsPsr4` 属性，最后引入 `vendor/composer/autoload_classmap.php` 文件，调用 `$loader->addClassMap($classMap);` 方法设置 classMap 属性。

###### 上述条件分支语句处理的最终结果是一致的，都是完成 `$loader` 实例 `prefixLengthsPsr4`、`prefixDirsPsr4`、`prefixesPsr0`、`classMap` 属性的初始化，其中 `classMap` 管理的是完整类名（可能包含命名空间）与文件路径的映射关系、`prefixDirsPsr4` 管理的是命名空间与文件目录的映射关系（遵循 psr-4 规范）、`prefixLengthsPsr4` 管理的是命名空间前缀长度、`prefixesPsr0` 管理也是命名空间与文件目录映射关系（遵循 psr-0 规范，极少数历史包在使用，目前 psr-0 规范已废弃）。这些映射关系都是运行 Composer 安装或更新命令时，系统自动帮我们维护的（从每个依赖包和项目根目录下的 comspoer.json 读取 autoload 配置实现），既包含位于 vendor 目录下依赖包的类和命名空间，也包含项目根目录下用户自定义的类和命名空间，对于用户自定义的命名空间，需要在 composer.json 的 autoload 配置项中显式配置，才能被系统识别
```
"autoload": {
    "psr-4": {
        "App\\": "app/"
    },
    "classmap": [
        "database/seeds",
        "database/factories"
    ]
},
```

###### 接下来，调用 $loader->register(true); 完成 Composer 包管理器类自动加载注册：
```
public function register($prepend = false)
{
    spl_autoload_register(array($this, 'loadClass'), true, $prepend);
}
```

###### 还是调用 `spl_autoload_register` 来完成，这是这次类自动加载处理函数变成了 `$loader` 实例的 `loadClass` 方法，当 Laravel 应用执行过程中遇到未定义类时，会调用该方法，然后通过传入的类名调用 `$loader->findFile($class)` 方法查找对应类所在文件路径并引入：
```
    /**
     * Finds the path to the file where the class is defined.
     *
     * @param string $class The name of the class
     *
     * @return string|false The path if found, false otherwise
     */
    public function findFile($class)
    {
        // class map lookup
        if (isset($this->classMap[$class])) {
            return $this->classMap[$class];
        }
        if ($this->classMapAuthoritative || isset($this->missingClasses[$class])) {
            return false;
        }
        if (null !== $this->apcuPrefix) {
            $file = apcu_fetch($this->apcuPrefix.$class, $hit);
            if ($hit) {
                return $file;
            }
        }

        $file = $this->findFileWithExtension($class, '.php');

        // Search for Hack files if we are running on HHVM
        if (false === $file && defined('HHVM_VERSION')) {
            $file = $this->findFileWithExtension($class, '.hh');
        }

        if (null !== $this->apcuPrefix) {
            apcu_add($this->apcuPrefix.$class, $file);
        }

        if (false === $file) {
            // Remember that this class does not exist.
            $this->missingClasses[$class] = true;
        }

        return $file;
    }
```

###### 如果 `$loader->classMap` 属性中已经包含该类则直接返回对应文件路径，否则继续往后执行，如果该类别标识为缺失则不再往下执行，如果启用了 apcu 扩展，则先从对应缓存中获取文件路径，接下来开始常规查找，调用 `$this->findFileWithExtension($class, '.php');` 方法先通过 PSR-4 规范查找类对应文件路径，如果找到则返回，否则再通过 PSR-0 规范查找，这其中会用到 `$loader` 初始化时设置的 `prefixLengthsPsr4`、`prefixDirsPsr4`、`prefixesPsr0` 属性来组合出查找条件。如果最终没有找到，则返回 false。回到 findFile 方法，如果返回值为 false，并且定义了 `HHVM_VERSION` 常量，则将文件后缀改为 `.hh` 再找一次。如果最后文件路径没有找到，则将其添加到 `$loader->missingClasses` 属性中，如果启用了 apcu 扩展，不管找没找到，都将结果保存到 apcu 缓存。

###### 如果类对应的文件路径最终没有找到，会抛出异常（`spl_autoload_register` 方法第二个参数为 `true` 表示未找到类时抛出异常）。

###### 下面我们回到 `ComposerAutoloaderInitXXX::getLoader()` 方法，继续往下面看，在 Lumen 项目中，除了类之外，还支持不归属于任何类的辅助函数，这些辅助函数通常定义在 `helpers.php` 文件中，Composer 通用支持对这类文件的自动加载，这一块的处理通样针对是否支持静态初始化进行了区分，如果支持静态初始化，则通过 `Composer\Autoload\ComposerStaticInitXXX::$files`; 返回自动加载的文件数组，否则的话引入 `vendor/composer/autoload_files.php` 返回需要自动加载的文件数组。最后遍历这些文件逐个引入，并将它们的标识符存放到全局变量 `__composer_autoload_files` 中。

###### 同样这些自动加载的文件数组由 Composer 帮我们维护，对于通过 Composer 安装的依赖包，Composer 会读取每个依赖包的 `composer.json` 中的 `autoload.files` 配置项获取自动加载文件，并维护到 `vendor/composer/autoload_files.php` 和 `vendor/composer/autoload_static.php` 数组中，对于项目根目录下用户自定义的 PHP 文件，需要在 `composer.json` 的 autoload 中维护到 files 数组配置项：
```
"autoload": {
    ...
    "files": [
        "app/helpers.php"
    ]
```

###### 对于通过 Composer 安装的依赖包，执行安装命令时 Composer 会自动维护命名空间、类和文件的映射，对于在项目根目录下由开发者自定义的命名空间、类和文件，需要在新增后手动运行 `composer dump-auto` 命令将其更新到 `vendor/composer` 目录下相应的自动加载配置中，否则系统将无法找到对应的类和文件。

##### 总结下来，Composer 支持四种自动加载方式，对应在 `composer.json` 中，即 `autoload` 配置项中的 `classmap`、`psr-4`、`psr-0` 和 `files` 四个配置，前三个都是维护类的自动加载，最后一个维护的是文件的自动加载，其中 classmap 通常用来管理不归属于命名空间的类，比如 databases 目录下的类，在配置的时候只需要配置目录路径即可，而 psr-4 通常用来管理归属于命名空间的类，我们在配置的时候只需配置根命名空间与对应目录的映射即可，psr-0 已经废弃，很少用到，对应 files，我们需要配置完整的文件路径。
```
"autoload": {
    "psr-4": {
        "src\\darren\\": "src/",
        "project\\darren\\": "project"
        },
    "files": ["common/Darren.php", "common/Since.php"],
    "classmamp": [lib]
}
```

###### 测试代码目录结构如下：
```
common
    Darren.php
    Since.php
lib
    Darren.php
    Since.php
project
    Darren.php
src
    Darren.php
vendor
composer.json
index.php
```

###### 代码中的命名空间习惯为：`目录名/Darren`当我们配置好 composer.json 文件，并执行 `compoer install` 命令后，在 `vendor/composer` 目录下会自动生成一些 php 文件，这些文件实际上记录了类、命名空间和对应的类文件的映射，下面一一举例说明；

###### PSR-4
###### 如上所述，通过 psr-4 方式配置了两个命名空间的自动加载，分别是 src\daren 和 projecr\darren；vendor/composer 目录下自动生成了 autoload_psr4.php 文件，具体代码如下所示：
```
<?php

// autoload_psr4.php @generated by Composer

$vendorDir = dirname(dirname(__FILE__));
$baseDir = dirname($vendorDir);

return array(
'src\\darren\\' => array($baseDir . '/src'),
'project\\darren\\' => array($baseDir . '/project'),
);
```

###### Classmap 方式
###### classmap 方式只需要我们配置需要自动加载的目录，compoer 会自动扫描目录下的的 .php 文件或 .inc 文件中的 class，并自动生成这些类和其对应的类文件的映射关系，保存在 vendor/composer 目录下的 autoload_classmap.php 文件中，具体代码如下：
```
<?php

// autoload_classmap.php @generated by Composer

$vendorDir = dirname(dirname(__FILE__));
$baseDir = dirname($vendorDir);

return array(
'lib\\darren\\Darren' => $baseDir . '/lib/Darren.php',
'lib\\since\\Since' => $baseDir . '/lib/Since.php',
);
```
###### 其中 lib\darren 为命名空间，Darren 为类名；

###### Files
###### files 方式其实就是手动指定要加载的文件，这通常适用于一些全局的 functions，可以将这些 functions 统一放在一个文件里，然后直接进行加载；
###### 上述的配置文件通过 files 方式加载了两个文件 common/Darren.php 和 common/Since.php，vendor/composer 目录下自动生成了 autoload_files.php 文件，具体代码如下所示：
```
<?php

// autoload_files.php @generated by Composer

$vendorDir = dirname(dirname(__FILE__));
$baseDir = dirname($vendorDir);

return array(
'b704865b506bf33e8097e6f62604fc7f' => $baseDir . '/common/Darren.php',
'603921ee67f9053beb44a88f05b115d2' => $baseDir . '/common/Since.php',
);
```



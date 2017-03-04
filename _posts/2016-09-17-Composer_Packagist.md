---
layout: blog
title: 如何创建一个自己的 Composer/Packagist 包
category: blogs
---

让我们先去Github创建一个新库，这里我取名`xunsearch-laravel`，欢快的将它克隆到本地：

```
git clone git@github.com:ShaoZeMing/xunsearch-laravel.git
cd xunsearch-laravel
```

这个`xunsearch-laravel`文件夹就是你的包的根目录了，你只需要记住composer.json在包的哪个目录下面，一般那就是包的根目录了。

不过github给了一些命令行建议，以后提交github项目的时候需要使用这些命令：

```
…or create a new repository on the command line
echo "# xunsearch-laravel" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/ShaoZeMing/xunsearch-laravel.git
git push -u origin master
…or push an existing repository from the command line
git remote add origin https://github.com/ShaoZeMing/xunsearch-laravel.git
git push -u origin master
…or import code from another repository
You can initialize this repository with code from a Subversion, Mercurial, or TFS project.
```

由于我们创建的是一个全新的库，还没有composer.json文件，你可以根据composer文档生成并编辑它，当然composer贴心的为我们准备了初始化命令：

```
composer init
```
![1452819710967277.png][1]


什么？你还没有在packagist.org上注册账号？赶紧访问https://packagist.org/，使用你的github账号授权登陆或者直接用邮箱注册一个用户名。

例如我的packagist的主页地址为：<https://packagist.org/packages/shaozeming/xunsearch-laravel>

那么我在packagist的用户名就是shaozeming，以后发布composer包的时候就是shaozeming/xxxx

```
OK，我们这里输入：shaozeming/xunsearch-laravel回车

Description []:   （这里填包的描述，可以不填直接回车）

Author [shaozeming <szm19920426@gmail.com>]: （如果你在shell下登陆过git应该有保存你的邮箱名，直接回车即可，如果没有按照上面的格式自己补充）

Minimum Stability []: （最低要求，该参数至关重要，建议填写dev可以直接将github上代码check到packagist， master、RC等版本类型我还没尝试过）

Package Type []: （大概是填镜像信息，可不填直接回车以后编辑composer.json）

License []: （授权协议，可不填直接回车）

Define your dependencies.
Would you like to define your dependencies (require) interactively [yes]? （填no直接回车）

Would you like to define your dev dependencies (require-dev) interactively [yes]? （填no直接回车）

Do you confirm generation [yes]? （填yes直接回车）

Would you like the vendor directory added to your .gitignore [yes]? （填yes直接回车，事实上github上很多项目都不会提交vendor目录）
```

由于我们在交互界面大部分都是输入no导致composer.json信息量较少，我们将其改为：

```
{
    "name": "shaozeming/xunsearch-laravel",
    "description": "基于XunSearch（讯搜）sdk的全文搜索Laravel 5.*包，支持全拼，拼音简写，模糊,同义词搜索。",
    "license": "MIT",
    "authors": [
        {
            "name": "shaozeming",
            "email": "szm19920426@gmail.com",
            "homepage": "http://blog.4d4k.com"
        }
    ],
    "minimum-stability": "dev",
    "require": {
        "php": ">=5.3"
    },
    "autoload": {
        "psr-4": {
            "shaozeming\\xunsearch\\": "src/wrapper"
        },
        "classmap": ["src/lib/"]
    }
}

```

好了，终于生成了一份composer.json文件，如果不喜欢命令行可以保留一份以后直接改改放到别的项目就行。

根据上面的命名空间和目录的映射关系，包的结构现在应该是下面这个样子：

```
xunsearch-laravel
- src
- - app
- - - demo.ini
- - lib
- - - XS.php
- - util
- - - Indexer.php
- - - IniWizzard.php
- - - Logger.php
- - - Quest.php
- - - RequiredCheck.php
- - - SearchSkel.php
- - - xs
- - - XSDataSource.class.php
- - - XSUtil.class.php
- - wrapper
- - - Search.php
- .gitignore
- composer.json
- README.md

```

下面给出这个Search.php文件的部分代码内容

```
namespace shaozeming\xunsearch;

/**
 * 封装所有功能类
 *
 * @author szm19920426@gmail.com
 * @link http://www.4d4k.com/
 * @version $Id$
 */
class Search extends \XS
{
    protected $config = [
        'flushIndex'     => true,       //立即刷新索引
        'setFuzzy'       => true,       //开启模糊搜索
        'autoSynonyms'   => true,       //开启自动同义词搜索功能
    ];

    public function __construct($file, array $config = [])
    {
        parent::__construct($file);

        if(!empty($config)){
            $this->config = array_merge($this->config,$config);
        }
    }

    /**
     * 添加索引数据
     *
     * @author szm19920426@gmail.com
     * $data array  一维||二维
     * @return mixed
     */
    public function addIndex(array $data)
    {
        if (!array($data)) {
            die('参数错误！');
        }
        if (count($data) == count($data, 1)) {
            // 一维数组
            $this->getIndex()->add(new \XSDocument($data));
        } else {
            // 多维数组
            foreach ($data as $v) {
                $this->getIndex()->add(new \XSDocument($v));
            }
        }

        //索引是否立即生效
        if ($this->config['flushIndex']) {
            $this->getIndex()->flushIndex();
        }

        return $this->getIndex();
    }
}

```

然后执行命令：

```
composer install
```

接下来我们通过composer install生成映射关系，之后将会在项目目录下自动生成vendor目录，并在vendor下自动生成了composer包autoload程序文件。这时，在vendor目录根本找不到我们之前上传的四个php文件，这是为什么呢？因为`verndor/composer/autoload_psr4.php`中记录了`$vendorDir`和`$baseDir`的路径，packagist会自动将你的项目文件放到vendor目录下：

![QQ截图20170221234216.jpg][2]

所以我们需要将xunsearch-laravel项目提交到github上，好让packagist是通过github来自动完成检出操作。

如果不知道怎么在github上提交项目可以看看之前创建项目时github给了一些命令行建议。我的执行命令如下：

![1452820070533464.png][3]

好了，现在访问你的github项目的网址，你会发现刚才增加的文件都会出现在github上，但是没有vendor目录，只有一个src/Ford目录

接下来访问：https://packagist.org/packages/submit


在输入框中填入你在github上的项目地址：<https://github.com/ShaoZeMing/xunsearch-laravel.git>
建议填入git项目的路径：<git@github.com:ShaoZeMing/xunsearch-laravel.git
这时packagist可能会提示你有一些别人的项目和你的同名，直接忽略点击“Submit”按钮。

稍等几秒，你在packagist上的包就发布成功啦：https://packagist.org/packages/shaozeming/xunsearch-laravel

这里有四个颜色的按钮对这个包进行操作：

Abandon：点击这个后提示使用者自己不再维护了

Delete：删除此项目

Update：从git更新到packagist上

Edit：修改git项目的地址

如果你修改了项目的代码并push到github之后，packagist是不知道项目已经更新了，你需要点击上面那个“update”的按钮完成更新操作，但是这样很麻烦，于是github和packagist合作，允许使用Service完成自动更新，你需要到github上对这个项目进行设置——Settings里Add Service。

选择左侧的`Webhooks & service`，然后点击右侧`Add Service`按钮，然后选择下拉选项中的`Packagist`，这时需要你输入github的密码继续操作。

接着你需要填入以下3条信息，Token信息全部可以从<https://packagist.org/profile/>获取——Show API Token

User

填你在packagist的用户名：ShaoZeMing

Token

填Show API Token之后的key：****************

Domain

填`http://packagist.org`

填完提交之后，在Services下新增了Packagist的自动更新服务，但是Packagist左侧的小圆点是灰色的，并且提示

`This hook has never been triggered`
这时你需要对github上的代码进行修改测试一下是否能够完成自动更新。

我们随便给改动一下README.md文件，再次提交到github。


神奇的事情发生了，在`Webhooks & service`的右侧Packagist的小圆圈变为绿色的钩：

`last delivery was successful`

还有更强大的功能，Webhooks & service上方的Webhooks可以自定义接收网址然后github在每次更新时自动post数据到你的网站，这里就不演示了。

现在我们尝试使用composer install来安装这个项目，然后在新的项目目录中新建一个这样的composer.json文件，注意此composer.json文件并非之前的那个composer.json文件。

```
{
    "require": {
        "php": ">=5.3.0",
        "shaozeming/xunsearch-laravel": "dev-master"
    },
    "repositories": {
      "packagist": {
         "type": "composer",
         "url": "http://packagist.org"
      }
   }
}
```

然后使用命令进入到该目录下执行composer install命令。


OK，一切都很顺利，成功使用composer安装了刚才发布的包。

可是，你知道为什么一定要带上repositories中的packagist镜像网址吗？

因为使用<http://packagist.org>镜像源的时候国内被墙，使用国内的镜像源会报不存在这个包，报错如下：

```
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - The requested package shuiguang/composer-car could not be found in any version, there may be a typo in the package name.

Potential causes:
 - A typo in the package name
 - The package is not available in a stable-enough version according to your minimum-stability setting
   see <https://groups.google.com/d/topic/composer-dev/_g3ASeIFlrc/discussion> for more details.

Read <https://getcomposer.org/doc/articles/troubleshooting.md> for further common problems.
[root@iZu116o2qrjZ composer-car]# composer install
Loading composer repositories with package information
Installing dependencies (including require-dev)
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - The requested package "shaozeming/xunsearch-laravel": "dev-master" could not be found in any version, there may be a typo in the package name.

Potential causes:
 - A typo in the package name
 - The package is not available in a stable-enough version according to your minimum-stability setting
   see <https://groups.google.com/d/topic/composer-dev/_g3ASeIFlrc/discussion> for more details.

```
所以建议使用国外的服务器做测试，如果没有国外的服务器，可以等到第二天尝试一下国内的全量镜像源：http://packagist.phpcomposer.com

一般来说，国内的镜像站不是很及时，刚发布的包不可能立刻同步，但是packagist.org官方镜像肯定是存在这个包的。接着我们在/xunsearch-laravel目录下新建一个show.php，代码如下：
```
<?php

require __DIR__ .'/vendor/autoload.php';

use wrapper\Search;

$s = new Search('demo')
var_dump($s);
```

然后在php命令行下执行这个show.php:

OK，composer自动加载类就是这么简单。



虽然运行没问题，但是建议不要提交show.php文件到github上，我们得写一个自动化测试套件之后发布到包里面，让使用者直接通过phpunit命令进行测试。

如果不知道phpunit怎么用可以先去学习一下，这里不讲。

在项目的根目录下首先建立一个tests的目录，然后编写bootstrap.php启动测试的文件：

```
<?php
// Enable Composer autoloader
/** @var \Composer\Autoload\ClassLoader $autoloader */
$autoloader = require dirname(__DIR__) . '/vendor/autoload.php';
// Register test classes
$autoloader->addPsr4('xunsearch-laravel\tests\\', __DIR__);
```

很明显，该操作的意图是将xunseatch-laravel/tests目录注册到需要autoload的目录。

建议在项目根目录下添加一个phpunit.xml文件，配置内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="tests/bootstrap.php">
</phpunit>
```

使用composer安装的包的tests目录和phpunit.xml并不会放到项目的根目录下，这时如果和git一起使用，我们就能直接在项目的根目录下执行phpunit tests命令直接测试了。

composer有一点不是很喜欢，可能是我对composer的原理不熟的缘故，例如我在一个未安装过的composer项目的目录中直接使用

```
composer require shaozeming/xunsearch-laravel 'dev-master'
```


也不会报错，可是根本没法将我的包下载完全，总是出现下面内容：

```
Using version ^x.x for shaozeming/xunsearch-laravel
Loading composer repositories with package information
Updating dependencies (including require-dev)
Nothing to install or update
Writing lock file
Generating autoload files
```
但是如果我手动编辑composer.json之后，然后执行`composer update`则可以下载。

我个人总结的经验是：如果你只使用了一个包，建议手动编辑composer.json文件然后执行`composer update`安装；如果使用了多个包，除第一个以外的包可以使用`composer require`安装



最后，我们测试一个github发行版，给我们的项目加一个版本号v1.0：

访问github的项目发行版网址：<https://github.com/ShaoZeMing/xunsearch-laravel>

我们创建一个releases项目：

```
Tag version：v1.0.0

Release title：composer-car-1.0.0-stable

Write：test
```

然后点击“Publish release”添加一个v1.0的版本。

接着访问packagist，更神奇的是刚才添加的v1.0版本已经被自动检出了。

然后将之前的

```
"shaozeming/xunsearch-laravel": "dev-master"
```

改为

```
"shaozeming/xunsearch-laravel": "^1.0"
```

这时运行`composer update`便可以从dev版升级到v1.0.0版本。


  [1]: http://blog.4d4k.com/usr/uploads/2017/02/3057530305.png
  [2]: http://blog.4d4k.com/usr/uploads/2017/02/2898084560.jpg
  [3]: http://blog.4d4k.com/usr/uploads/2017/02/4022595870.png
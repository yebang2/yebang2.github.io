## **问题说明**

我们经常要往现有的项目中添加扩展包，有时候因为文档的错误引导，如下图来自 这个文档 的：
![file](https://lccdn.phphub.org/uploads/images/201806/07/23400/1mL3SYSkzR.png?imageView2/2/w/1240/h/0)
composer update 这个命令在我们现在的逻辑中，可能会对项目造成巨大伤害。

因为 composer update 的逻辑是按照 composer.json 指定的扩展包版本规则，把所有扩展包更新到最新版本，注意，是 所有扩展包，举个例子，你在项目一开始的时候使用了 monolog，当时的配置信息是

> "monolog/monolog": "1.*",
> 安装的是 monolog 1.1 版本，而一个多月以后的现在，monolog 已经是 1.2 了，运行命令后直接更新到 1.2，这时项目并没有针对 1.2 进行过测试，项目一下子变得很不稳定，情况有时候会比这个更糟糕，尤其是在一个庞大的项目中，你没有对项目写完整覆盖测试的情况，什么东西坏掉了你都不知道。

那应该使用哪个命令呢？install, update 还是 require ？

接下来我们一一解释。

## **简单解释**

> composer install - 如有 composer.lock 文件，直接安装，否则从 composer.json 安装最新扩展包和依赖；
> composer update - 从 composer.json 安装最新扩展包和依赖；
> composer update vendor/package - 从 composer.json 或者对应包的配置，并更新到最新；
> composer require new/package - 添加安装 new/package, 可以指定版本，如： composer require new/package ~2.5.

## **流程**

下来介绍几个日常生产的流程，来方便加深大家的理解。

**流程一：新项目流程**
创建 composer.json，并添加依赖到的扩展包；
运行 composer install，安装扩展包并生成 composer.lock；
提交 composer.lock 到代码版本控制器中，如：git;

**流程二：项目协作者安装现有项目**
克隆项目后，根目录下直接运行 composer install 从 composer.lock 中安装 指定版本 的扩展包以及其依赖；

> 此流程适用于生产环境代码的部署。
> **流程三：为项目添加新扩展包**

使用 composer require vendor/package 添加扩展包；
提交更新后的 composer.json 和 composer.lock 到代码版本控制器中，如：git;

## **关于 composer.lock 文件**

composer.lock 文件里保存着对每一个代码依赖的版本记录（见下图），提交到版本控制器中，并配合composer install 使用，保证了团队所有协作者开发环境、线上生产环境中运行的代码版本的一致性。
![file](https://lccdn.phphub.org/uploads/images/201806/07/23400/XKuRQme1xN.png?imageView2/2/w/1240/h/0)

## **关于扩展包的安装方法**

那么，准备添加一个扩展包，install, update, require 三个命令都可以用来安装扩展包，选择哪一个才是正确的呢？

答案是：使用 composer require 命令

另外，在手动修改 composer.json 添加扩展包后，composer update new/package 进行指定扩展包更新的方式，也可以正确的安装，不过不建议使用这种方法，因为，一旦你忘记敲定后面的扩展包名，就会进入万劫不复的状态，别给自己留坑呀。

上面的概念不论对新手或者老手来说，都比较混淆，主要记住这个概念：

> 原有项目新添加扩展的，都使用 composer require new/package 这种方式来安装。

需要加版本的话

> composer require "foo/bar:1.0.0"

## **更新指定扩展到指定版本**

有时候你之前使用过的扩展包，加入了新功能，你想更新单独这个扩展包到指定版本，也可以使用 require 来操作。

如下面例子，需要更新 “sami/sami”: “3.0.” 到 “sami/sami”: “3.2.”
![file](https://lccdn.phphub.org/uploads/images/201806/07/23400/atA7yD6Cvb.png?imageView2/2/w/1240/h/0)
命令行运行：
![file](https://lccdn.phphub.org/uploads/images/201806/07/23400/gmcFdRZpkJ.png?imageView2/2/w/1240/h/0)

# over！
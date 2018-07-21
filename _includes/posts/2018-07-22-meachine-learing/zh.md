## 机器学习教程网站

[莫烦PYTHON](https://morvanzhou.github.io/)




## 直接从Python 使用H 2 O.

1.先决条件安装了：Python 2.7.x，3.5.x或3.6.x.

2.安装依赖项（如果需要，可以在前面添加`sudo`）：
```
pip install requests 
pip install tabulate 
pip install scikit-learn 
pip install colorama 
pip install future
```
在命令行中，一次一行地复制并粘贴这些命令：
```
＃以下命令删除Python 的H 2 O模块。
pip uninstall h2o 

＃接下来，使用pip安装这个版本的H 2 O Python模块。
pip install http://h2o-release.s3.amazonaws.com/h2o/rel-wright/3/Python/h2o-3.20.0.3-py2.py3-none-any.whl
```

安装完成之后运行,命令如下所示：
```
python
import h2o
h2o.init()
h2o.demo("glm")
```


## Tip:
1. 当遇到pip的版本过低的情况执行如下命令进行更新
```
python -m pip install --upgrade pip
```
2. 当遇到安装 h2o-3.20.0.3-py2.py3-none-any.whl失败
```
多数是因为网络导致的，直接将此文件下载到本地然后安装
$ pip install file
```

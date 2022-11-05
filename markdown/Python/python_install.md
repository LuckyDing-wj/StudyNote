[TOC]

# 1. 安装Python

## 1.1 下载python安装包

+ 官网下载安装包，复制到服务器目录下

+ 命令下载

  ```shell
  wget https://www.python.org/ftp/python/3.9.5/Python-3.9.5.tgz
  ```

## 1.2 安装依赖包

避免安装过程中出现的"zipimport.ZipImportError"错误 和"ModuleNotFoundError: No module named '_ctypes'" 错误

```shell
yum -y install zlib-devel libffi-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
```

## 1.3 解压、编译、安装

```shell
tar -zxvf Python-3.9.2.tgz

cd Python-3.9.2

./configure --prefix=/usr/local/python3 --enable-optimizations

make && make install
```

如果 编译时提示 Could not import runpy module 错误

导致原因：

在低版本的gcc中带有–enable-optimizations参数

解决方法（不推荐使用方法1）：

+ 升级gcc至高版本，gcc 8.1.0已修复此问题

+ ./configure参数中去掉–enable-optimizations

如果2无法解决，则只能用方法1： 升级 gcc 版本(注意退出root/Python-3.9.2/目录)

安装 CentOS 软件集：

```sh
yum install -y centos-release-scl
```

安装编译工具链：

```sh
yum install -y devtoolset-8-toolchain
```

启用新的工具链：

```sh
scl enable devtoolset-8 bash
```

其他：

--enable-optimization 和 --with-lto 做了哪些优化 ？

在 stackoverflow 上有个关于这个问题的讨论： Python 编译优化都做了哪些？大体意思是能够使 python 提升 10%-20% 速度。

## 1.4 修改环境变量

```shell
vim /etc/profile
# 找到 "export PATH" 开头的行，在此行之前，插入新行，内容如下
PATH=$PATH:/usr/local/python3/bin
source /etc/profile
```

## 1.5 查看两个版本python和pip是否共存

```shell
[root@VM-0-2-centos ~]# python3 -V
Python 3.9.5
[root@VM-0-2-centos ~]# pip3 -V
pip 21.1.1 from /usr/local/python3/lib/python3.9/site-packages/pip (python 3.9)
[root@VM-0-2-centos ~]# python -V
Python 2.7.5
[root@VM-0-2-centos ~]# pip -V
-bash: pip: command not found
[root@VM-0-2-centos ~]# 

# 腾讯CentOS7自带的python2.7.5 没有pip
```

因为系统的 yum，以及其他组件，均依赖 python2.7.5，所以不建议使用软链接的方式替换原来的 python2 环境。使用 python3 的时候，我们只需要按如下的操作即可：

```shell
# pip3 install 包名

# python3 -m pip install 包名

# python3 -m pip install pymongo (安装Python3的pymongo包)

# python3 myscripts.py

# pip3 install --upgrade pip (升级pip3版本)
```

# 2. 安装python包

```shell
pip3 install --upgrade pip -i https://pypi.tuna.tsinghua.edu.cn/simple/  

pip3 install copyheaders  -i   https://pypi.tuna.tsinghua.edu.cn/simple/  

pip3 install pandas  -i   https://pypi.tuna.tsinghua.edu.cn/simple/  

pip3 install tushare  -i   https://pypi.tuna.tsinghua.edu.cn/simple/  

pip3 install pymongo -i   https://pypi.tuna.tsinghua.edu.cn/simple/  

pip3 install pymysql  -i   https://pypi.tuna.tsinghua.edu.cn/simple/  

pip3 install xlrd -i   https://pypi.tuna.tsinghua.edu.cn/simple/  

pip3 install cryptography -i   https://pypi.tuna.tsinghua.edu.cn/simple/  
```

# 3. Python包导出与安装

使用 pip freeze 获取安装的 Python 包

有时，我们为了代码稳定、代码迁移等，需要获取当前 Python 工程依赖包的安装列表。这个列表要包括需要安装什么包、以及包的版本。这便是：requirements.txt。

pip 使用 requirements.txt 安装

输入命令：

pip install -r requirements.txt

即可安装 requirements.txt 中的所有包（指定版本）。

pip freeze

使用 pip freeze 会输出所有在本地已安装的包（但不包括 pip、wheel、setuptools 等自带包），若需要输出内容与 pip list 一致，需使用 pip freeze -all。

使用方法：

pip freeze > requirements.txt

适用场合

由于 pip freeze 与 pip list 内容区别不大，所以，若想要用其作为工程依赖包列表，一定要配合 Python 虚拟环境 virtualenv 使用。：python -m venv 位置

 

# 4. 运行后台程序

为了避免本机突然断电等突然问题，我们需要将程序放在服务器后台运行，（如果有一些输出文件，输出到文件中方便查看，因为放在后台运行之后，将不能看到控台输出的信息）

## 1、在后台运行程序命令

仍然是上面test.py的例子，其对应的后台运行的命令是：

nohup python3 test.py >myoutput.file 2>&1 &

命令具体释义如下：

①nohup表示在后台运行

②python3 test.py是运行python文件的命令

③>表示输出，其右侧接文件名。

③myoutput.file是系统的自动输出文件（不是代码中的，代码中的信息会输出到代码指定的目录文件中。这里的输出内容是指程序自身的运行信息，如：数据库拒绝连接等）

④2>&1表示把错误重定向到myoutput.file的重定向输出中。带&的命令行，就是terminal终端即使关闭，或者电脑死机程序依然运行。（执行完此命令后，就可以将控制台直接关闭了，不会影响程序的运行。）

## 2、进程查看

（1）使用jobs 可查看当前控制台正在运行的进程（只能查到当前控制台，若关闭了则不能使用jobs查看）

（2）使用ps -ef可查看所有正在运行的进程（所有程序）

若想要限定查看python3的进程，可使用ps -ef | grep python3，其中| grep表示筛选条件。

（3）若想要关闭此后台运行的程序，则需要先使用ps -ef命令查找到当前进程的pid，再使用kill命令杀死此进程：

```shell
ps -ef | grep python3

kill -9 pid
```


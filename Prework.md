# 搬运一些施工必要材料
* 安装java
* 安装GoGui
* 安装GnuGo
* 安装MuGo
* GoGui常用命令
* 在服务器上运行DarkForest
* DarkForest vs MuGo

## 安装java
如果ubuntu上已经有java的话可以跳过这一步，没有的话需要去安装一下并配置好环境变量

可以使用下面的command检查是否装好了java：

```Bash
java -version
```

下载地址：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
我在ubuntu上下载的是jdk-8u152-linux-x64.tar.gz

## 安装GoGUI
Google上能找到的相关网站是这个：https://senseis.xmp.net/?Gogui

但是这个网站给出的链接下载的是.tar.gz文件，安装时会提示缺少lib文件夹，所以建议直接下载repo上的.zip文件

* 解压.zip文件
```Bash
unzip gogui-1.4.9.zip
```

* 运行install.sh文件
```Bash
sudo ./install.sh -j JAVA_HOME
```
在服务器下
```Bash
./install.sh
```
注意这一步因人而异，install.sh 默认 JAVA_HOME=/usr. 如果你的JAVA_HOME和默认的不一致的话会导致安装失败。可以用下面这个command找到你的JAVA_HOME:
```Bash
which java
```
在成功安装java后应该会显示：JAVA_HOME/bin/java

如果以上步骤都没有问题的话，这里安装是非常快的，但是安装成功也不会有提示...可以在terminal中输入以下command来确认，出现GoGui的界面则安装成功
```Bash
gogui
```

### 在服务器上安装GoGui
由于没有服务器的sudo权限，最大区别在于安装脚本和PREFIX上，需要作出一点相应的改动

新建一个local文件夹，具体路径为~/envi/local，然后打开install.sh做两处修改:
1) 第3行： PREFIX=/usr/local => PREFIX=~/envi/local
2) 第56行： if [ -f $FILE -a -x $FILE ]; then => if [ -f $FILE -a $FILE ]; then

余下安装与在本地安装是同理
使用时
```Bash
~/envi/local/gogui
```
## 安装GnuGo
输入以下command直接安装：
```Bash
sudo apt-get install gnugo
```

## 安装MuGo
MuGo是用python3写成的部分复现AlphaGo的围棋AI，原repo：https://github.com/brilee/MuGo

直接下载release中的版本，直接clone整个repo的话需要重新训练网络，而release中的版本有一个训练好的模型

Ubuntu下默认的python版本是python2，README中的几个指令需要相应地改成pip3：
```Bash
pip3 install -r requirements.txt
```
同理运行时指定python3 运行
```Bash
python3 main.py gtp policy --read-file=saved_models/20170718
```
其中 saved_models/20170718 是release版本中训练好的模型参数
### 在服务器上安装MuGo
需要先配置好虚拟环境，助教提供的服务器上没有acaconda，需要用virtualenv，基本步骤如下：
* 创建一个管理环境的文件夹
```Bash
mkdir goenvs
```
* 移动到刚创建好的目录下，新建一个虚拟环境
```Bash
virtualenv -p /usr/bin/python3 go
```
* 激活环境，感觉virtualenv激活环境比conda麻烦多了...必须要到指定的目录下才能激活
```Bash
source ~/goenvs/go/bin/activate
deactivate
```
* 安装相关依赖，在激活环境后，直接输入python和pip都是默认python3的
```Bash
cd ~/MuGo-1.0
pip install -r requirements.txt
```


## GoGui常用命令
### 在GoGui中导入一个 Go Engine
在terminal中打开GoGui. Program => New Program 之后按照提示做即可

### 让两个Go AI 在gogui中对局
以MuGo和GnuGo为例，每个程序需要指定启动命令
```Bash
BLACK="gnugo --mode gtp"
WHITE="python3 main.py gtp policy --read-file=saved_models/20170718"
TWOGTP="gogui-twogtp -black \"$BLACK\" -white \"$WHITE\""
gogui -program "$TWOGTP" -computer-both -auto
```

### 让两个Go AI 对局并分析数据
同样以MuGo为例，关于-analyze更多可以参考：https://www.mankier.com/1/gogui-twogtp

文档中给出的命令会自动在当前路径下生成.sgf文件，对局多的时候会出现一大堆文件，可以用一个文件夹来保存所有对局数据。在保存数据的路径下运行下面指令即可：
```Bash
BLACK="gnugo --mode gtp"
WHITE="python3 ../main.py gtp policy --read-file=../saved_models/20170718"
gogui-twogtp -black "$BLACK" -white "$WHITE" -games 10 -size 19 -alternate -sgffile gnuvsmugo -auto

gogui-twogtp -analyze gnuvsmugo.dat
```
对局结果可以在.html文件中看到，“B+3.5”指黑棋赢3.5目，“B+R”指黑棋胜，白棋投降

### Darkforest vs MuGo
原理与gnugo vs MuGo是一样的，语法逻辑还是指定相应的运行指令, 注意相对应地修改指定的路径即可。注意Darkforest源码指定了运行主程序要在./cnnPlayerV2路径下，不想改代码的话就在这个路径下运行好了。MuGo没有这样的要求
```Bash
cd ./cnnPlayerV2
BLACK="th cnnPlayerMCTSV2.lua --num_gpu 2 --time_limit 10 --pipe_path ../data/local/go"
WHITE="python ../../MuGo-0.1/main.py gtp policy --read-file=../../MuGo-0.1/saved_models/20170718"
../../local/bin/gogui-twogtp -black "$BLACK" -white "$WHITE" -games 100 -size 19 -sgffile ./dfvsmugo/dfvsmugo -auto -verbose
```
上面的路径可以根据具体情况进行修改
-auto 让两个AI自动进行对战
-verbose 将两个AI的gtp输出打印在终端上，DF打印的信息是非常详细的

## 在服务器上运行DarkForest
这一步踩了很多坑，本来试图在虚拟机上安装DF相关依赖，如torch，CUDA等等，都没有成功。查阅网上资料显示在虚拟机里面使用CUDA是非常困难的，无奈只能找助教接了一个服务器账号，在服务器上运行。本次课程提供的服务器已经装好了torch、CUDA等等相关依赖。首先连接服务器后下载DarkForest整个repo:
```Bash
git clone https://github.com/facebookresearch/darkforestGo.git
```
DarkForest主页上Build部分基本已经完成，只需要执行compile这一步:
```
sh ./compile.sh
```
Usage部分需要注意Step 2. 我们没有权限访问/data/local/go，需要重新创建一个path来指定后面的pipe file path。比如我在实验室提供的账号下创建了路径: ~/data/local/go,则Step 2相对应要改为：
```Bash
cd ./local_evaluator
sh cnn_evaluator.sh 1 ~/data/local/go
```
最后注意Step 3中要运行主程序的话，主程序内默认的pipe file path = /data/local/go，这里运行时同样需要重新指定:
```Bash
th cnnPlayerMCTSV2.lua [other option] --pipe_path ~/data/local/go
```
To load an existing game up to move 23:
```Bash
th cnnPlayerMCTSV2.lua [other_options] --setup_board "/path/to/sgf 23"
```

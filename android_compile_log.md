# 编译AOSP源码
参考android open source project 文档 https://source.android.com/
## 1.环境搭建
建议使用官方推荐的ubuntu 16.04。其他linux发行版也没啥大的问题，但是需要自己解决下依赖的问题。具体安装方法自行google。我的screenfetch如下：
```
                          ./+o+-       eng@g750
                  yyyyy- -yyyyyy+      OS: Ubuntu 16.04 xenial
               ://+//////-yyyyyyo      Kernel: x86_64 Linux 4.4.0-64-generic
           .++ .:/++++++/-.+sss/`      Uptime: 3h 10m
         .:++o:  /++++++++/:--:/-      Packages: 2065
        o:+o+:++.`..```.-/oo+++++/     Shell: bash 4.3.46
       .:+o:+o/.          `+sssoo+/    Resolution: 1920x1080
  .++/+:+oo+o:`             /sssooo.   DE: Unity 7.4.0
 /+++//+:`oo+o               /::--:.   WM: Compiz
 \+/+o+++`o++o               ++////.   WM Theme: Flatabulous
  .++.o+++oo+:`             /dddhhh.   GTK Theme: Flatabulous [GTK2/3]
       .+.o+oo:.          `oddhhhh+    Icon Theme: Papirus-Dark
        \+.++o+o``-````.:ohdhhhhh+     Font: Ubuntu 11
         `:o+++ `ohhhhhhhhyo++os:      CPU: Intel Core i7-4700HQ CPU @ 3.4GHz
           .o:`.syhhhhhhh/.oo++o`      GPU: GeForce GTX 860M
               /osyyyyyyo++ooo+++/     RAM: 2601MiB / 11898MiB
                   ````` +oo+++o\:    
                          `oo++.      
```

1. 解决依赖
```
//更新系统
sudo apt-get update && sudo apt-get dist-upgrade 

//安装openjdk
sudo apt-get install openjdk-8-jdk

//安装shadowsocks-qt5 (安装之后配置代理)
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5

//安装并配置proxychains
sudo apt-get install proxychains
sudo gedit  /etc/proxychains.conf
修改代理为
socks5  127.0.0.1 1080  //1080改为你自己的端口

//安装依赖环境
sudo apt-get install git-core gnupg flex bison gperf build-essential \
  zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
  lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
  libgl1-mesa-dev libxml2-utils xsltproc unzip

//配置usb访问权限
wget -S -O - http://source.android.com/source/51-android.rules | sed "s/<username>/$USER/" | sudo tee >/dev/null /etc/udev/rules.d/51-android.rules; sudo udevadm control --reload-rules

//安装repo 
mkdir ~/bin
PATH=~/bin:$PATH
proxychains curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

//创建源码下载文件夹 
mkdir 你的路径
```
2. 配置环境变量

Ubuntu的个人环境变量目录为~/.bashrc 

```
gedit ~/.bashrc 
```

在文件末尾添加以下内容并修改为自己的路径

```
#repo path
export PATH=/home/你的用户名/bin:$PATH

#android compile path
export OUT_DIR_COMMON_BASE=你的路径
export USE_CCACHE=1
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4096m"
export CCACHE_DIR=你的路径/.ccache
```
添加完成之后，使其立即生效
```
source ~/.bashrc
```

3. 下载源码

repo是基于git开发的版本管理工具，如果你没配置过git,需要先执行

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

如果你想在实体机器上面安装 到这里寻找对应机型的branch
```
https://source.android.com/source/build-numbers.html#source-code-tags-and-builds
```
如果没有branch ，进入你的路径并初始化repo
```
cd 你的路径
proxychains repo init -u https://android.googlesource.com/platform/manifest
```
如果带有branch 

```
cd 你的路径
proxychains repo init -u https://android.googlesource.com/platform/manifest -b 你的branch
```
最后是同步，时间较长,断了重新运行 多线程自己计算，公式：cpu线程数x(1~2)，我的是4核8线程，就是8～16 ，不过太多线程容易断
```
proxychains repo sync -j8
```
4. 编译源码

先clean一下源码：
```
make clobber
```
配置初始化环境
```
source build/envsetup.sh
```
选择输出版本
```
lunch 
```
我这里是nexus 6,根据官网文档 选择带有userdebug的版本

如果你要刷入硬件，需要添加相应的驱动，下载以后放在源码目录，并执行shell文件
```
https://developers.google.com/android/drivers
```


最后一步，编译,线程选多点，时间视电脑配置而定，一个小时到几个小时不等
```
make -j16
```
编译完成后不要关闭终端，启动模拟器
```
emulator
```
如果模拟器运行较慢
```
sudo apt-get install qemu-kvm
```
5. 刷入手机

如果没配置android sdk环境变量，先配置一下 
```
gedit ~/.bashrc
```
添加以下路径
```
#android sdk tools path
export PATH=/你的sdk路径/tools:$PATH
export PATH=/你的sdk路径/platform-tools:$PATH
```
添加完成后，使其生效
```
source ~/.bashrc
```
编译完成后刷机文件在你的输出路径下的  /target/product/你的版本 下，连接手机
```
adb reboot bootloader //进入bootloader
fastboot flash boot boot.img
fastboot flash system system.img
fastboot flash userdata userdata.img
fastboot flash cache cache.img
fastboot reboot
```



# 1 nuc基本配置

## 1.1 安装Ubuntu服务器版

插入装有ubuntu服务器版的u盘，进入u盘启动，正常安装即可。给/swap分配16g，/efi分配512m，剩下的硬盘空间全部挂载到/目录下。设备名看这台nuc给哪个机器人用，用户名dynamicx（下方name），再填机器类型（上方name），密码dynamicx。安装完成后按照提示移除u盘，重启。

## 1.2 安装ssh，并通过ssh用你的电脑操控nuc

1、将自己的电脑连接上wifi，然后用一根网线将nuc和你的电脑连接。这时你的电脑左上方会出现有线连接的图标。进入有线连接，点击设置，点击IPV4,设置“与其他计算机共享网络”（类似的意思）。可用`ping baidu.com`来检查是否已联网。 
---
*若无法连接，请手动配置wifi*
--- 
2、安装ssh服务器端
`sudo apt install openssh-server`
**如需要安装依赖，按提示安装**
3、在nuc上使用`ip a`查看nuc的ip地址。在你的电脑上输入指令`ssh dynamicx@“nuc的ip地址”`，这样你就能在你的电脑上操控nuc了。

## 1.3 换源

**网上搜索国内的源，例如清华源，阿里源，换源。网上的清华源有的能用，有的是坏的，如果装清华源锅了就换一个。**

## 1.4 安装easywifi

1、在github上搜索easywifi,第一个就是。将源代码clone下来，注意要用http。
2、安装easywifi依赖：`sudo apt-get install network-manager-config-connectivity-ubuntu`
3、进入easywifi文件夹，输入`sudo python3 easywifi.py`
4、成功运行easywifi，运行  *1*<!--Scan for networks-->  搜索wifi，然后运行  *5*<!--Setup new network-->  输入wifi名称和密码，让nuc连上wifi。

5、和nuc连上同一个wifi，继续用ssh操控nuc。

## 1.5 安装ros

安装ros请主要参考ros_wiki上的安装教程。请注意安装base而不是full-desktop。

### 1.5.1 安装catkin tools

catkin tools官方文档：https://catkin-tools.readthedocs.io/en/latest/
如果你使用catkin build时需要你安装osrf-pycommon>0.1.1这个依赖，请输入以下指令：
`sudo apt-get install python3-pip`

`pip3 install osrf-pycommon`

### 1.5.2 rosdep

rosdep update 失败的参考解决方法：https://github.com/SparkChen927/rosdep

`git clone https://gitclone.com/github.com/SparkChen927/rosdep.git`

## 1.6 把github上我们团队rm的代码拉下来，装依赖。

**
我们团队为rm_control和rm_controllers搭建了软件源，请根据[这个网站](https://rm-control-docs.netlify.app/quick_start/rm_source)给nuc添加软件源并把相应的软件包拉下来。**

添加该软件源后，执行`sudo apt update` 可能会出现报错。

解决方法：

`sudo vim /etc/apt/sources.list`

删除该内容:`deb https://rmsource.gdutelc.com/ubuntu/ focal main`

终端执行 `sudo apt upgrade` 后重新添加软件源。

[这个网站](https://rm-control-docs.netlify.app/quick_start/rm_source)中的`rm-control`多了个`s`

<big>**重要！！不要把rm_gazebo拉到nuc上**</big>

<big>**重要！！不要把rm_gazebo拉到nuc上**</big>

<big>**重要！！不要把rm_gazebo拉到nuc上**</big>

大多数依赖都是ros软件包，名称大多为ros-noetic-××××。可以用`sudo apt-get install ros-noetic-****`来安装。

## 1.7 优化

1、你会发现开机很慢，这是一个系统服务导致的，可以设置将其跳过。

`$ sudo vim /etc/netplan/01-netcfg.yaml`
注：这个文件可能不叫这个名字，可能需要转到/etc/netplan这个目录下看看。

在网卡的下一级目录中增加

`optional: true`

修改完后生效设置
`$ sudo netplan apply`

2、阻止nuc休眠

nuc长时间不用会休眠，会给工作带来一定麻烦。因此需要设置阻止nuc休眠。输入以下指令：

`sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target`

## 1.8 换内核

1、使用搜索引擎搜索xanmod，通常搜索结果第一个就是，打开[此网站](https://xanmod.org)。

2、我们需要更换一个实时性更强的内核，这样的内核通常名字里会带有“rt”（realtime）。在这个网站往下拉会看到“Install via Terminal”(通过命令行安装)。根据提示安装自己想要的内核。

3、使用指令`sudo dpkg --get-selections | grep linux-image `来查看你想要安装的内核是否安装成功。

4、重启，按F2进入BIOS模式。在boot->Boot Priority勾选Fast boot。Power选项里勾选Max Performance Enabled,Dynamic Power
Technology设为最长的那个，Power->Secondary Power Settings将After Power Failure设为Power on。cooling选项里将Fan Control
Mode设置为Fixed，Fixed Duty Cycle设为100。***关闭安全启动***然后退出BIOS，正常启动。

5、测试新内核的实时性和can总线传输速率

P.S:如果进不了BIOS可尝试长按开机键直至指示灯变成橙色。

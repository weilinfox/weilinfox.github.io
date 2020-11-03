# 树莓派系统安装和配置 WiringPi

2020-11-03

吃灰多年的树莓派终于要派上用场了，三年来树莓派的系统总体上来说变化不少。

## 系统镜像烧录

这里只给出简单说明。

1. 在其[官网](https://www.raspberrypi.org/downloads/raspberry-pi-os/)下载需要的版本镜像并验证SHA-356。此处建议“Download Torrent”，相对更快；
2. 使用家喻户晓的Win32 disk imager烧录至卡，并验证；
3. 插卡上电开机。

**我使用的是 Raspberry Pi OS (32-bit) Lite** ，毕竟使用WiringPi并不需要图形界面；
值得注意的是和三年前的版本比起来，它会自动检测TF卡剩余空间并自动扩充根目录所在分区。

## 简单配置

用户名为 ``pi`` ，初始密码为 ``raspberry`` 。

+ 修改密码

```sh
sudo passwd pi
sudo passwd root
```

均两次输入新密码即可。

+ 本地化

我傻乎乎的按着 archwiki 找半天（完全不一样），最后发现明明自带有极其方便的工具：

```sh
sudo raspi-config
```

进入“Localisation Options”。

“Change Locale”修改语言环境。取消 ``en_GB.UTF-8 UTF-8`` ，选择 ``en_US.UTF-8 UTF-8`` ；如果需要中文请选上 ``zh_CN GB2312`` 、 ``zh_CN.GB18030 GB18030`` 和 ``zh_CN.UTF-8 UTF-8`` 。选择 Ok 后修改默认语言，英文选择 ``en_US.UTF-8`` ，中文选择 ``zh_CN.UTF-8`` 。这里使用空格键和回车键选中和确认，使用Tab和方向键移位，必要时使用Page Up和Page Down。

“Change Time Zone”修改市区，对于生活在中华大地上的人们，自然选择 “Asia->Shanghai”。

“Change Keyboard Layout”修改键盘映射。默认的键盘映射啥的都是英国的，这里改为美式键盘即可。

“Change WLAN Country”设置Wifi地区（不同国家可能有Wifi密码长度限制），选择 ``CN China`` 即可。

+ 更换软件源

我使用了清华源，具体配置可以看[清华源的帮助](https://mirrors.tuna.tsinghua.edu.cn/help/raspbian/)。下面是部分摘抄内容。

这里 Raspbian 对应的 Debian 版本为 Debian10(buster) 。相对于删除原文件所有内容，个人认为注释掉更好。

```sh
# 编辑 `/etc/apt/sources.list` 文件，删除原文件所有内容，用以下内容取代：
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib rpi
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib rpi

# 编辑 `/etc/apt/sources.list.d/raspi.list` 文件，删除原文件所有内容，用以下内容取代：
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
```

最后执行 ``sudo apt-get update`` 检查配置是否正确。

+ 连接Wifi

刚开始并没有意识到 ``raspi-config`` 里面连接Wifi功能的存在，而是手动配置了Wifi。所以这一段仅供参考。对于 raspi-config ，执行 ``sudo raspi-config`` ，选择“Network Options -> Wireless Lan”即可。

*下面是我手动配置Wifi的过程。*

首先取消rfkill的限制， ``rfkill list`` 可以看到相关默认配置：

```sh
rfkill unblock all
```

再次执行 ``rfkill list`` 可以看到限制已经取消。

其次打开无线网卡开关 ``sudo ifconfig wlan0 up`` ，之后就可以使用 ``wpa_cli`` 等配置之。

我还安装了比较喜欢的 ``iwd`` ： 

```sh
sudo apt-get install iwd
sudo systemctl enable iwd
```

``iwd`` 在重启后依然会自动连接上次的Wifi。

+ 打开ssh

树莓派自带了ssh但是并没有默认打开。

这也可以通过 ``raspi-config`` 设置，在“Interfacing Options”的“SSH”选项，同时也有“VNC”的选项。

下面是手动设置的过程，仅供参考。

```sh
sudo systemctl enable ssh
sudo systemctl start ssh
```

此时直接试图用管理员用户登陆的时候会被拒绝，需要在配置文件中修改使其允许。当然也可以建立一个普通用户登陆后 ``su`` 为管理员用户。

```sh
# 想用vim自行安装 sudo apt-get install vim
sudo nano /etc/ssh/sshd_config
```

其中有一行：

```sh
#PermitRootLogin prohibit-password
```

依照之在文件中添加一行（而不是直接在原行上改）：

```sh
PermitRootLogin yes
```

保存后重启ssh即可：

```sh
sudo systemctl restart ssh
```

+ 安装WiringPi

其页面有非常详细的[安装向导](http://wiringpi.com/download-and-install/)。最简单的方法莫过于直接**从源中安装**：

```sh
sudo apt-get install wiringpi
```

也可以**从源码构建**，但是这个网站（[git.drogon.net](https://git.drogon.net/)）并不是那么容易打开。好在Github上也有。

首先安装必要的构建工具：

```sh
# 很多教程写的是 git-core，表示非常不能理解
sudo apt-get install gcc git
```

然后在一个你喜欢的地方克隆WiringPi在Github上的仓库：

```sh
git clone https://github.com/WiringPi/WiringPi.git
cd WiringPi/
./build
# 这条可能需要
sudo ldconfig
```

最后验证一下是否安装成功：

```sh
gpio -v
```

我的输出如下：

```
gpio version: 2.60
Copyright (c) 2012-2018 Gordon Henderson
This is free software with ABSOLUTELY NO WARRANTY.
For details type: gpio -warranty

Raspberry Pi Details:
  Type: Pi 3, Revision: 02, Memory: 1024MB, Maker: Embest 
  * Device tree is enabled.
  *--> Raspberry Pi 3 Model B Rev 1.2
  * This Raspberry Pi supports user-level GPIO access.
```

读取gpio引脚编号信息：

```sh
gpio readall
```

这是我的Raspberry Pi 3B的输出：

```
 +-----+-----+---------+------+---+---Pi 3B--+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 |   IN | 1 |  3 || 4  |   |      | 5v      |     |     |
 |   3 |   9 |   SCL.1 |   IN | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 0 | IN   | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 1 | IN   | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI |   IN | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO |   IN | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK |   IN | 0 | 23 || 24 | 1 | IN   | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | IN   | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | IN   | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 0 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+---Pi 3B--+---+------+---------+-----+-----+
```

by SDUST weilinfox
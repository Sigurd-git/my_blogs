# This is a note showing how to install RaspberryOS Desktop 64bit on Raspberry Pi 3B.

## 1. Download the image 下载镜像

Download the image from [here](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2023-05-03/2023-05-03-raspios-bullseye-arm64.img.xz).

## 2. 格式化SD卡

MAC中无法直接看到已经烧录了linux系统的SD卡，因此在MAC上格式化SD卡需要特殊技巧

### 方法1(Easy)

使用打包好的工具，如[SD Card Formatter](https://www.sdcard.org/downloads/formatter/sd-memory-card-formatter-for-mac-download/)

### 方法2(Hard but do not need to install any software)

使用命令行格式化

```bash
# 查看SD卡的信息
df -h
## 根据最后一列Mounted on的信息，找到SD卡对应的行， 这一行的第一列（Filesystem）为SD卡的路径，如/dev/disk4s1

#使用diskutil unmount将这些分区卸载
diskutil unmount /dev/disk4s1

#使用使用指令dd覆盖磁盘的第一个扇区512个字节，我们之前看到SD卡对应的磁盘为disk4，那么这里就是rdisk4（其实写disk4也行，但是rdisk4可以加速）
sudo dd bs=512 count=1 if=/dev/zero of=/dev/rdisk4

#使用diskutil eject弹出SD卡
diskutil eject /dev/rdisk4

##重新插入SD卡，在弹出的messagebox中点击初始化（Initialize）
##格式化SD卡 格式为FAT32，磁盘名为RASPBERRY， 分区表为MBR（磁盘不大于2T都可以用这个，否则就要用GPT格式，但需要额外的配置步骤，所以直接选择使用MBR）
diskutil eraseDisk FAT32 RASPBERRY MBRFormat /dev/disk4
```

## 3. 烧录镜像

解压下载好的镜像，我这里是ubuntu-22.04.2-preinstalled-server-arm64+raspi.img.xz
然后使用dd命令烧录镜像到SD卡



```bash
# 解压镜像，记得cd到镜像所在的目录
xz -d 2023-05-03-raspios-bullseye-arm64.img.xz

# 查看SD卡的信息，理论上来说跟之前一样，都是/dev/disk4s1，但你此时可以看到，这个分区已经被格式化了，而且磁盘名为RASPBERRY，且Size为SD卡的大小
df -h

#为了防止在写入镜像的时候有其他读取或写入，我们需要卸载设备
diskutil unmount /dev/disk4s1

#使用dd命令写入镜像，这里的if是输入文件（镜像），of是输出文件（SD卡的路径），bs是每次读取的字节数（一般1m或4m或8m），status=progress是显示进度（有的dd不支持显示进度，这个参数不能用）
sudo dd if=2023-05-03-raspios-bullseye-arm64.img of=/dev/rdisk4 bs=8m status=progress

#烧录完成后，使用diskutil eject弹出SD卡
diskutil eject /dev/rdisk4
```

## 4. 配置网络(桌面版不需要这一步，直接在设置里面连接即可)

给树莓派连接显示屏，鼠标和键盘并开机。
默认账号密码为pi和raspberry
会让你马上改密码并重新登陆

然后开始配置网络
    
```bash
# 查看网卡信息
ip a
## 一般来说，有线网卡名为eth0，无线网卡名为wlan0，这里我们需要配置无线网卡。如果网卡名不是wlan0，那么请将下面的wlan0替换为你的网卡名

# 编辑配置文件 vim不存在则使用vi或者nano
sudo vim /etc/wpa_supplicant/wpa_supplicant.conf

```

打开vim后，按i进入编辑模式，然后输入以下内容

1. 如果你的wifi没有密码，则输入

network={
    ssid="你的wifi名"
    key_mgmt=NONE
}

2. 如果你的 WiFi 使用WEP加密，则输入

network={
    ssid="你的wifi名"
    key_mgmt=NONE
    wep_key0="你的wifi密码"
}

3. 如果你的 WiFi 使用WPA/WPA2加密，则输入

network={
    ssid="你的wifi名"
    key_mgmt=WPA-PSK
    psk="你的wifi密码"
}

然后按esc退出编辑模式，输入:wq保存并退出

然后执行以下命令启动无线网卡并获取ip地址

```bash

# 启动无线网卡
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf

# 获取ip地址
sudo dhclient wlan0

# 查看ip地址
ip a
```



## 更新并配置SSH

```bash
# 更新软件源
sudo apt update

# 更新软件
sudo apt upgrade

#安装net-tools
sudo apt install net-tools

```

## 查看IP地址以便之后SSH连接

```bash
# 查看ip地址
ifconfig
```

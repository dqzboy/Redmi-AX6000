# Redmi AX6000 刷入 OpenWrt 教程

> 所需刷机软件包下载：[网盘链接](https://pan.baidu.com/s/1oidLLpI6KeaGFq3SMjJVBA?pwd=3ms4)

## 一、准备工作说明

在进行红米AX6000路由器的刷机操作之前，需要做好以下准备工作，以确保刷机过程的顺利和稳定：

### 硬件连接：
- 将红米AX6000路由器的WAN口连接到现有的网络，以确保路由器可以接收到网络信号。
- 使用网线将路由器的LAN口与一台电脑相连。建议使用有线连接，因为这样可以提供更稳定的连接，减少刷机过程中可能出现的连接问题。

### 软件准备：
- 准备一个可以连接SSH（Secure Shell）和Telnet的工具。SSH是一种加密的网络协议，用于安全地访问和控制远程计算机或设备。Telnet是一种较旧的网络协议，用于远程登录到服务器，但安全性低，通常不推荐使用。
- 确保电脑已安装相应的SSH客户端软件，以便在刷机过程中可以远程登录到路由器进行操作。

### 其他注意事项：
- 在刷机前，建议备份当前路由器的配置文件，以防万一刷机失败，可以恢复到刷机前的状态。
- 确保了解刷机的具体步骤和可能的风险，如果不熟悉刷机过程，可以查找相关教程或寻求专业人士的帮助。
- 刷机过程中，保持电源稳定，避免突然断电导致刷机失败。
- 刷机完成后，可能需要重新配置路由器的网络设置，包括WAN和LAN的相关参数。

通过以上步骤，可以为红米AX6000路由器的刷机工作做好充分的准备，降低刷机过程中出现问题的风险，确保刷机过程的顺利进行。

> ⚠️ **重要提醒**：刷机有风险，刷机需谨慎，请在了解以及掌握一定的相关知识后再决定是否进行刷机。

**免责声明**：本文档仅供学习和参考之用，不对因刷机导致的问题承担责任。如果刷机过程中出现任何问题，包括设备损坏（俗称"变砖"）等，需要您自行解决。

## 二、解锁路由器SSH

截止本文发布时，Redmi AX6000固件版本为1.0.67，此版本无需刷固件即可解锁SSH。据说小米已经在修复SSH的漏洞了，以后解锁SSH就比较困难了。

解锁SSH的详细步骤教程，可以参考科学上网中的教程

## 三、刷入mt798x uboot

### 1、刷机方案介绍

**使用官方固件**：
- 通过SSH命令行安装所需插件。
- 优点：可以方便地恢复到官方版本，适合对系统稳定性有要求的用户。

**刷入第三方固件**：
- 第三方固件通常有两个版本：
  - **官方分区版**：基于官方固件的分区大小，固件体积受限，仅能安装基本插件，但易于恢复到官方固件。
  - **Uboot大分区版本**：利用路由器的全部128M ROM空间，允许安装更大体积的固件和更多插件，提供更多自定义选项。
- Uboot版本的优点是刷写第三方固件更为方便，且不易变砖，而且还可以直接通过uboot刷入官方固件（注意是修改版的官方固件，不是直接从官方下载的固件）。
- 刷写过程存在一定风险，需要确保不断电。

mt798x uboot 项目地址：https://github.com/hanwckf/bl-mt798x

### 2、备份原厂分区

所需要准备的工具有hanwckf大神编译的红米AX6000 uboot大分区版的openWRT固件，并通过scp工具上传到路由器。

官方固件解锁后SSH软件登录路由器查看原厂分区，可以看到原厂的ubi和ubi1两个固件分区是30MB，ubinfo -a查看实际可用29MB

```bash
cat /mtd
cat /partitions
```

![image](https://github.com/user-attachments/assets/d1f32921-7505-428c-8007-493ed9fdf25b)
![image](https://github.com/user-attachments/assets/f542e8c7-507d-4282-9c82-13f1540d09c4)


**重要**：刷机之前我们可以用命令行备份几个原厂的分区，这样以后还可以通过备份恢复到官方固件

```bash
# 运行dd命令备份分区到tmp文件夹
dd if=/dev/mtd1 of=/tmp/mtd1_BL2.bin
dd if=/dev/mtd2 of=/tmp/mtd2_Nvram.bin
dd if=/dev/mtd3 of=/tmp/mtd3_Bdata.bin
dd if=/dev/mtd4 of=/tmp/mtd4_Factory.bin
dd if=/dev/mtd5 of=/tmp/mtd5_FIP.bin
```
![image](https://github.com/user-attachments/assets/578baeda-01c1-4cad-9f8e-d382453787ce)

- 通过WinSCP等SCP协议软件登录路由器，打开tmp文件夹，将上面的备份文件下载到电脑保存好。

![image](https://github.com/user-attachments/assets/374705ce-fedf-45c4-b5c8-bb565cf2ff0f)


### 3、刷入Uboot

所需要准备的工具有hanwckf大神编译的红米AX6000的mt798x uboot文件和237大佬的uboot大分区版的OpenWRT固件，并通过scp工具上传到路由器。

> **注意**：此步骤这里上传的是mt798x uboot文件，非openwrt系统固件

![image](https://github.com/user-attachments/assets/18b95cd5-1793-4301-90d5-a5398f0fc3ac)


我们需要先刷入uboot 再通过启动uboot来刷入OpenWRT系统

然后逐条输入以下命令，把uboot刷入到FIP分区：

```bash
# 验证md5
md5sum /mt7986_redmi_ax6000-fip-fixed-parts-multi-layout.bin
# 下面的操作就是擦出分区和写入uboot的命令了。
## 注意：擦除和写入FIP分区时不能断电、重启，不然路由器就会直接变砖。
mtd erase FIP
mtd write /mt7986_redmi_ax6000-fip-fixed-parts-multi-layout.bin FIP
mtd verify /mt7986_redmi_ax6000-fip-fixed-parts-multi-layout.bin FIP
```

![image](https://github.com/user-attachments/assets/490cc665-d9c6-4616-b5bf-04d4b31f13a1)


### 4、刷入OpenWRT固件

对比检查最后输出 "Success" 说明刷入已成功，此时拔掉路由器电源，接着按住路由器的RESET按钮后通电开机，等待15秒后松开RESET。心里默念15秒

> **注意**：红米AX6000官方系统支持WAN、LAN切换，可以随意插网线自动识别，但是刷op后WAN口固定是1口（靠近电源插头的那个口），2-4口是LAN口，网线插LAN口才能获取到IP，登录路由器。

电脑通过网线连接到路由器LAN口(2-4)，然后将电脑的IP手动更改为192.168.31.100/24，用Chrome浏览器进入192.168.31.1，就会见到uboot的webui(Web failsafe UI)，在这个webui页面选择要刷入的大分区固件。

- 刷入固件的兼容性说明，查看官方项目文档：[Redmi AX6000 uboot 固件兼容性](https://github.com/hanwckf/bl-mt798x)

> **注意**：图示中的二进制固件包名称可能与网盘中实际提供的不同。随着版本更新，包名会发生变化，请以网盘中的实际文件为准。

- 选择支持immortalwrt-mt798x固件的immortalwrt-110m分区

![image](https://github.com/user-attachments/assets/33e3d230-aa1a-464c-816f-d5105b64b2a5)
![image](https://github.com/user-attachments/assets/345276a4-b641-4ec6-b8cc-71993ce0af4a)
![image](https://github.com/user-attachments/assets/6884a723-256a-4044-9384-c7acdfb8fe55)

- 这里提示信息表示为：文件上传成功，更新正在进行中，等待设备自动复位!
![image](https://github.com/user-attachments/assets/5979821f-bb84-4dc0-a900-8c6f4e51d17f)

- 出现下面的界面说明固件安装完成，等待路由器初始化完成。路由器没有灯光显示，只有通过在无线里面找到以 ImmortalWrt开头的无线名称存在了，我们就可以进行下面的配置了，然后下面中的页面就可以直接关闭了
![image](https://github.com/user-attachments/assets/7f471ae4-bc39-4cb8-8c0f-8179945662b3)

> **注意**如果手动修改过网卡IP，那么刷入OP后需要改为DHCP，不然访问不了OP后台管理页面!


## 四、OpenWrt配置

### 1、访问后台

- 默认访问地址是：192.168.6.1  账号密码：root/password

![image](https://github.com/user-attachments/assets/b4b859eb-b32b-4083-bf61-6ec1c9bddf0a)


### 2、网络配置

默认刷的op系统，作者把上网的接口就固定在了路由器上的WAN口了(上面刷操作时已经说过了)，如果你是光猫拨号，这里无需修改，保持默认DHCP；如果是光猫桥接路由器拨号的这里改为 PPPoE 协议 
保存之后，记得应用下，然后我们访问网页看下是否可以正常上网了。正常情况下就可以直接使用无线上网了，接下来就是配置无线密码

![image](https://github.com/user-attachments/assets/45341572-f924-4b10-9fb3-26c86c9956e9)



## 五、刷回官方固件

如果使用了一段时间的op系统后，觉得不好用或者觉得还不如官网系统好用，那么我们也可以刷回到官方系统去，操作请参考下面教程

同样在这里需要提醒大家：**刷机有风险，刷机需谨慎！**

### 方式一：刷回官方Uboot恢复

#### 1、恢复工具下载

小米恢复工具和官方固件下载地址：http://www.miwifi.com/miwifi_download.html

#### 2、还原FIP分区

通过winscp工具连接到路由器，然后把自己备份的mtd5_FIP.bin文件上传到路由器的目录下

使用md5sum检查上传到tmp文件夹的原厂uboot文件的md5值和你保存的是否一样，无误后用mtd write将原厂uboot文件写入FIP分区，再用mtd verify对比检查原厂uboot文件是否已写入FIP分区

> **注意**：写入FIP分区时不能断电、重启，不然就直接变砖

```bash
# 验证md5
md5sum /mtd5_FIP.bin
 
mtd write /mtd5_FIP.bin FIP
mtd verify /mtd5_FIP.bin FIP
```
![image](https://github.com/user-attachments/assets/55cf69ca-6b9b-4181-8fc0-36f60117935b)
![image](https://github.com/user-attachments/assets/76104fd0-fe53-46b6-8743-7031eaa6f6ee)

> 对比检查最后输出 "Success" 说明刷入已成功，可以断电路由器，然后打开小米路由官方修复工具进行修复了。

![image](https://github.com/user-attachments/assets/6592a615-7d00-4b7f-ad73-0c90ff02265e)
![image](https://github.com/user-attachments/assets/1d67665b-758a-4d43-9f2a-422351d198d9)


#### 3、刷回官方固件

电脑通过网线连接到路由器LAN口(2-4)，路由器断电，电脑退出杀毒软件，还有Windows的自带Windows Defender防火墙杀毒，必要时关闭电脑防火墙

打开小米路由修复工具，选择官方的rb06固件，点下一步
![image](https://github.com/user-attachments/assets/2818d852-24f5-43e7-ac89-b9039eacf6b3)

网卡选择当前连接路由器的网卡，点下一步，工具会自动配置网卡IP为192.168.31.100/24，然后点击下一步（如果出现提示网卡配置错误，那么继续执行此步骤，此时有可能已经出现了31.100的网卡信息了）

![image](https://github.com/user-attachments/assets/f3e87aad-08ef-4a07-be04-8b35b3cf0a67)

配置好后会显示刷机步骤，然后按住路由器RESET按键，然后插电开机，大概12秒后等到黄灯闪烁后松开RESET，等待小米路由修复工具自动连接路由器开始上传固件，上传完后会自动刷机，刷机成功后蓝灯闪烁。等待10秒后重新断电插电即可恢复到官方系统。点击退出小米路由修复工具，网卡会自动恢复自动获取的配置。

![image](https://github.com/user-attachments/assets/c80e8160-31bc-482e-98f1-9c1e609415ee)


> **注意**：蓝光颜色比较浅，要仔细观察才能看清楚，不然会一直以为刷机没成功

### 方式二：通过mt798x uboot刷官方固件恢复

需要再次恢复至hanwckf mt798x uboot即可，拔掉路由器电源，接着按住路由器的RESET按钮后通电开机，等待15秒后松开RESET。心里默念15秒

> **注意**：恢复后记得手动更改PC以太网IP为192.168.31.100/24，然后浏览器输入192.168.31.1 就可以进到 mt798x uboot UI界面了

#### 1、上传官方修改版uboot专用固件

选择default分区，然后上传官方红米AX6000固件，版本1.0.70，这个版本是官方未公布的版本，是一个隐藏版本
![image](https://github.com/user-attachments/assets/57b38165-a37f-4d29-b414-aeb2655e6673)
![image](https://github.com/user-attachments/assets/0d044fc6-b050-489a-b91a-b59e44868f07)
![image](https://github.com/user-attachments/assets/7d02a231-588b-4c35-b597-6752cd5bde6e)

#### 2、配置官方路由设置

固件上传完成之后，等待几分钟后路由器亮蓝灯，这个时候还需要继续等待，当路由器蓝灯闪烁，并且通过PC WIFI可以查看到小米官方默认的WIFI名称后，就说明固件刷入完成了，接下来就是需要进行配置了。把以太网IP获取方式改为DHCP，然后浏览器输入192.168.31.1进行配置路由器网络就可以了

![image](https://github.com/user-attachments/assets/2b9e1f5b-235d-4166-b300-5ded3c347a38)
![image](https://github.com/user-attachments/assets/5f4b8659-226c-4db5-84f9-067b58a3d559)


# Redmi-AX6000路由器科学上网详细教程

## ShellClash介绍
- 在Linux环境中使用Shell脚本一键部署和管理Clash服务
- 项目地址：https://github.com/juewuy/ShellClash

## 步骤一、解锁路由器SSH
新版本的Redmi AX6000固件 无需刷固件；例如我这里固件版本为：`1.0.64`
### 1、获取stok码
- 登入路由器后台，然后在搜索地址栏复制stok码
- **注意**：`stok`码再每次重新登入路由都会改变，所以以下的步骤都需要是最新的stok码的情况下进行操作
 
> ![image](https://user-images.githubusercontent.com/42825450/215307084-4640f9de-140e-445e-ad5d-ce091713b67b.png)

### 2、开启TELNET
- **注意**：将下面红色标记的对应的stok改为自己上面获取到的路由器对应的stok码
#### 2.1：开启开发/调试模式
```shell
http://192.168.31.1/cgi-bin/luci/;stok=对应的stok/api/misystem/set_sys_time?timezone=%20%27%20%3B%20zz%3D%24%28dd%20if%3D%2Fdev%2Fzero%20bs%3D1%20count%3D2%202%3E%2Fdev%2Fnull%29%20%3B%20printf%20%27%A5%5A%25c%25c%27%20%24zz%20%24zz%20%7C%20mtd%20write%20-%20crash%20%3B%20
```
> ![image](https://user-images.githubusercontent.com/42825450/215307203-7061dd2a-8b07-43bd-b597-af5529a55fb3.png)

#### 2.2：重启路由器
```shell
http://192.168.31.1/cgi-bin/luci/;stok=对应的stok/api/misystem/set_sys_time?timezone=%20%27%20%3b%20reboot%20%3b%20
```
> ![image](https://user-images.githubusercontent.com/42825450/215307224-36051831-ccb1-4c19-a9f5-19bc6634145b.png)
- 等待路由器重启完成，大概2分钟左右时间

#### 2.3：设置Bdata永久开启telnet
- **注意：** 上面重启了路由器之后重新连接无线之后，访问路由器后台需要重新登入，这个时候我们需要获取最新的stok码
> ![image](https://user-images.githubusercontent.com/42825450/215307265-26799c9a-d7f4-4c03-81ce-854fff882417.png)

```shell
http://192.168.31.1/cgi-bin/luci/;stok=对应的stok/api/misystem/set_sys_time?timezone=%20%27%20%3B%20bdata%20set%20telnet_en%3D1%20%3B%20bdata%20set%20ssh_en%3D1%20%3B%20bdata%20set%20uart_en%3D1%20%3B%20bdata%20commit%20%3B%20
```
> ![image](https://user-images.githubusercontent.com/42825450/215307274-03c6a489-b9a9-42ca-b9a2-c36339f7cd3c.png)

#### 2.4：重启路由器
```shell
http://192.168.31.1/cgi-bin/luci/;stok=对应的stok/api/misystem/set_sys_time?timezone=%20%27%20%3b%20reboot%20%3b%20
```
> ![image](https://user-images.githubusercontent.com/42825450/215307288-1620aaef-d72c-4340-8ad6-9af6ef328ce2.png)


### 3、开启SSH
#### 3.1：连接路由器
- 连接到对应的无线WiFi，然后主机输入192.168.31.1，协议选择TELNET 端口默认23
- **提示：** 我这里使用的xshell工具
> ![image](https://user-images.githubusercontent.com/42825450/215307331-1ca2614c-b7c6-4521-81c5-5a6edd216b44.png)
> ![image](https://user-images.githubusercontent.com/42825450/215307332-40e45a74-558c-40af-83dd-769d457ba8ed.png)

#### 3.2：修改root密码
- 连上telnet后运行下列命令
```shell
echo -e 'admin\nadmin' | passwd root
```
- root密码修改为admin
> ![image](https://user-images.githubusercontent.com/42825450/215307356-f391b655-d2a1-48ef-b3f5-86aa928e947e.png)

#### 3.3：固化SSH
- 固化SSH之后，以后升级路由器固件SSH也会保留不会被恢复
```shell
nvram set ssh_en=1
nvram set telnet_en=1
nvram set uart_en=1
nvram set boot_wait=on
nvram commit
```
#### 3.4：永久开启SSH
- 重启不会关闭
```shell
mkdir /data/auto_ssh && cd /data/auto_ssh
curl -O https://cdn.jsdelivr.net/gh/lemoeo/AX6S@main/auto_ssh.sh
chmod +x auto_ssh.sh

uci set firewall.auto_ssh=include
uci set firewall.auto_ssh.type='script'
uci set firewall.auto_ssh.path='/data/auto_ssh/auto_ssh.sh'
uci set firewall.auto_ssh.enabled='1'
uci commit firewall
```

#### 3.5：修改时区设置
```shell
cd
uci set system.@system[0].timezone='CST-8'
uci set system.@system[0].webtimezone='CST-8'
uci set system.@system[0].timezoneindex='2.84'
uci commit
```

#### 3.6：关闭开发/调试模式
```shell
mtd erase crash
```

#### 3.7：重启路由器
```shell
reboot
```
> ![image](https://user-images.githubusercontent.com/42825450/215307420-4d4d3ec8-b5fa-409c-b7e1-efca3064e32b.png)

### 4、连接SSH
- 路由器重启之后，确保PC连接成功，然后再使用xshell工具并选择ssh协议连接路由器
> ![image](https://user-images.githubusercontent.com/42825450/215307443-c1b5861e-3058-4f7b-8357-7be424842ac6.png)
> ![image](https://user-images.githubusercontent.com/42825450/215307448-ef1266e5-8da0-4487-97ce-42505727a0ee.png)
- **提示：** 这里的root密码就是上面修改的密码 admin
> ![image](https://user-images.githubusercontent.com/42825450/215307457-da8d94e8-0c72-45e5-adef-ab6114d66550.png)
> ![image](https://user-images.githubusercontent.com/42825450/215307462-d4a7e041-708c-43a2-9aa6-03924c0fb0e4.png)

## 步骤二、配置科学上网
- 上面步骤一解锁了路由SSH连接之后，现在我们就需要配置路由器科学上网了；
- **注意：** 需提前准备科学上面的订阅地址或者连接地址

### 1、安装前设备环境检查
- 主要是检查设备是否安装了iptables以及加载了对应的模块，不然你部署的时候只能使用环境变量在配置clash代理了！

#### (1) 确保系统上已安装了 iptables 或者nftables，以及加载了必要的 iptables模块。你可以通过运行以下命令检查：
```shell
# iptables 模式
command -v iptables
lsmod | grep '^xt_owner'
 
# nftables 模式
command -v nft
```

#### (2) 如果 `lsmod | grep '^xt_owner'` 为空，说明在内核中没有加载 `xt_owner` 模块，这可能导致 iptables 的某些功能不可用。首先，确保 iptables 软件包已正确安装。
```shell
sudo apt-get update
sudo apt-get install iptables

# 加载 xt_owner 模块： 手动加载 xt_owner 模块，可以使用 modprobe 命令：
sudo modprobe xt_owner
# 再次检查 xt_owner 是否加载
lsmod | grep '^xt_owner'
```
> 说明：如果 xt_owner 模块不可用，可能是因为系统内核没有相应的模块。在某些系统中，该模块可能已经被包含在内核中，而在其他系统中，你可能需要重新编译内核或安装额外的内核模块。

#### (3) 如果 command -v nft 返回为空，表示在系统中找不到 nft 命令。这可能是因为 nft 工具没有被正确安装或者安装路径不在系统的可执行路径中。以下是一些解决方法：
```shell
sudo apt-get update
sudo apt-get install nftables
```

### 2、安装Clash
#### 路由设备使用curl安装：

```shell
#GitHub源(可能需要代理)
export url='https://raw.githubusercontent.com/juewuy/ShellCrash/master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
```
或者
```shell
#jsDelivrCDN源
export url='https://fastly.jsdelivr.net/gh/juewuy/ShellCrash@master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
```
或者
```shell
#作者私人源
export url='https://gh.jwsc.eu.org/master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
```

#### 路由设备使用wget安装：

```Shell
#GitHub源(可能需要代理)
export url='https://raw.githubusercontent.com/juewuy/ShellCrash/master' && wget -q --no-check-certificate -O /tmp/install.sh $url/install.sh  && sh /tmp/install.sh && source /etc/profile &> /dev/null
```
或者
```shell
#jsDelivrCDN源
export url='https://fastly.jsdelivr.net/gh/juewuy/ShellCrash@master' && wget -q --no-check-certificate -O /tmp/install.sh $url/install.sh  && sh /tmp/install.sh && source /etc/profile &> /dev/null
```

- 通过XShell连接到路由器，执行上面的命令进行安装
```shell
export url='https://cdn.jsdelivr.net/gh/juewuy/ShellClash@master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
```
> ![image](https://user-images.githubusercontent.com/42825450/215307524-9965ca83-d5fc-44b1-9aa0-bbda738ffde2.png)

### 2、配置科学上网
> ![image](https://user-images.githubusercontent.com/42825450/215307586-69bb5f34-f581-4556-9227-49fa6d4fd62d.png)
> ![image](https://user-images.githubusercontent.com/42825450/215307591-40f46c61-8d80-4bd3-a603-a1a97e7ec08d.png)
> ![image](https://user-images.githubusercontent.com/42825450/215307597-e38ff197-4a4b-4ccb-8ec2-73f4ddeec5bf.png)
> ![image](https://user-images.githubusercontent.com/42825450/215307603-2b5a9390-211a-4720-a832-609436d13ef3.png)
> ![image](https://user-images.githubusercontent.com/42825450/215307614-b1930c8c-587b-451c-ba31-ffc15c55913b.png)


### 3、访问管理面板
- 浏览器访问地址为：http://192.168.31.1:9999/ui/
> ![image](https://user-images.githubusercontent.com/42825450/215307636-324a7938-9f55-4351-8946-1c718ae74d94.png)

### 4、访问外网测试
- 现在访问Google和YouTube测试
> ![image](https://user-images.githubusercontent.com/42825450/215307656-3b48a90c-dc84-409f-8deb-696bcc3e00ed.png)
> ![image](https://user-images.githubusercontent.com/42825450/215307659-cd41204f-edcd-4873-b245-d9cb1f17877c.png)

### 5、clash功能设置
> ![image](https://user-images.githubusercontent.com/42825450/215307670-2658ac9a-edbc-46f7-9608-aaa3dbb5c6bc.png)
![image](https://user-images.githubusercontent.com/42825450/215307674-398712f2-2805-4d46-aa7b-b8ca528d15ce.png)
![image](https://user-images.githubusercontent.com/42825450/215307679-73be6a5c-973e-47d4-936f-19b0b177411b.png)
![image](https://user-images.githubusercontent.com/42825450/215307680-002889a8-40b8-43ee-98d4-166541c4ce1f.png)
![image](https://user-images.githubusercontent.com/42825450/215307682-0ae03c61-c35d-4744-b266-5da38f3171c1.png)

### 6、Clash开机启动
> ![image](https://user-images.githubusercontent.com/42825450/215307712-6cfb0f07-f8f2-46b9-9e40-65a37eb84aaa.png)

### 7、自定义规则
```bash
cd /data/clash/yamls/
# 添加自定义规则
vi rules.yaml
```

如 chatgpt(注意前面有空格)
```yaml
 - DOMAIN-SUFFIX,openai.com,🚀 节点选择
 - DOMAIN-SUFFIX,chatgpt.com,🚀 节点选择
 - DOMAIN-SUFFIX,statsig.com,🚀 节点选择
 - DOMAIN-SUFFIX,ingest.com,🚀 节点选择
 - DOMAIN-SUFFIX,revenuecat.com,🚀 节点选择
```
> 以上就是所有的操作步骤

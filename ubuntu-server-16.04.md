# ubuntu server 16.04 

# 关机、重启

```bash
reboot  # 重启
shutdown    # 关机
```

# 系统安装 

- 安装过程`向导语言`选择`英语`，其他语系有BUG，安装过程会报错失败！
- `DHCP`自动配置可能不成功

# 启动 root 帐号

```bash
# 安装过程会让你新建一个用户，安装完成后，你需要手动开启root
sudo passwd root
#输入你当前用户的密码
#输入你希望的root用户的密码
#确认密码
```

# 网络配置

## 1. 查看网络配置，启动网卡

```bash
ifconfig
# 如果只显示lo（本地环回接口）信息，代表网卡未启动
ifconfig -a
# 显示所有网络接口的信息，无论是否激活
# 可以看见如 eth0 或 enp5s0 或 ens33 或 ens192 等， 如果看不到，代表缺少驱动！
ifconfig  eth0 或 enp5s0 或 ens33 或 ens192  up
ifconfig
# 再 ifconfig 就可以看见不是lo一个了
```

## 2. 配置网卡

```bash
vi /etc/network/interfaces
# 不是root帐号，没有权限时： chmod 777 /etc/network/interfaces
    auto enp5s0 # 开机自动连接网络(ens0 为网卡名称,ifconfig -a看自己的)
    iface enp5s0 inet static    # static表示使用固定ip，dhcp表述使用动态ip
    address 192.168.3.105   # 设置ip地址
    netmask 255.255.255.0   # 设置子网掩码
    # broadcask 114.113.149.127 # 广播地址，不知道啥用，暂时没配
    gateway 192.168.3.254   # 设置网关
    # :wq 保存退出
```

## 3. 配置 DNS 域名

```bash
vi /etc/resolv.conf
    nameserver 202.106.0.20  # 北京联通
    nameserver 159.226.5.65  # 北京市中国科学院软件研究所
    # 国外的 DNS 写前面
    # :wq 保存退出

# 也可以直接在 /etc/network/interfaces 里的最后直接加dns
    # dns-nameservers 8.8.8.8   # 设置DNS,谷歌dns
    # dns-nameservers 202.106.0.20  # 北京市联通dns

cat /etc/resolv.conf    # 查看dns配置
cat /etc/network/interfaces 
```

## 4. 重启网络服务

```bash
service networking restart
ifconfig    # 就会显示刚才配置的
```

# bashrc 和 profile

|A|B|C|D|
|-|-|-|-|
|/etc/profile |用来设置系统环境参数 |- |对系统内所有用户生效 |
|/etc/bashrc |用来设置系统bash shell相关 |- |对系统内所有用户生效，用户运行bash命令 |
|~/.bash_profile |用来设置系统环境参数 |和 /etc/profile 类似 |针对用户自己，只有用户登录时才会生效 |
|~/.bashrc |用来设置系统bash shell相关 |和 /etc/bashrc 类似 |针对用户自己，用户不一定登录，只要以该用户身份运行命令行 |

# 设置外网代理

```bash
# 方法一： 全体帐号使用
vi /etc/profile
export http_proxy="http://username:password@address:port"  # 追加
export https_proxy="http://username:password@address:port"  # 追加
export ftp_proxy="http://username:password@address:port"  # 追加
# 退出重新登录

# 方法二： 本帐号使用
vi ~/.bashrc
export http_proxy="http://username:password@address:port"
export https_proxy="http://username:password@address:port"
export ftp_proxy="http://username:password@address:port"
source ~/.bashrc    # 生效

# 方法三： 临时使用 如：用apt-get之前于终端下输入：
export http_proxy="http://username:password@address:port"
export https_proxy="http://username:password@address:port"
export ftp_proxy="http://username:password@address:port"
```

# SSH 远程访问

```bash
# 安装
apt-get install -y openssh-server

# 提示 apt-get: Package has no installation candidate 时 如何解决：
# 同步 /etc/apt/sources.list 和 /etc/apt/sources.list.d 中列出的源的索引，
# 这样才能获取到最新的软件包
apt-get update  
# 升级 已安装的所有软件包，升级之后的版本就是本地索引里的，
# 因此，在执行 upgrade 之前一定要执行 update, 这样才能是最新的
apt-get upgrade 

# 查看
ps -e | grep ssh    # sshd 代表ssh服务已经启动
service ssh start   # 没有启动 请启动

# 配置
vim /etc/ssh/sshd_config
    port 22     # 默认端口为 22， 需要的话可以修改
    PermitRootLogin yes # 修改 允许root帐号远程登录
    # PermitRootLogin prohibit-password 允许root登录，但是禁止root用密码登录
    PermitEmptyPasswords no # 修改 不允许密码为空
    # :wq 保存退出

# 服务重启
service ssh restart

# 远程登录
ssh username@address # lunux 机器终端下输入
# windows 机器下载ssh客户端并使用，如 xshell, putty
```

# GIT

## 1. 全局配置

```bash
# 默认已安装 git
git --version
git config --list
# git config --global user.name "John Doe"
# git config --global user.email johndoe@example.com
```

## 2. 使用ssh协议

```bash
# git 4种协议： file, git, ssh(默认), http(s)
# 使用ssh协议， 你必须是你要克隆的项目的拥有者或管理员，且需要添加 SSH key
cd ~/.ssh
# 存在 id_rsa.pub 或 id_rsa.pub，表示已有SSH key

# 产生密钥
ssh-keygen -t rsa -C "your_email@example.com"
# 输入文件名保存key（默认：id_rsa.pub私钥 或 id_rsa.pub共钥 ）
# 提示你输入两次密码（push文件的时候要输入的密码，最好不用）

# 公共仓服务器，文件所属用户的当前目录下，找到 .ssh/authorized_keys 文件
# 追加内容： id_rsa.pub 共钥 (跨服务器间复制资料用到命令： scp)
# 可能需要重启服务

# 从其他服务器克隆项目
cd folder
git clone user@server:/path.git # 表示ssh方式
```

# NODE

```bash
# 安装 nvm
git clone https://github.com/creationix/nvm.git ~/.nvm 
cd ~/.nvm 
git checkout `git describe --abbrev=0 --tags`
cd
vim .bashrc
将 source ~/.nvm/nvm.sh 添加进我们的.bashrc中，保存退出
执行source .bashrc

nvm -v #查看nvm版本
nvm ls-remote #查看远端可用node版本
nvm install x.x.x #安装node到本地
nvm alias default x.x.x #调整node版本
nvm use x #临时调整node版本
```

# node-oracledb

```bash
# 下载安装 Instant Client
# ORACLE网站下载Instant Client for Linux x86-64： Basic、SDK、sqlplus 的ZIPs
cd /opt/oracle
unzip instantclient-basic-linux.x64-12.2.0.1.0.zip
unzip instantclient-sdk-linux.x64-12.2.0.1.0.zip
mv instantclient_12_2 instantclient
cd instantclient
ln -s libclntsh.so.12.1 libclntsh.so
ln -s libocci.so.12.1 libocci.so

# 安装 libaio1 包 (有的linux下叫libaio)
sudo apt install libaio1

# ~/.bashrc添加配置环境变量
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=$ORACLE_BASE/instantclient
export OCI_LIB_DIR=/opt/oracle/instantclient
export OCI_INC_DIR=/opt/oracle/instantclient/sdk/include
export PATH=$ORACLE_HOME:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME:${LD_LIBRARY_PATH}
export TNS_ADMIN=$ORACLE_HOME/network/admin/
export NLS_LANG="AMERICAN_AMERICA.UTF8"

# 重进终端，测试sqlplus
sqlplus username/password@//serverIP:1521/serviceName
>sql select sysdate from dual;

# 安装 GCC
sudo apt install gcc

# 安装 python
# 默认已安装 python2.7
sudo ln -s /usr/bin/python2.7 /usr/bin/python

# 安装 node-oracledb
# 安装过程提示：可能需要再装： sudo apt install make, node-gyp, c++
npm install oracledb
```

# htop

sudo apt install htop

|a|b|
|-|-|
|PID|进程id|
|USER|进程所有者|
|PR|进程优先级|
|NI|nice值。负值表示高优先级，正值表示低优先级|
|VIRT|进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES|
|RES|进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA|
|SHR|共享内存大小，单位kb|
|S|进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程|
|%CPU|上次更新到现在的CPU时间占用百分比|
|%MEM|进程使用的物理内存百分比|
|TIME+|进程使用的CPU时间总计，单位1/100秒|
|COMMAND|进程名称（命令名/命令行）|

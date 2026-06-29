---
title: Linux学习笔记
date: 2026-06-29 16:00:00
permalink: /2026/06/29/linux-notes/
tags:
  - Linux
  - 运维
categories:
  - 技术学习
---

## 一、系统信息

- hostnamectl ：查询Linux系统信息

- uname -a ：查看内核版本等系统信息

- cat /etc/os-release ：查看操作系统发行版信息

- curl ifconfig.me ：查看外网 IP 信息

- curl ip.sb ：访问公网接口,获取本机公网ip

- echo：打印输出

- who：查看当前登录用户信息

- whoami：查看当前用户名

- uptime：显示系统运行时间，负载情况

- getent passwd：查看系统所有用户

## 二、进程管理

### 1、查看进程

- ps：查看系统中进程状态

    - ps -ef：显示所有进程的完整信息

    - ps -aux：显示所有进程的详细信息（含CPU、内存占用）

    - ps -ef\|grep java ：查看Java进程

- top：动态监视进程活动与系统负载

    - top中按 P：按CPU排序；按 M：按内存排序；按 q：退出

- htop：增强版top（需安装，界面更友好）

### 2、结束进程

```Shell
# 根据进程号结束进程
kill -9 PID        # 强制杀死进程
kill -15 PID       # 优雅终止进程（允许进程清理资源）

# 根据进程名结束进程
killall java       # 杀死所有名为java的进程
pkill -f "ecology" # 杀死包含ecology关键字的进程
```

### 3、后台运行

```Shell
# nohup：退出终端后进程不中断
nohup java -jar app.jar > app.log 2>&1 &

# &：将命令放入后台运行
java -jar app.jar &

# jobs：查看当前终端的后台任务
jobs -l

# fg/bg：将后台任务调至前台/继续运行
fg %1     # 将1号任务调至前台
bg %1     # 将1号任务继续在后台运行
```

## 三、Vim文本编辑器

### 1、模式切换

- vim filename ：打开/创建文件

- i：进入插入模式（在光标前插入）

- a：进入插入模式（在光标后插入）

- o：在当前行下方新开一行并进入插入模式

- Esc：返回普通模式

### 2、保存与退出

```Shell
:w        # 保存文件
:q        # 退出 Vim
:wq       # 保存并退出
:q!       # 强制退出，不保存更改
:wq!      # 强制保存并退出（用于只读文件）
```

### 3、光标移动

```Shell
gg        # 跳到文件首行
G         # 跳到文件末行
:n        # 跳到第n行
0         # 跳到行首
$         # 跳到行尾
Ctrl+f    # 向下翻页
Ctrl+b    # 向上翻页
```

### 4、编辑操作

```Shell
dd        # 删除（剪切）当前行
ndd       # 删除（剪切）当前行起的n行
yy        # 复制当前行
nyy       # 复制当前行起的n行
p         # 在光标后粘贴
u         # 撤销
Ctrl+r    # 恢复撤销
```

### 5、搜索与替换

```Shell
/keyword        # 向下搜索关键字
?keyword        # 向上搜索关键字
n               # 跳到下一个匹配
N               # 跳到上一个匹配

:%s/old/new/g       # 全文替换（无确认）
:%s/old/new/gc      # 全文替换（逐个确认）
:s/old/new/g        # 替换当前行
:1,10s/old/new/g    # 替换第1-10行
```

### 6、可视模式

```Shell
v         # 字符可视模式（逐字符选择）
V         # 行可视模式（逐行选择）
Ctrl+v    # 块可视模式（列选择，批量注释时常用）
```

## 四、文件与目录操作

### 1、目录操作

- pwd：显示当前所在目录

- cd：切换目录

```Shell
cd ~        # 回到当前用户主目录
cd -        # 返回上一次所在目录
cd ..       # 返回上一级目录
```

- ls：列出目录内容

```Shell
ls -l       # 详细信息
ls -lh      # 详细信息，文件大小可读（K/M/G）
ls -la      # 包含隐藏文件
ls -lt      # 按修改时间排序（最新在前）
```

- mkdir：创建文件夹

```Shell
mkdir dir1                # 创建单级目录
mkdir -p a/b/c/d          # 递归创建多级目录
```

### 2、文件操作

- touch：创建空文件或更新文件时间戳

- cp：复制文件

```Shell
cp file1 file2            # 复制文件
cp -r dir1 dir2           # 递归复制目录
cp -i file1 file2         # 覆盖前确认
```

- mv：移动/重命名文件

```Shell
mv file1 /tmp/            # 移动文件到指定目录
mv -i a /user/b           # 移动并覆盖（覆盖前确认）
mv oldname newname        # 重命名文件
```

- rm：删除文件

```Shell
rm file1                  # 删除文件（需确认）
rm -f file1               # 强制删除文件
rm -rf dir1               # 递归强制删除目录（⚠️ 慎用）
```

### 3、文件传输

- sz：下载文件（sz ecology）

- rz：上传文件

    - rz -y：上传并覆盖同名文件

- scp：远程复制文件

```Shell
# 本地 → 远程
scp local_file user@remote:/path/

# 远程 → 本地
scp user@remote:/path/file /local/path/

# 传输目录
scp -r dir1 user@remote:/path/
```

### 4、压缩与解压

**zip/unzip：**

```Shell
# 压缩单个文件
zip filename.zip file

# 压缩文件夹
zip -r archive.zip folder/

# 解压到指定目录（覆盖已存在文件）
unzip -o archive.zip -d /path/to/extract
```

**tar：更常用的打包压缩方式**

```Shell
# 打包并gzip压缩
tar -zcvf archive.tar.gz folder/

# 解压tar.gz
tar -zxvf archive.tar.gz

# 解压到指定目录
tar -zxvf archive.tar.gz -C /path/to/extract

# 仅打包不压缩
tar -cvf archive.tar folder/

# 解压tar
tar -xvf archive.tar

# 查看压缩包内容（不解压）
tar -ztvf archive.tar.gz
```

> 参数说明：z-gzip压缩，c-创建，x-解压，v-显示过程，f-指定文件名，C-指定解压目录

### 5、查找文件

- find：在指定目录下查找文件

```Shell
find -name "qiyuesuo"              # 查找和契约锁有关的文件
find /home -name "*.txt"           # 在/home目录下查找以.txt结尾的文件名
find / -type f -name "*.log"       # 全盘查找所有log文件
find /tmp -mtime +7                # 查找7天前修改过的文件
find / -size +100M                 # 查找大于100M的文件
find . -name "*.jar" -exec ls -lh {} \;   # 查找jar文件并显示详情
```

- which/whereis：查找命令位置

```Shell
which java         # 查看java命令的路径
whereis java       # 查看java命令及源码、帮助文件位置
```

### 6、查看文件

```Shell
# 查看文件编码格式
file -i 文件名

# 查看整个文件
cat filename

# 查看文件中包含request的地方
cat hhjcSend.log | grep "request"

# 按照时间查看日志
grep "2024-11-18 10:50" hhjcsend.log

# 统计文件行数
wc -l filename

# tail 命令
tail -n 10 test.log      # 查看最后10行
tail -n +10 test.log     # 查看第10行之后的所有内容
tail -fn 100 test.log    # 实时追踪最后100行（最常用）

# less 命令（大文件推荐）
less -N info.log         # 显示行号查看日志
#  b: 上一屏  空格: 下一屏  G: 文档尾部  gg: 文档首部
#  /keyword: 向下搜索  ?keyword: 向上搜索
#  n: 下一个匹配  N: 上一个匹配
#  F: 实时滚动模式（类似tail -f，Ctrl+C退出）
#  一般排查日志流程：
#    1. less -N info.log
#    2. G 定位到文档最尾部
#    3. ?NullPointerException 向上搜索关键字
#    4. n 上一个 / N 下一个匹配

# head 命令
head -n 20 filename      # 查看文件前20行
```

## 五、文件权限与用户管理

### 1、文件权限

```Shell
# 查看权限
ls -l file.txt
# -rw-r--r-- 1 owner group 1234 Jun 29 10:00 file.txt
#  第1位：-文件 d目录 l链接
#  第2-4位：所有者权限(rwx)
#  第5-7位：所属组权限(rwx)
#  第8-10位：其他用户权限(rwx)

# chmod：修改权限
chmod 755 file.sh         # rwxr-xr-x
chmod 644 file.txt        # rw-r--r--
chmod +x file.sh          # 给所有用户添加执行权限
chmod -R 755 dir/         # 递归修改目录权限

# chown：修改文件所有者
chown user:group file     # 同时修改所有者和所属组
chown -R user:group dir/  # 递归修改
```

> 权限数字对照：r=4, w=2, x=1，所以 755 = rwx(4+2+1) + r-x(4+1) + r-x(4+1)

### 2、用户管理

```Shell
su - username       # 切换用户（加载环境变量）
su username         # 切换用户（不加载环境变量）

# 创建用户
useradd username
passwd username     # 设置密码

# 删除用户
userdel username          # 删除用户（保留主目录）
userdel -r username       # 删除用户及主目录

# 用户组
groupadd groupname        # 创建用户组
usermod -aG groupname user  # 将用户加入组
groups username           # 查看用户所属组
```

## 六、磁盘管理

### 1、查看磁盘信息

- lsblk：列出系统中所有的块设备及其相关信息（设备名称、大小、类型、挂载点等）

- df -h：查看各分区使用情况

- du -sh：查看当前目录所占空间大小

```Shell
du -sh *           # 查看当前目录下每个文件/文件夹的大小
du -sh --max-depth=1 /  # 查看根目录下各一级子目录大小
```

- fdisk -l：查看磁盘分区信息

### 2、磁盘挂载

```Shell
# 1. 查看系统是否检测到新硬盘
lsblk

# 2. 格式化新磁盘（以 /dev/sdb 为例）
mkfs.ext4 /dev/sdb

# 3. 创建挂载点
mkdir /data

# 4. 临时挂载（重启后失效）
mount /dev/sdb /data

# 5. 开机自动挂载（写入fstab）
echo '/dev/sdb /data ext4 defaults 0 0' >> /etc/fstab

# 6. 验证挂载
df -h

# 7. 卸载
umount /data
```

## 七、网络相关

### 1、网络连通性

```Shell
ping 8.8.8.8                # 测试网络连通性
ping -c 4 baidu.com         # 发4个包后停止
telnet host port            # 测试端口是否可达
curl -I https://www.google.com  # 请求头部信息，检查连接
```

### 2、端口查看

```Shell
# netstat（传统命令）
netstat -tlnp              # 查看所有监听的TCP端口及进程
netstat -anp | grep 3306   # 查看3306端口状态

# ss（推荐，更快）
ss -tlnp                   # 查看所有监听的TCP端口
ss -tlnp | grep 8080       # 查看8080端口占用
```

### 3、防火墙（firewalld）

```Shell
# 查看防火墙状态
systemctl status firewalld
# 关闭防火墙
sudo systemctl stop firewalld
# 打开防火墙
sudo systemctl start firewalld
# 开机禁用防火墙
sudo systemctl disable firewalld

# 查看已开放的端口
firewall-cmd --list-ports

# 开放端口（以3306为例）
firewall-cmd --zone=public --query-port=3306/tcp           # 查看端口状态
firewall-cmd --zone=public --add-port=3306/tcp --permanent  # 永久开放端口
firewall-cmd --reload                                      # 重载防火墙
firewall-cmd --zone=public --query-port=3306/tcp           # 验证是否开放

# 关闭端口
firewall-cmd --zone=public --remove-port=3306/tcp --permanent
firewall-cmd --reload

# 开放端口范围
firewall-cmd --zone=public --add-port=3000-3100/tcp --permanent
```

## 八、服务管理（systemctl）

```Shell
# 服务基本操作
systemctl start servicename      # 启动服务
systemctl stop servicename       # 停止服务
systemctl restart servicename    # 重启服务
systemctl status servicename     # 查看服务状态
systemctl reload servicename     # 重新加载配置（不中断服务）

# 开机自启
systemctl enable servicename     # 设置开机自启
systemctl disable servicename    # 取消开机自启
systemctl is-enabled servicename # 查看是否开机自启

# 查看所有服务
systemctl list-units --type=service              # 查看所有运行中的服务
systemctl list-units --type=service --all        # 查看所有服务（含未运行）
```

## 九、yum软件包管理

### 1、查找与显示

```Shell
# 检查 MySQL 是否已安装
yum list installed | grep mysql
yum list installed mysql*

yum info package1      # 显示安装包信息package1
yum list               # 显示所有已经安装和可以安装的程序包
yum list package1      # 显示指定程序包安装情况package1
yum groupinfo group1   # 显示程序组group1信息
yum search string      # 根据关键字string查找安装包
```

### 2、安装

```Shell
yum install              # 全部安装
yum install package1     # 安装指定的安装包package1
yum groupinsall group1   # 安装程序组group1
```

### 3、更新与升级

```Shell
yum update               # 全部更新
yum update package1      # 更新指定程序包package1
yum check-update         # 检查可更新的程序
yum upgrade package1     # 升级指定程序包package1
yum groupupdate group1   # 升级程序组group1
```

### 4、卸载

```Shell
yum remove package1      # 卸载指定程序包
yum groupremove group1   # 卸载程序组
yum clean all            # 清除缓存目录下的软件包
```

## 十、关闭自动更新

```Shell
# 安装 yum-cron 包
sudo yum install yum-cron

# 重新启动 systemd 服务
sudo systemctl daemon-reload

# 启动或启用 yum-cron.service
sudo systemctl start yum-cron.service
sudo systemctl enable yum-cron.service

# 停止
systemctl stop yum-cron
# 禁用
systemctl disable yum-cron
# 检查服务状态
systemctl status yum-cron
```

## 十一、使用MySQL

### 1、登录与退出

```Shell
mysql -u ecology -p
# ecology为用户名称
# 退出: exit / quit
```

### 2、操作数据库

```Shell
# 展示数据库
show databases;
# 切换数据库
use ecology

# 在select语句末尾使用"\G"来格式化输出（竖排显示，字段多时更易读）
select * from table where id = 1 \G;

# 查看表结构
desc table_name;

# 查看当前使用的数据库
select database();

# 查看当前用户权限
show grants;
```

## 十二、curl命令测试接口

```Shell
# 请求头部信息，用于检查连接是否正常
curl -I https://www.google.com

# get请求
curl http://example.com

# get带参数
curl "http://example.com/api?name=JohnDoe&age=30"

# post请求 - json数据
curl -X POST http://your_server_ip/auth-center/oauth/sign \
  -H "Content-Type: application/json" \
  -d '{"appId": "your_app_id", "appSecret": "your_app_secret","loginName":"your_login_name"}'

# post请求 - 表单数据
curl -X POST https://example.com/api -d "name=JohnDoe&age=30"

# 自定义请求头的请求
curl -X GET https://example.com/api \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Accept: application/json"

# 输出响应详情（调试常用）
curl -v http://example.com

# 只查看HTTP状态码
curl -s -o /dev/null -w "%{http_code}" http://example.com

# 跟随重定向
curl -L http://example.com
```

## 十三、定时任务（crontab）

```Shell
# 查看当前用户的定时任务
crontab -l

# 编辑定时任务
crontab -e

# 删除所有定时任务
crontab -r
```

**crontab 时间格式：**

```
# 格式：分 时 日 月 周
# ┌──────── 分钟 (0-59)
# │ ┌────── 小时 (0-23)
# │ │ ┌──── 日 (1-31)
# │ │ │ ┌── 月 (1-12)
# │ │ │ │ ┌星期 (0-7，0和7都代表周日)
# * * * * * command
```

**常用示例：**

```Shell
# 每天凌晨2点执行备份
0 2 * * * /home/backup.sh

# 每隔5分钟检查服务
*/5 * * * * /home/check.sh

# 每周一早上8点执行
0 8 * * 1 /home/weekly.sh

# 每月1号凌晨3点执行
0 3 1 * * /home/monthly.sh

# 工作日(周一到周五)每天9点执行
0 9 * * 1-5 /home/workday.sh
```

## 十四、环境变量

```Shell
# 查看所有环境变量
env

# 查看某个变量
echo $JAVA_HOME

# 临时设置环境变量（仅当前终端有效）
export JAVA_HOME=/usr/lib/jvm/java-11

# 永久设置环境变量
# 方式一：用户级别（写入 ~/.bashrc 或 ~/.bash_profile）
echo 'export JAVA_HOME=/usr/lib/jvm/java-11' >> ~/.bashrc
source ~/.bashrc

# 方式二：系统级别（写入 /etc/profile，所有用户生效）
echo 'export JAVA_HOME=/usr/lib/jvm/java-11' >> /etc/profile
source /etc/profile

# PATH中添加新路径
export PATH=$JAVA_HOME/bin:$PATH
```

## 十五、SSH相关

```Shell
# SSH远程登录
ssh user@192.168.1.100           # 默认22端口
ssh -p 2222 user@192.168.1.100   # 指定端口

# 生成SSH密钥对
ssh-keygen -t rsa                # 一路回车即可

# 将公钥复制到远程服务器（免密登录）
ssh-copy-id user@192.168.1.100

# 查看SSH服务状态
systemctl status sshd

# 重启SSH服务
systemctl restart sshd
```

> 💡 免密登录原理：本地生成密钥对 → 公钥上传到远程服务器的 `~/.ssh/authorized_keys` → 登录时自动用私钥认证，无需输入密码。

## 十六、其他实用技巧

```Shell
# 查看历史命令
history
history | grep "mysql"          # 搜索历史中包含mysql的命令
!123                            # 执行history中第123条命令

# 别名设置
alias ll='ls -lh'               # 设置别名
alias                             # 查看所有别名

# 输出重定向
command > file.txt              # 覆盖写入
command >> file.txt             # 追加写入
command 2>&1                    # 错误输出也重定向到标准输出
command > /dev/null 2>&1        # 丢弃所有输出

# 管道组合
ps -ef | grep java | grep -v grep   # 查找java进程但排除grep本身
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10
# 统计访问日志中IP出现次数TOP10

# 批量操作
xargs：将前一个命令的输出作为后一个命令的参数
find /tmp -name "*.log" | xargs rm -f    # 批量删除tmp下的log文件
```

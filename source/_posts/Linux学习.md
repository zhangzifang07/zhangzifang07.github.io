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
- curl ifconfig.me ：查看外网 IP 信息
- curl ip.sb ：访问公网接口,获取本机公网ip

- echo：打印输出
- ps -ef|grep java ：查看Java进程

- ps：查看系统中下进程状态

    - -a：显示所有进程

    - -u：用户以及其他详细信息

- top：动态监视进程活动与系统负载

- free -h：查看内存使用情况

- df -h：查看磁盘空间使用情况（-g表示以G显示）

- du -sh：查看当前目录所占空间大小

- who：查看当前登录用户信息

- getent passwd

- find：在指定目录下查找文件

    ```Shell
    find -name "qiyuesuo"  #查找和契约锁有关的文件
    find /home -name "*.txt"  #在/home目录下查找以.txt结尾的文件名
    ```

- su ：切换用户

- uptime：显示系统运行时间，负载情况

## 二、Vim文本编辑器

- vim 进入

- i 进入编辑模式 esc退回

- Q退出  wq!保存退出

```Shell
i：进入插入模式。
Esc：返回普通模式。
:w：保存文件。
:q：退出 Vim。
:wq：保存并退出。
:q!：强制退出，不保存更改。
```

## 三、yum软件包管理

### 1、查找与显示（yum info）

```Shell
# 检查 MySQL 是否已安装
yum list installed | grep mysql
yum list installed mysql*

yum info package1      #显示安装包信息package1
yum list               #显示所有已经安装和可以安装的程序包
yum list package1      #显示指定程序包安装情况package1
yum groupinfo group1   #显示程序组group1信息yum search string 根据关键字string查找安装包

```

### 2、安装

```Shell
yum install              #全部安装
yum install package1     #安装指定的安装包package1
yum groupinsall group1   #安装程序组group1

```

### 3、更新与升级

```Shell
yum update               #全部更新
yum update package1      #更新指定程序包package1
yum check-update         #检查可更新的程序
yum upgrade package1     #升级指定程序包package1
yum groupupdate group1   #升级程序组group1

```

## 四、磁盘挂载操作手册

### 1、查看系统是否检测到新的硬盘

- 查看磁盘情况：lsblk（会列出系统中所有的块设备及其相关信息。这些信息可以帮助你了解设备的名称、大小、类型、挂载点等

## 五、开放MySQL3306端口，用于远程连接

- 查看防火墙

```Shell
#查看防火墙状态
systemctl status firewalld
#关闭防火墙
sudo systemctl stop firewalld
#打开防火墙
sudo systemctl start firewalld
```

- 查看3306端口状态

```Shell
firewall-cmd --zone=public --query-port=3306/tcp
```

- 如果是no，表示关闭，打开3306端口

```Shell
firewall-cmd --zone=public --add-port=3306/tcp --permanent 
```

- 防火墙重载

```Shell
firewall-cmd --reload
```

- 再次查看3306状态

```Shell
firewall-cmd --zone=public --query-port=3306/tcp
```

- yes说明端口已经打开，可以去navicat测试连接

## 六、使用MySQL

### 1、登录与退出

```shell
mysql -u ecology -p
#ecology为用户名称
#密码 Weaver@2023
#退出:exit/quit
```
### 2、操作数据库

```Shell
#展示数据库
show databases;
#切换数据库
use ecology

# 在select语句末尾使用"\G"来格式化输出
select * from table where id = 1 \G;
```

## 七、操作文件

- 创建文件夹：mkdir

- sz：下载文件（sz ecology）

- rz： 上传文件

    - rz -y  上传并覆盖同名文件

- rm -f 删除文件

- 移动并覆盖文件
   ```Shell
    #移动并覆盖文件
    mv -i a /user/b
    ```

- 复制文件

    ```Shell
    cp workflow_20240925.zip /tmp/log
    #将workflow_20240925.zip复制到指定目录
    ```

- 压缩文件

    ```Shell
    zip filename.zip file
    #将file压缩为filename.zip，file不能是目录
    
    # 压缩文件夹
    zip -r 压缩后的文件名.zip 要压缩的文件夹名
    ```

- 解压

    ```Shell
    unzip -o archive.zip -d /path/to/extract
    #-o 选项表示覆盖（overwrite）已存在的文件 
    #-d /path/to/extract 指定解压到的目标路径。
    ```

- 查看文件大小
  ```Shell
    #可以自动转换为可读的单位大小
    ls -lh
    
    #查看固定文件夹大小
    du -sh
    ```

- 查看文件

    ```Shell
    #查看文件编码格式
    file -i 文件名
    
    #查看文件中包含request的地方
    cat hhjcSend.log |grep "request";
    
    #按照时间查看日志
    grep "2024-11-18 10:50" hhjcsend.log
    
    #tail 命令
    tail  -n  10   test.log   查询日志尾部最后10行的日志;
    tail  -n +10   test.log   查询10行之后的所有日志;
    tail  -fn 10   test.log   循环实时查看最后1000行记录(最常用的)
    
    #less命令
    less -N hhjc.log (显示行号查看日志)
      b:查看上一屏幕
      空格：查看下一屏幕
      G：定位到文档尾部
    #进入less模式后，键入F，即实时滚动文档
        在日志中查找，因日志一般是追加的，从下向上查找更为常用。
        一般我们的查找的顺序就是：
        1. 进入日志：less -N info.log
        2. 定位到文档最尾部：G
        3. 向上匹配查询：?NullPointerException
        4. 定位上一个关键字:n; 定位下一个关键字:`N`
    ```
## 八、关闭自动更新

```Shell
#安装 yum-cron 包
sudo yum install yum-cron

#重新启动 systemd 服务
sudo systemctl daemon-reload

#启动或启用 yum-cron.service
sudo systemctl start yum-cron.service
sudo systemctl enable yum-cron.service

#停止
systemctl stop yum-cron
#禁用
systemctl disable yum-cron
#检查服务状态
systemctl status yum-cron

```
## 九、curl命令测试接口

```Shell
#请求头部信息，用于检查连接是否正常
curl -I https://www.google.com

#get请求
curl http://example.com
#get带参数
curl "http://example.com/api?name=JohnDoe&age=30"

#post请求
#json数据
curl -X POST http://139.196.228.204/auth-center/oauth/sign -H "Content-Type: application/json" -d '{"appId": "hua-heng-portal", "appSecret": "u82nDrid5m4xMQrP","loginName":"HH002286"}'

#表单数据
curl -X POST https://example.com/api  -d "name=JohnDoe&age=30"

#自定义请求头的请求
curl -X GET https://example.com/api  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"  -H "Accept: application/json"

```

## 十、系统运行资源
- df -h ：查看各分区使用情况
## 十一、PS 
---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Linux"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-09-23T16:08:58+08:00
lastmod: 2020-09-23T16:08:58+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

Linux - CentOS

#### 查看系统版本

```
cat /etc/redhat-release
```

#### hostname

1. 设置 hostname

   ```
   sudo hostnamectl set-hostname newname
   ```

2. 修改 /etc/hosts

   ```
   cat >> /etc/hosts << EOF
   192.168.1.100    domain1
   192.168.1.101    domain2
   EOF
   ```

3. 重启networking

   ```
   sudo systemctl restart network
   ```

#### 跟踪脚本

```
bash -x *.sh
```

#### 挂载光盘为本地源

1. 挂载光盘

   ```
   mount /dev/cdrom /mnt/cdrom
   ```

2. 配置本地源文件CentOS-Media.repo

   ```
   baseurl修改第二个路径为上步的挂载点(/mnt/cdrom)
   enabled=1
   ```

3. 禁用默认的yum网络源

   ```
   将yum 网络源配置文件改名为CentOS-Base.repo.bak,否则会先在网络源中寻找适合的包,改名之后直接从本地源读取.
   ```

   

#### 硬盘挂载及扩容

##### 新硬盘挂载

*参考：http://www.linuxidc.com/Linux/2012-07/65294.htm*

1. 查看硬盘分区情况

   ```
   sudo fdisk -l
   ```

2. 新硬盘分区

   ```
   sudo fdisk /dev/sda
   Command (m for help)
   n：增加新分区
   e：扩展分区
   Partition number(1-4)：默认1,只分一个区
   cylinder：默认
   分区大小：默认
   p：显示分区表
   w：保存分区表
   ```

3. 查看分区

4. 格式化

   ```
   sudo mkfs -t ext4 /dev/sda
   ```

5. 挂载目录

   ```
   sudo mount -t ext4 /dev/sda /mnt/sda
   ```

6. 查看

   ```
   df -h
   ```

7. 开机自动挂载(编辑/etc/fstab)

   ```
   在最后一行加入
   /dev/sda1 /mnt/sda1 ext4 defaults 1 2
   或者
   UUID=37eaa526-5d96-4237-8468-603df5216ce9 /mnt/sda ext4 defaults 0 3
   **UUID可以通过使用blkid命令来查看(sudo blkid /dev/sda)
   **０表示不备份；１表示要将整个<file system>中的内容备份。此处建议设置为０。
   **０表示不检查；挂载点为分区／（根分区）必须设置为１，其他的挂载点不能设置为１
   分区      挂载目录 文件格式
   ```

##### 新硬盘扩容根节点

1. 查看硬盘分区情况

   ```
   sudo fdisk -l
   ```

2. 创建物理卷

   ```
   pvcreate /dev/sda
   ```

3. 查看创建好的物理卷

   ```
   pvdisplay /dev/sda
   ```

4. 卷组扩容

   ```
   vgextend gafis70-vg /dev/sda
   ```

5. 查看扩容之后的卷组信息

   ```
   vgdisplay
   ```

6. 逻辑卷扩容

   ```
   # 指定大小
   lvextend -l +8G /dev/gafis-vg/root
   # 全部
   lvextend -l +100%free /dev/gafis-vg/root
   ```

7. 查看扩容之后的逻辑卷

   ```
   lvdisplay /dev/gafis-vg/root
   ```

8. 文件系统扩容

   ```
   resize2fs /dev/mapper/gafis--vg-root
   ```

9. 查看

   ```
   df -h
   ```

#### ssh

##### 免密

1. 生成密钥

   ```
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   ```

2. 将公钥追加到认证文件中(本机免密)

   ```
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   ```

3. 将公钥追加到其他机器

   ```
   ssh-copy-id root@remote_ip
   ```

##### 免密2

1. 生成密钥

   ```
   ssh-keygen -t rsa -C "pass"
   文件名: 回车
   密码: pass
   ```

2. 将公钥追加到其他机器

   ```
   ssh-copy-id root@remote_ip
   ```

3. ssh-agent

   ```
   ssh-agent bash
   ssh-add .ssh/id_rsa
   输入密码
   ```

#### 防火墙

-- firewalld默认开放ssh服务, 默认规则都添加到zone=public
	参数
--zone #作用域
--add-port=80/tcp #添加端口，格式为：端口/通讯协议
--permanent #永久生效，没有此参数重启后失效

- 开放http(80)服务

  ```
  firewall-cmd --add-service=http
  ```

- 永久开放http服务

  ```
  firewall-cmd --add-service=http --permanent
  ```

- 永久关闭

  ```
  firewall-cmd --remove-service=http --permanent
  ```

- 重启防火墙生效

  ```
  firewall-cmd --reload
  ```

- 查看

  ```
  # 所有
  firewall-cmd --list-all
  # 端口
  firewall-cmd --list-ports
  # 服务
  firewall-cmd --list-services
  ```

- 查询服务启用状态

  ```
  firewall-cmd --query-service http
  ```

- 永久开放端口

  ```
  firewall-cmd --add-port=3128/tcp --permanent
  ```

#### ulimit（最大进程数和最大文件打开数)

1. 查看

   ```
   # 最大文件打开数
   ulimit -n
   # 最大进程数
   ulimit -u
   ```

2. 临时修改（重启后失效）

   ```
   ulimit -n 204800
   ulimit -u 204800
   ```

3. 永久修改，修改 /etc/security/limits.conf 文件，在文件末尾添加

   ```
   * soft nofile 204800
   * hard nofile 204800
   * soft nproc 204800
   * hard nproc 204800
    
   *          代表针对所有用户 
   noproc     是代表最大进程数 
   nofile     是代表最大文件打开数
   ```


4. 简化

   ```
   echo "*   soft    nofile  65535" >> /etc/security/limits.conf
   echo "*   hard    nofile  65535" >> /etc/security/limits.conf
   echo "*   soft    nproc  65535" >> /etc/security/limits.conf
   echo "*   hard    nproc  65535" >> /etc/security/limits.conf
   ```

#### vm.max_map_count

1. 查看

   ```
   sysctl -a | grep vm.max_map_count
   ```

2. 临时修改（重启后失效）

   ```
   sysctl -w vm.max_map_count=262144
   ```

3. 永久修改，修改 /etc/sysctl.conf 文件，在文件末尾添加

   ```
   vm.max_map_count=262144
   ```

4. 永久生效

   ```
   sysctl -p
   ```

#### 后台守护进程

```
setsid myscript.sh >/dev/null 2>&1 &
# 输出到日志文件
setsid myscript.sh >/path/to/logfile 2>&1 &
```

#### swap

##### 添加swap分区

```
# 添加8G的swap分区
dd if=/dev/zero of=/home/swap bs=1024 count=8388608
mkswap /home/swap
swapon /home/swap
```

##### 禁用swap

```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

#### 禁用SELinux

1. 查看状态

   ```
   sestatus
   ```

2. 临时关闭

   ```
   sudo setenforce 0
   ```

3. 永久关闭

   ```
   sed -i 's/enforcing/disabled/g' /etc/selinux/config /etc/selinux/config
   ```


#### 监控文件夹

```
watch -n 0.1 ls <your_folder>
```

#### 查看CPU个数

```
## 方法1
grep -c ^processor /proc/cpuinfo
## 方法2
nproc
## 方法3
getconf _NPROCESSORS_ONLN
## 方法4
lscpu | grep 'CPU(s):' | head -1 | awk '{print $2}'
```

#### 代理上网

```
## /etc/profile
export http_proxy="http://proxy_username:proxy_password@proxy_ip:proxy_port"
```

#### 目录结构查看

```
## 目录
ls -R | grep ":$" | sed -e 's/:$//' -e 's/[^-][^\/]*\//--/g' -e 's/^/   /' -e 's/-/|/'
## 文件
find . | sed -e "s/[^-][^\/]*\//  |/g" -e "s/|\([^ ]\)/|-\1/"
```


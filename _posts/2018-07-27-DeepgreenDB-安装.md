---
title: DeepgreenDB 集群搭建
date: 2018-07-27 07:07:07
categories:
- Greenplum
tags:
- Deepgreen
- 集群搭建
---

# DeepgreenDB 集群搭建

系统版本：CentOS 7.2 KVM

DGDB 版本：[ deepgreendb.18.07.rh7.x86_64.180718.bin](https://s3.amazonaws.com/vitessedata/download/deepgreendb.18.07.rh7.x86_64.180718.bin) 

集群规划：1 主 1 备 5 从，计算节点  3 segment 3 mirror

## 系统设置

### 关闭 selinux

```shell
# 查看状态：
# sestatus
SELinuxstatus: disabled
# 如果与上面不一致，请编辑文件，并将状态改为disabled
# vim /etc/selinux/config 
SELINUX=disabled
```

### 关闭 iptables

```shell
service iptables status
service iptables stop
```

### 修改 hostname 和 host

```shell
vi /etc/hostname
# mdw1
# mdw2
# sdw1
...
vi /etc/hosts
# 集群所有主机映射
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

### 创建 dgadmin 用户

```shell
userdel -r dgadmin
groupdel dgadmin
groupadd dgadmin
useradd -g dgadmin dgadmin
passwd dgadmin
# input password
```

## 安装 DGDB

### 安装数据库文件

使用 dgadmin 用户

上传安装文件到 master 上（/home/dgadmin/）

运行 bin 文件，自动生成数据库文件夹及软连接

```shell
deepgreendb -> /home/dgadmin/deepgreendb.18.07.180718
deepgreendb.18.07.180718
```

### 添加环境变量

```shell
vi ~/.bashrc
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
source /home/dgadmin/deepgreendb/greenplum_path.sh
```

使立即生效

```shell
source ~/.bashrc
```

### 建立集群 ssh 通信

```shell
cd 
mkdir dgconfigs
# 包含了集群所有主机的网口对应的主机名（因为有可能是双网卡的服务器）
vi dgconfigs/hostfile_exkeys
# 包含了集群所有主机名
vi dgconfigs/hostfile
# 包含了所有segment主机名
vi dgconfigs/hostfile_segonly
```

使用 gpssh-exkeys 建立通信

```shell
gpssh-exkeys -f /home/dgadmin/dgconfigs/hostfile_exkeys
```

### 安装软件到各个节点

```shell
gpseginstall -f /home/dgadmin/dgconfigs/hostfile_exkeys -u dgadmin
# check
gpssh -f /home/dgadmin/dgconfigs/hostfile_exkeys -e ls -l $GPHOME
```

### 检查主机参数配置

此处可以在配置系统环境时先配置好

实际未做此优化，测试中未看到效果，后续再研究

```shell
gpcheck -h hostname
```

此时会提示一些系统配置不是 deepgreen 推荐配置，参照 `$GPHOME/etc/gpcheck.cnf` 修改

使用 root 用户

```shell
# 内核参数
vim /etc/sysctl.conf
kernel.shmmax = 500000000
kernel.shmmni = 4096
kernel.shmall = 4000000000
kernel.sem = 250 512000 100 2048
kernel.sysrq = 1
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.ip_forward = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.all.arp_filter = 1
net.ipv4.ip_local_port_range = 1025 65535  # 此处我设置了 10000 65535
net.core.netdev_max_backlog = 10000
vm.overcommit_memory = 2

#  系统的最大进程数和最大文件打开数限制
vim /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072

# I/O 调度器，使用 deadline
# SSD 一般使用 noop，此处未测试
# 在 centos 7 KVM 下默认是 deadline，但是查看始终是 none
dmesg | grep -i scheduler
cat /sys/block/[vdb]/queue/scheduler
# 修改方法 -- 临时
echo deadline > /sys/block/[vdb]/queue/scheduler
# 修改方法 -- 永久
grubby --update-kernel=ALL --args="elevator=deadline"
reboot
# 或者使用vi编辑器修改配置文件，添加 elevator=deadline
vi /etc/default/grub
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet elevator=deadline numa=off"
# 然后保存文件，重新编译配置文
BIOS-Based： grub2-mkconfig -o /boot/grub2/grub.cfg
UEFI-Based： grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg

# 设置磁盘的预读扇区（blockdev）值为 16384 
# 查看命令
/sbin/blockdev --getra /dev/[vdb]
# 设置命令
/sbin/blockdev --setra 16384 /dev/[vdb]
# 为防止重启失效，将配置写入/etc/rc.local
echo '/sbin/blockdev --setra 16384 /dev/[vdb]' >> /etc/rc.local

# 关闭THP(Transparent Huge Pages)
# 检查THP的启用状态
cat /sys/kernel/mm/transparent_hugepage/defrag
# [always] madvise never
cat /sys/kernel/mm/transparent_hugepage/enabled
# [always] madvise never
# 这样说明是启用状态
# 永久修改 -- 在 rc.local 下添加一下内容
vim /etc/rc.d/rc.local
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
  echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
reboot
```

## 集群初始化

使用 root 用户创建各个节点下的数据目录

```shell
# master / standby maseter
mkdir -p /data/dgdata/master
# segment
mkdir -p /data/dgdata/dgdatap1
mkdir -p /data/dgdata/dgdatap2
mkdir -p /data/dgdata/dgdatap3
mkdir -p /data/dgdata/dgdatam1
mkdir -p /data/dgdata/dgdatam2
mkdir -p /data/dgdata/dgdatam3
# all hosts
chown -R dgadmin /data/dgdata
```

使用 dgadmin 用户配置初始化文件并初始化系统

在 `$GPHOME/docs/cli_help/gpconfigs` 下有一些模板，如： `gpinitsystem_config` 

```shell
cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config ~/dgconfigs/
vi ~/dgconfig/gpinitsystem_config

# FILE NAME: gpinitsystem_config

# Configuration file needed by the gpinitsystem

################################################
#### REQUIRED PARAMETERS
################################################

#### Name of this Greenplum system enclosed in quotes.
ARRAY_NAME="Greenplum Data Platform"
CLUSTER_NAME="Greenplum Data Platform"

#### Naming convention for utility-generated data directories.
SEG_PREFIX=gpseg

#### Base number by which primary segment port numbers 
#### are calculated.
PORT_BASE=40000

#### File system location(s) where primary segment data directories 
#### will be created. The number of locations in the list dictate
#### the number of primary segments that will get created per
#### physical host (if multiple addresses for a host are listed in 
#### the hostfile, the number of segments will be spread evenly across
#### the specified interface addresses).
declare -a DATA_DIRECTORY=(/data/dgdata/dgdatap1 /data/dgdata/dgdatap2 /data/dgdata/dgdatap3)

#### OS-configured hostname or IP address of the master host.
MASTER_HOSTNAME=mdw1

#### File system location where the master data directory 
#### will be created.
MASTER_DIRECTORY=/data/dgdata/master

#### Port number for the master instance. 
MASTER_PORT=5432

#### Shell utility used to connect to remote hosts.
TRUSTED_SHELL=ssh

#### Maximum log file segments between automatic WAL checkpoints.
CHECK_POINT_SEGMENTS=8

#### Default server-side character set encoding.
ENCODING=UNICODE

################################################
#### OPTIONAL MIRROR PARAMETERS
################################################

#### Base number by which mirror segment port numbers 
#### are calculated.
MIRROR_PORT_BASE=50000

#### Base number by which primary file replication port 
#### numbers are calculated.
REPLICATION_PORT_BASE=41000

#### Base number by which mirror file replication port 
#### numbers are calculated. 
MIRROR_REPLICATION_PORT_BASE=51000

#### File system location(s) where mirror segment data directories 
#### will be created. The number of mirror locations must equal the
#### number of primary locations as specified in the 
#### DATA_DIRECTORY parameter.
declare -a MIRROR_DATA_DIRECTORY=(/data/dgdata/dgdatam1 /data/dgdata/dgdatam2 /data/dgdata/dgdatam3)


################################################
#### OTHER OPTIONAL PARAMETERS
################################################

#### Create a database of this name after initialization.
DATABASE_NAME=init_with_this_database

#### Specify the location of the host address file here instead of
#### with the the -h option of gpinitsystem.
MACHINE_LIST_FILE=/home/dgadmin/dgconfigs/hostfile_segonly

# Hosts to allow to connect to the QD (and Segment Instances)
# By default, allow everyone to connect (0.0.0.0/0)
IP_ALLOW=0.0.0.0/0

export MASTER_DATA_DIRECTORY
export TRUSTED_SHELL

# Keep max_connection settings to reasonable values for
# installcheck good execution.
DEFAULT_QD_MAX_CONNECT=25
QE_CONNECT_FACTOR=5
```

初始化系统

```shell
gpinitsystem -c gpinitsystem_config -s [standby_hostname]
```

添加环境变量

```shell
vim greenplum_path.sh
export MASTER_DATA_DIRECTORY=/data/dgdata/master/gpseg-1
export PGPORT=5432
export PGDATABASE=[database]  # 使用 psql 进入时的默认数据库
```

检查系统状态

```shell
gpstate
```

psql 查看节点状态

```sql
psql
select * from gp_segment_configuration order by dbid;
```

role 和 preferred_role 应该一致

mode 全为 s

status 全为 u

## 配置远程访问

```shell
# 修改 master 节点的数据库文件，添加远程访问ip及权限
vi /data/dgdata/master/gpseg-1/pg_hba.conf
host    all     dgadmin         10.10.0.0/16    md5

# 给 dgadmin 用户添加密码
psql
alter role dgadmin with password '...';
```


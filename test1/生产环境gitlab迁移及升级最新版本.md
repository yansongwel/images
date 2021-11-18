> 升级需求描述：
>
> ​		由于当前gitlab 12.3.5版本存在严重漏洞，导致黑客利用该漏洞提权当前主机管理员权限后部署了挖矿程序，是当前主机的cpu长时间使用率在100%居高不下，严重影响开发人员对gitlab的使用。临时解决方案，只允许公司的公网ip地址访问，进出入端口限制。（这种方法影响在家办公对gitlab的体验感，只开放了公司内部局域网）
>
> ​		永久解决方案使用最新的操作系统，安装相同gitlab版本然后把数据同步当前系统后，验证可用性后再升级到最新版本来修复漏洞。

![image-20211116111651770](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031195.png)

# 一、升级前准备工作：

## 1.1当前版本主机信息收集

```shell
1，腾讯云主机，本地原主机gitlab版本升级。

2，确认安装方式：
[root@BetaTWS1 ~]# rpm -qa gitlab-ce
gitlab-ce-12.3.5-ce.0.el7.x86_64


3，查看gitlab版本
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION

4，检查gitlab状态
gitlab-rake gitlab:check SANITIZE=true --trace
gitlab-rake gitlab:check
gitlab-rake gitlab:check SANITIZE=true


5，#系统版本确认
[root@BetaTWS1 ~]# cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)

6，#系统内存信息确认：
[root@BetaTWS1 ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           7.6G        2.9G        161M        206M        4.5G        4.2G
Swap:            0B          0B          0B

7，#内核版本确认
[root@BetaTWS1 ~]# uname -r
3.10.0-1127.el7.x86_64


8，服务停止前后端口检查（服务器上存在其他服务防止端口被占用问题）
[root@VM-16-13-centos packages]# netstat -nlpt #gitlab启动状态当前端口
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:28022           0.0.0.0:*               LISTEN      1533/sshd           
tcp        0      0 127.0.0.1:3000          0.0.0.0:*               LISTEN      21774/grafana-serve 
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1506/master         
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      21727/nginx: master 
tcp        0      0 0.0.0.0:8060            0.0.0.0:*               LISTEN      21727/nginx: master 
tcp        0      0 127.0.0.1:9121          0.0.0.0:*               LISTEN      20748/redis_exporte 
tcp        0      0 127.0.0.1:9090          0.0.0.0:*               LISTEN      20714/prometheus    
tcp        0      0 127.0.0.1:9187          0.0.0.0:*               LISTEN      21756/postgres_expo 
tcp        0      0 127.0.0.1:9093          0.0.0.0:*               LISTEN      20600/alertmanager  
tcp        0      0 127.0.0.1:9100          0.0.0.0:*               LISTEN      20691/node_exporter 
tcp        0      0 127.0.0.1:9229          0.0.0.0:*               LISTEN      21715/gitlab-workho 
tcp        0      0 127.0.0.1:9168          0.0.0.0:*               LISTEN      21734/puma 4.3.3.gi 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      21727/nginx: master 
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      21819/unicorn maste 
tcp        0      0 127.0.0.1:8082          0.0.0.0:*               LISTEN      21701/sidekiq 5.2.7 
tcp        0      0 127.0.0.1:9236          0.0.0.0:*               LISTEN      20619/gitaly        
tcp6       0      0 ::1:25                  :::*                    LISTEN      1506/master         
tcp6       0      0 :::9094                 :::*                    LISTEN      20600/alertmanager  
tcp6       0      0 ::1:9168                :::*                    LISTEN      21734/puma 4.3.3.gi 

[root@VM-16-13-centos packages]# gitlab-ctl stop 
ok: down: alertmanager: 1s, normally up
ok: down: gitaly: 0s, normally up
ok: down: gitlab-exporter: 0s, normally up
ok: down: gitlab-workhorse: 1s, normally up
ok: down: grafana: 0s, normally up
ok: down: logrotate: 1s, normally up
ok: down: nginx: 0s, normally up
ok: down: node-exporter: 1s, normally up
ok: down: postgres-exporter: 0s, normally up
ok: down: postgresql: 0s, normally up
ok: down: prometheus: 1s, normally up
ok: down: redis: 0s, normally up
ok: down: redis-exporter: 1s, normally up
ok: down: sidekiq: 1s, normally up
ok: down: unicorn: 0s, normally up

[root@VM-16-13-centos packages]# gitlab-ctl status
down: alertmanager: 19s, normally up; run: log: (pid 1564) 235301s
down: gitaly: 18s, normally up; run: log: (pid 1581) 235301s
down: gitlab-exporter: 18s, normally up; run: log: (pid 1563) 235301s
down: gitlab-workhorse: 18s, normally up; run: log: (pid 1562) 235301s
down: grafana: 17s, normally up; run: log: (pid 1574) 235301s
down: logrotate: 17s, normally up; run: log: (pid 1560) 235301s
down: nginx: 16s, normally up; run: log: (pid 1575) 235301s
down: node-exporter: 16s, normally up; run: log: (pid 1571) 235301s
down: postgres-exporter: 15s, normally up; run: log: (pid 1559) 235301s
down: postgresql: 15s, normally up; run: log: (pid 1587) 235301s
down: prometheus: 15s, normally up; run: log: (pid 1561) 235301s
down: redis: 14s, normally up; run: log: (pid 1598) 235301s
down: redis-exporter: 14s, normally up; run: log: (pid 1565) 235301s
down: sidekiq: 13s, normally up; run: log: (pid 1580) 235301s
down: unicorn: 11s, normally up; run: log: (pid 1586) 235301s

[root@VM-16-13-centos packages]# netstat -nlpt 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:28022           0.0.0.0:*               LISTEN      1533/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1506/master         
tcp6       0      0 ::1:25  



#gitlab认证文件
/etc/gitlab/gitlab-secrets.json
#gitlab配置文件
/etc/gitlab/gitlab.rb   

#默认情况下，GitLab 管理authorized_keys位于 git用户主目录中的文件。对于大多数安装，这将位于 下 /var/opt/gitlab/.ssh/authorized_keys，但您可以使用以下命令authorized_keys在您的系统上找到：
getent passwd git | cut -d: -f6 | awk '{print $1"/.ssh/authorized_keys"}'
该authorized_keys文件包含允许访问 GitLab 的用户的所有公共 SSH 密钥。但是，为了维护单一的事实来源，需要配置Geo以通过数据库查找执行 SSH 指纹查找。

```

## 1.2当前主机服务检查

```shell
# 查看服务状态 
gitlab-ctl status 

gitlab-rake gitlab:check SANITIZE=true --trace
gitlab-rake gitlab:check
gitlab-rake gitlab:check SANITIZE=true

```





## 1.3当前主机配置及数据备份

```shell
#默认情况下，GitLab 管理authorized_keys位于 git用户主目录中的文件。对于大多数安装，这将位于 下 /var/opt/gitlab/.ssh/authorized_keys，但您可以使用以下命令authorized_keys在您的系统上找到：
getent passwd git | cut -d: -f6 | awk '{print $1"/.ssh/authorized_keys"}'
该authorized_keys文件包含允许访问 GitLab 的用户的所有公共 SSH 密钥。但是，为了维护单一的事实来源，需要配置Geo以通过数据库查找执行 SSH 指纹查找。

#备份相关文件目录
/var/opt/gitlab/.ssh/authorized_keys
/etc/gitlab/gitlab.rb
/etc/gitlab/gitlab-secrets.json
/etc/postfix/main.cfpostfix 邮件配置备份

#nginx相关文件备份
检查默认nginx配置文件，并迁移至新Nginx服务
/var/opt/gitlab/nginx/conf/nginx.conf #nginx配置文件,包含gitlab-http.conf文件
/var/opt/gitlab/nginx/conf/gitlab-http.conf #gitlab核心nginx配置文件

/etc/nginx/conf.d/test-betawm-com.conf

/beta/ssl/5419901__betawm.com.pem
/beta/ssl/5419901__betawm.com.key
/beta/ssl/star.betawm.com/ca_bundle.crt
/beta/ssl/star.betawm.com/certificate.crt
/beta/ssl/star.betawm.com/private.key

#配置文件内容查看
[root@BetaTWS1 gitlab]# egrep -v "^$|^#" gitlab.rb
external_url 'https://git.betawm.com'
gitlab_rails['lfs_enabled'] = true
gitlab_rails['lfs_storage_path'] = "/beta/gitlab/lfs-objects"
git_data_dirs({
   "default" => {
     "path" => "/beta/gitlab/git-data"
    }
 })
gitlab_rails['gitlab_shell_ssh_port'] = 28022
gitlab_rails['dir'] = "/beta/gitlab/gitlab-rails"
gitlab_rails['log_directory'] = "/beta/gitlab/log/gitlab-rails"
gitlab_rails['uploads_directory'] = "/beta/gitlab/gitlab-rails/uploads"
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/beta/ssl/5419901__betawm.com.pem"
nginx['ssl_certificate_key'] = "/beta/ssl/5419901__betawm.com.key"
nginx['custom_nginx_config'] = "include /etc/nginx/conf.d/test-betawm-com.conf;"


#备份数据：
gitlab-backup create STRATEGY=copy 
#默认备份目录：
/var/opt/gitlab/backups/


# 文件及数据备份
[root@BetaTWS1 tmp]# mkdir git-backup
[root@BetaTWS1 tmp]# cd git-backup/
[root@BetaTWS1 git-backup]# cp /var/opt/gitlab/.ssh/authorized_keys .
[root@BetaTWS1 git-backup]# cp /etc/gitlab/gitlab.rb .
[root@BetaTWS1 git-backup]# cp /etc/gitlab/gitlab-secrets.json .
[root@BetaTWS1 git-backup]# cp /beta/ssl/5419901__betawm.com.pem .
[root@BetaTWS1 git-backup]# cp /beta/ssl/5419901__betawm.com.key .
[root@BetaTWS1 git-backup]# cp /etc/nginx/conf.d/test-betawm-com.conf .
[root@BetaTWS1 git-backup]# gitlab-backup create STRATEGY=copy 
2021-11-16 13:23:43 +0800 -- Dumping database ... 
Dumping PostgreSQL database gitlabhq_production ... [DONE]
2021-11-16 13:23:47 +0800 -- done
2021-11-16 13:23:47 +0800 -- Dumping repositories ...
 * knowledge/android (@hashed/4e/07/4e07408562bedb8b60ce05c1decfe3ad16b72230967de01f640b7e4729b49fce) ... [SKIPPED]
[DONE] Wiki
 * knowledge/ios (@hashed/4b/22/4b227777d4dd1fc61c6f884f48641d02b4d121d3fd328cb08b5531fcacdabf8a) ... [SKIPPED]
[DONE] Wiki
 * knowledge/service (@hashed/ef/2d/ef2d127de37b942baad06145e54b0c619a1f22327b2ebbcfbec78f5564afe39d) ... [SKIPPED]
 .........
  * tech/Beta.ShortVideoWeb (@hashed/48/a1/48a1706eca5ee6148f748ca91a0f7db6ebcf59943532044a7bf60bbe44e5b1d2) ... [DONE]
[SKIPPED] Wiki
2021-11-16 13:30:35 +0800 -- done
2021-11-16 13:30:35 +0800 -- Dumping uploads ... 
2021-11-16 13:30:36 +0800 -- done
2021-11-16 13:30:36 +0800 -- Dumping builds ... 
2021-11-16 13:30:36 +0800 -- done
2021-11-16 13:30:36 +0800 -- Dumping artifacts ... 
2021-11-16 13:30:36 +0800 -- done
2021-11-16 13:30:36 +0800 -- Dumping pages ... 
2021-11-16 13:30:36 +0800 -- done
2021-11-16 13:30:36 +0800 -- Dumping lfs objects ... 
2021-11-16 13:30:40 +0800 -- done
2021-11-16 13:30:40 +0800 -- Dumping container registry images ... 
2021-11-16 13:30:40 +0800 -- [DISABLED]
Creating backup archive: 1637040640_2021_11_16_12.3.5_gitlab_backup.tar ... done
Uploading backup archive to remote storage  ... skipped
Deleting tmp directories ... done
done
done
done
done
done
done
done
Deleting old backups ... skipping
Warning: Your gitlab.rb and gitlab-secrets.json files contain sensitive data 
and are not included in this backup. You will need these files to restore a backup.
Please back them up manually.
Backup task is done.

[root@BetaTWS1 git-backup]# ll /var/opt/gitlab/backups/
total 33065368
-rw------- 1 git git 16805560320 Nov  5 15:32 1636097380_2021_11_05_12.3.5_gitlab_backup.tar
-rw------- 1 git git 17053368320 Nov 16 13:33 1637040640_2021_11_16_12.3.5_gitlab_backup.tar
[root@BetaTWS1 git-backup]# mv /var/opt/gitlab/backups/1637040640_2021_11_16_12.3.5_gitlab_backup.tar .
[root@BetaTWS1 git-backup]# ll
total 16653860
-rw------- 1 git  git  17053368320 Nov 16 13:33 1637040640_2021_11_16_12.3.5_gitlab_backup.tar
-rw-r--r-- 1 root root        1675 Nov 16 13:22 5419901__betawm.com.key
-rw-r--r-- 1 root root        3847 Nov 16 13:22 5419901__betawm.com.pem
-rw------- 1 root root       51226 Nov 16 13:21 authorized_keys
-rw------- 1 root root       94956 Nov 16 13:21 gitlab.rb
-rw------- 1 root root       15615 Nov 16 13:22 gitlab-secrets.json
-rw-r--r-- 1 root root        2330 Nov 16 13:22 test-betawm-com.conf
[root@BetaTWS1 git-backup]# tar zcvf gitlab_12.3.5_backup.tar.gz ./
./
./gitlab.rb
./test-betawm-com.conf
./5419901__betawm.com.key
./5419901__betawm.com.pem
./gitlab-secrets.json
./authorized_keys
./1637040640_2021_11_16_12.3.5_gitlab_backup.tar
tar: .: file changed as we read it

#注意：数据备份完毕后，需要停止服务防止客户端连接数据传输，导致最后新版的gitlab数据不一致问题。
gitlab-ctl stop
gitlab-ctl status
```

## 1.4升级包准备

```shell
#在线安装添加yum源：
vi /etc/yum.repos.d/gitlab-ce.repo 
//添加如下内容 
[gitlab-ce] 
name=Gitlab CE Repository 
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/ 
gpgcheck=0 
enabled=1 

# 更新本地yum缓存
yum makecache fast 
yum --showduplicates list gitlab-ce

#离线安装：
官方下载地址：
https://packages.gitlab.com/app/gitlab/gitlab-ce/search?q=

gitlab-ce-12.3.5-ce.0.el7.x86_64.rpm		MD5	f93cc255ef3ba67887edd631d487f301
gitlab-ce-12.10.14-ce.0.el7.x86_64.rpm  	MD5	78a9eb82f598c516676b45e567d3b478
gitlab-ce-13.0.14-ce.0.el7.x86_64.rpm		MD5	cb8c595cc21bb01f6ce9a305b833f2a9
gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm		MD5	241aa71a57313679a5b396780d453b18
gitlab-ce-13.8.8-ce.0.el7.x86_64.rpm		MD5	9aec490bcc403759c79d1fa5c98c9155
gitlab-ce-13.12.15-ce.0.el7.x86_64.rpm		MD5	3553267d315c29d10a267833d499a0d6
gitlab-ce-14.0.12-ce.0.el7.x86_64.rpm		MD5	479f77326f25619e55ba50ef2cd7ff42
gitlab-ce-14.4.2-ce.0.el7.x86_64.rpm		MD5	2b8b781598aa0219e75d32cfac1f7b9f
```

![image-20211116133438896](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031196.png)

## 1.5新机器磁盘格式化

```shell
#磁盘创建8e类型分区--->partprobe --->创建物理卷(pv)--->创建卷组(vg)--->创建逻辑卷(lv)--->格式化磁盘--->挂载数据目录--->添加fstab

[root@VM-16-9-centos ~]# cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)

[root@VM-16-9-centos ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G        267M        2.3G        624K        1.2G        3.2G
Swap:            0B          0B          0B

[root@VM-16-9-centos ~]# lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 142.4M  0 rom  
vda    253:0    0   100G  0 disk 
└─vda1 253:1    0   100G  0 part /
vdb    253:16   0   500G  0 disk 

[root@VM-16-9-centos ~]# mkdir /beta

[root@VM-16-9-centos ~]# fdisk -l

Disk /dev/vda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0009ac89

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048   209715166   104856559+  83  Linux

Disk /dev/vdb: 536.9 GB, 536870912000 bytes, 1048576000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

[root@VM-16-9-centos ~]# fdisk /dev/vdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x2cd28c2a.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-1048575999, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-1048575999, default 1048575999): 
Using default value 1048575999
Partition 1 of type Linux and of size 500 GiB is set

Command (m for help): p

Disk /dev/vdb: 536.9 GB, 536870912000 bytes, 1048576000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x2cd28c2a

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048  1048575999   524286976   83  Linux

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): p

Disk /dev/vdb: 536.9 GB, 536870912000 bytes, 1048576000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x2cd28c2a

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048  1048575999   524286976   8e  Linux LVM

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@VM-16-9-centos ~]# lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 142.4M  0 rom  
vda    253:0    0   100G  0 disk 
└─vda1 253:1    0   100G  0 part /
vdb    253:16   0   500G  0 disk 
└─vdb1 253:17   0   500G  0 part 

#使kernel重新读取分区表
[root@VM-16-9-centos gitlab]# partprobe /dev/vdb1
 对于 /dev/sda 的警告不予理会


#创建pv
[root@VM-16-9-centos gitlab]# pvcreate /dev/vdb1
  Physical volume "/dev/vdb1" successfully created.
#查看PV
[root@VM-16-9-centos gitlab]# pvdisplay 
  "/dev/vdb1" is a new physical volume of "<500.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/vdb1
  VG Name               
  PV Size               <500.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               xi6xJa-1XRT-Kr7Z-uL7x-Pe6g-bsMk-46LoX2
#扫描系统pv
  [root@VM-16-9-centos gitlab]# pvscan 
  PV /dev/vdb1                      lvm2 [<500.00 GiB]
  Total: 1 [<500.00 GiB] / in use: 0 [0   ] / in no VG: 1 [<500.00 GiB]

# 创建VG：
[root@VM-16-9-centos gitlab]# vgcreate vg_gitlab /dev/vdb1
  Volume group "vg_gitlab" successfully created
# 查看VG：vgdisplay
[root@VM-16-9-centos gitlab]# vgdisplay 
  --- Volume group ---
  VG Name               vg_gitlab
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <500.00 GiB
  PE Size               4.00 MiB
  Total PE              127999
  Alloc PE / Size       0 / 0   
  Free  PE / Size       127999 / <500.00 GiB
  VG UUID               4vpUyP-TLRt-2Kro-q3RG-UD3P-94Uf-OX9FNQ
# 扫面系统VG：vgscan   
[root@VM-16-9-centos gitlab]# vgscan 
  Reading volume groups from cache.
  Found volume group "vg_gitlab" using metadata type lvm2
 
# 创建LV:
[root@VM-16-9-centos gitlab]# lvcreate -l 127999 -n lv_gitlab vg_gitlab  #127999是VG中PE的个数
  Logical volume "lv_gitlab" created.
  
# 查看LV
[root@VM-16-9-centos gitlab]# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/vg_gitlab/lv_gitlab
  LV Name                lv_gitlab
  VG Name                vg_gitlab
  LV UUID                0AZSDj-FgR9-EUkv-TAX3-Vle6-0BOT-ALlGms
  LV Write Access        read/write
  LV Creation host, time VM-16-9-centos, 2021-11-16 15:23:53 +0800
  LV Status              available
  # open                 0
  LV Size                <500.00 GiB
  Current LE             127999
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           252:0

#扫面系统LV
[root@VM-16-9-centos gitlab]# lvscan 
  ACTIVE            '/dev/vg_gitlab/lv_gitlab' [<500.00 GiB] inherit
  
#格式化刚刚创建的LV
[root@VM-16-9-centos gitlab]# mkfs -t ext4 /dev/vg_gitlab/lv_gitlab
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
32768000 inodes, 131070976 blocks
6553548 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2279604224
4000 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
        102400000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done     

[root@VM-16-9-centos gitlab]# lsblk 
NAME                    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0                      11:0    1 142.4M  0 rom  
vda                     253:0    0   100G  0 disk 
└─vda1                  253:1    0   100G  0 part /
vdb                     253:16   0   500G  0 disk 
└─vdb1                  253:17   0   500G  0 part 
  └─vg_gitlab-lv_gitlab 252:0    0   500G  0 lvm  

#挂载磁盘
[root@VM-16-9-centos gitlab]# mount /dev/vg_gitlab/lv_gitlab /beta/
[root@VM-16-9-centos gitlab]# df -Th
Filesystem                      Type      Size  Used Avail Use% Mounted on
devtmpfs                        devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs                           tmpfs     1.9G   24K  1.9G   1% /dev/shm
tmpfs                           tmpfs     1.9G  600K  1.9G   1% /run
tmpfs                           tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/vda1                       ext4       99G  8.3G   86G   9% /
tmpfs                           tmpfs     379M     0  379M   0% /run/user/0
/dev/mapper/vg_gitlab-lv_gitlab ext4      493G   73M  467G   1% /beta

[root@VM-16-9-centos gitlab]# blkid 
/dev/sr0: UUID="2021-11-16-14-01-59-00" LABEL="config-2" TYPE="iso9660" 
/dev/vda1: UUID="4b499d76-769a-40a0-93dc-4a31a59add28" TYPE="ext4" 
/dev/vdb1: UUID="9K6mDt-67Tv-Y3Lb-eA08-WJY5-a6YX-posxi4" TYPE="LVM2_member" 
/dev/mapper/vg_gitlab-lv_gitlab: UUID="2b3bc092-9896-458c-9ab5-6e2673e0e625" TYPE="ext4"  #这个是我们需要uuid
[root@VM-16-9-centos gitlab]# vim /etc/fstab 
[root@VM-16-9-centos gitlab]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Thu Mar  7 06:38:37 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=4b499d76-769a-40a0-93dc-4a31a59add28 /                       ext4    defaults        1 1
UUID=2b3bc092-9896-458c-9ab5-6e2673e0e625 /beta                       ext4    defaults        1 2

#重新挂载所有盘符
[root@VM-16-9-centos beta]# mount -a
```

## 1.6数据上传md5校验

```shell
[root@VM-16-9-centos gitlab]# pwd
/tmp/gitlab
[root@VM-16-9-centos gitlab]# ll
total 15498380
-rw-r--r-- 1 root root 11044782080 Nov 16 15:42 gitlab_12.3.5_backup.tar.gz
-rw-r--r-- 1 root root   797740292 Nov 16 14:59 gitlab-ce-12.10.14-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   696213194 Nov 16 14:54 gitlab-ce-12.3.5-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   728656981 Nov 16 14:59 gitlab-ce-13.0.14-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   736750197 Nov 16 15:02 gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   911634590 Nov 16 15:03 gitlab-ce-14.0.12-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   954527068 Nov 16 15:06 gitlab-ce-14.4.2-ce.0.el7.x86_64.rpm

[root@VM-16-9-centos gitlab]# md5sum gitlab-ce-12.3.5-ce.0.el7.x86_64.rpm 
f93cc255ef3ba67887edd631d487f301  gitlab-ce-12.3.5-ce.0.el7.x86_64.rpm
[root@VM-16-9-centos gitlab]# md5sum gitlab-ce-12.10.14-ce.0.el7.x86_64.rpm 
78a9eb82f598c516676b45e567d3b478  gitlab-ce-12.10.14-ce.0.el7.x86_64.rpm
[root@VM-16-9-centos gitlab]# md5sum gitlab-ce-13.0.14-ce.0.el7.x86_64.rpm 
cb8c595cc21bb01f6ce9a305b833f2a9  gitlab-ce-13.0.14-ce.0.el7.x86_64.rpm
[root@VM-16-9-centos gitlab]# md5sum gitlab-ce-13.8.8-ce.0.el7.x86_64.rpm 
9aec490bcc403759c79d1fa5c98c9155  gitlab-ce-13.8.8-ce.0.el7.x86_64.rpm
[root@VM-16-9-centos gitlab]# md5sum gitlab-ce-13.12.15-ce.0.el7.x86_64.rpm 
3553267d315c29d10a267833d499a0d6  gitlab-ce-13.12.15-ce.0.el7.x86_64.rpm
[root@VM-16-9-centos gitlab]# md5sum gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm 
241aa71a57313679a5b396780d453b18  gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm
[root@VM-16-9-centos gitlab]# md5sum gitlab-ce-14.0.12-ce.0.el7.x86_64.rpm 
479f77326f25619e55ba50ef2cd7ff42  gitlab-ce-14.0.12-ce.0.el7.x86_64.rpm
[root@VM-16-9-centos gitlab]# md5sum gitlab-ce-14.4.2-ce.0.el7.x86_64.rpm 
2b8b781598aa0219e75d32cfac1f7b9f  gitlab-ce-14.4.2-ce.0.el7.x86_64.rpm
```



## 1.7升级路线规划

```
当前gitlab版本号：12.3.5
官方路线：
12.1.17-> 12.10.14 -> 13.0.14 -> 13.1.11 -> 13.8.8 -> latest 13.12.Z -> latest 14.0.Z -> latest 14.Y.Z
我的路线
12.3.5 -> 12.10.14 -> 13.0.14 -> 13.1.11 -> 13.8.8 -> 13.12.15 -> 14.0.12 -> 14.4.2
```

# 二、新主机原版本部署及数据恢复

## 2.1创建相关目录

```
[root@VM-16-9-centos gitlab]# mkdir /beta/{gitlab,ssl}
[root@VM-16-9-centos gitlab]# mkdir /beta/ssl/star.betawm.com/
[root@VM-16-9-centos gitlab]# mkdir /beta/gitlab/git-data
```

## 2.2依赖包安装

```
[root@VM-16-9-centos gitlab]# yum -y install epel-release
[root@VM-16-9-centos gitlab]# yum -y install policycoreutils-python
```

## 2.3原版本安装

```shell
[root@VM-16-9-centos gitlab]# rpm -ivh gitlab-ce-12.3.5-ce.0.el7.x86_64.rpm 
warning: gitlab-ce-12.3.5-ce.0.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID f27eab47: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:gitlab-ce-12.3.5-ce.0.el7        ################################# [100%]
It looks like GitLab has not been configured yet; skipping the upgrade script.

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.
  


     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/
  

Thank you for installing GitLab!
GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

```

## 2.4备份文件恢复

```shell
[root@VM-16-9-centos gitlab]# ll
total 23008024
-rw-r--r-- 1 root root 16960835248 Nov 16 15:56 gitlab_12.3.5_backup.tar.gz
-rw-r--r-- 1 root root   797740292 Nov 16 14:59 gitlab-ce-12.10.14-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   696213194 Nov 16 14:54 gitlab-ce-12.3.5-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   728656981 Nov 16 14:59 gitlab-ce-13.0.14-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   736750197 Nov 16 15:02 gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   906240064 Nov 16 15:50 gitlab-ce-13.12.15-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   867568698 Nov 16 15:53 gitlab-ce-13.8.8-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   911634590 Nov 16 15:03 gitlab-ce-14.0.12-ce.0.el7.x86_64.rpm
-rw-r--r-- 1 root root   954527068 Nov 16 15:06 gitlab-ce-14.4.2-ce.0.el7.x86_64.rpm
[root@VM-16-9-centos gitlab]# tar zxvf gitlab_12.3.5_backup.tar.gz 
./
./gitlab.rb
./test-betawm-com.conf
./5419901__betawm.com.key
./5419901__betawm.com.pem
./gitlab-secrets.json
./authorized_keys
./1637040640_2021_11_16_12.3.5_gitlab_backup.tar
[root@VM-16-9-centos gitlab]# cd /etc/gitlab/
[root@VM-16-9-centos gitlab]# ls
gitlab.rb
[root@VM-16-9-centos gitlab]# mv gitlab.rb gitlab.rb.bak
[root@VM-16-9-centos gitlab]# cd /tmp/gitlab/
[root@VM-16-9-centos gitlab]# ls
1637040640_2021_11_16_12.3.5_gitlab_backup.tar  gitlab_12.3.5_backup.tar.gz             gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm   gitlab-ce-14.4.2-ce.0.el7.x86_64.rpm
5419901__betawm.com.key                         gitlab-ce-12.10.14-ce.0.el7.x86_64.rpm  gitlab-ce-13.12.15-ce.0.el7.x86_64.rpm  gitlab.rb
5419901__betawm.com.pem                         gitlab-ce-12.3.5-ce.0.el7.x86_64.rpm    gitlab-ce-13.8.8-ce.0.el7.x86_64.rpm    gitlab-secrets.json
authorized_keys                                 gitlab-ce-13.0.14-ce.0.el7.x86_64.rpm   gitlab-ce-14.0.12-ce.0.el7.x86_64.rpm   test-betawm-com.conf
[root@VM-16-9-centos gitlab]# cp gitlab.rb /etc/gitlab/
[root@VM-16-9-centos gitlab]# cp gitlab-secrets.json /etc/gitlab/
[root@VM-16-9-centos gitlab]# cp 5419901__betawm.com.* /beta/ssl/
[root@VM-16-9-centos gitlab]# mkdir /etc/nginx/conf.d/ -p
[root@VM-16-9-centos gitlab]# cp test-betawm-com.conf /etc/nginx/conf.d/  

#下面三个配置文件是遗漏备份恢复文件后面加的：
[root@VM-16-9-centos gitlab]# cp ca_bundle.crt /beta/ssl/star.betawm.com/
[root@VM-16-9-centos gitlab]# cp certificate.crt /beta/ssl/star.betawm.com/
[root@VM-16-9-centos gitlab]# cp private.key /beta/ssl/star.betawm.com/
```

## 2.5初始化gitlab

```shell
#重新配置并启动
[root@VM-16-9-centos gitlab]# gitlab-ctl reconfigure
#查看当前服务状态
[root@VM-16-9-centos gitlab]# gitlab-ctl status
run: alertmanager: (pid 27292) 56471s; run: log: (pid 624) 61370s
run: gitaly: (pid 27319) 56470s; run: log: (pid 32090) 61482s
run: gitlab-exporter: (pid 27340) 56470s; run: log: (pid 477) 61384s
run: gitlab-workhorse: (pid 27348) 56470s; run: log: (pid 32572) 61429s
run: grafana: (pid 27361) 56469s; run: log: (pid 735) 61359s
run: logrotate: (pid 3624) 2468s; run: log: (pid 375) 61396s
run: nginx: (pid 11073) 817s; run: log: (pid 32619) 61422s	
run: node-exporter: (pid 27605) 56438s; run: log: (pid 437) 61390s
run: postgres-exporter: (pid 27611) 56437s; run: log: (pid 684) 61364s
run: postgresql: (pid 27626) 56437s; run: log: (pid 32253) 61478s
run: prometheus: (pid 27635) 56436s; run: log: (pid 572) 61376s
run: redis: (pid 27646) 56436s; run: log: (pid 32039) 61485s
run: redis-exporter: (pid 27651) 56435s; run: log: (pid 526) 61380s
run: sidekiq: (pid 27667) 56433s; run: log: (pid 32529) 61434s
run: unicorn: (pid 27680) 56432s; run: log: (pid 32495) 61440s
```

> #页面访问验证（第一次访问要设置root密码，要符合密钥长度安全）

![image-20211117095715296](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031197.png)

> #版本验证

![image-20211117095859800](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031198.png)

## 2.6备份数据恢复

```shell
#1，此过程假设：
您已经安装了与创建备份完全相同的 GitLab Omnibus版本和类型 (CE/EE)。
你sudo gitlab-ctl reconfigure至少跑过一次。
GitLab 正在运行。如果没有，请使用sudo gitlab-ctl start.

#2，首先确保您的备份 tar 文件位于gitlab.rb配置中描述的备份目录中 gitlab_rails['backup_path']。默认值为 /var/opt/gitlab/backups. 它需要归git用户所有。
sudo cp 11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar /var/opt/gitlab/backups/
sudo chown git.git /var/opt/gitlab/backups/11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar

#3，停止连接到数据库的进程。让 GitLab 的其余部分保持运行：
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq

#4，# Verify
sudo gitlab-ctl status

#5，接下来，恢复备份，指定要恢复的备份的时间戳：
# This command will overwrite the contents of your GitLab database!
sudo gitlab-backup restore BACKUP=11493107454_2018_04_25_10.6.4-ce

#6，注意事项
GitLab 12.1 及更早版本的用户应改用该命令gitlab-rake gitlab:backup:restore。
gitlab-rake gitlab:backup:restore没有在您的注册表目录上设置正确的文件系统权限。这是一个已知问题。在 GitLab 12.2 或更高版本中，您可以使用gitlab-backup restore来避免此问题。
如果您的备份 tar 文件和已安装的 GitLab 版本之间存在 GitLab 版本不匹配，则恢复命令将中止并显示错误消息。安装正确的 GitLab 版本，然后重试。
当您的安装使用 PgBouncer 时，restore 命令需要额外的参数，无论是出于性能原因还是与 Patroni 集群一起使用时。

#7，接下来，/etc/gitlab/gitlab-secrets.json如有必要，恢复， 如前所述。
重新配置、重启并检查 GitLab：
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
sudo gitlab-rake gitlab:check SANITIZE=true

#8，在 GitLab 13.1 及更高版本中，检查数据库值可以被解密， 尤其是/etc/gitlab/gitlab-secrets.json在恢复时，或者如果不同的服务器是恢复的目标。
sudo gitlab-rake gitlab:doctor:secrets
```

### 2.6.1数据恢复后验证：

```shell
#数据恢复过程较长请耐心等待：
[root@VM-16-9-centos backups]# gitlab-backup restore BACKUP=1637040640_2021_11_16_12.3.5
Unpacking backup ... done
Before restoring the database, we will remove all existing
tables to avoid future upgrade problems. Be aware that if you have
custom tables in the GitLab database these tables and all data will be
removed.

Do you want to continue (yes/no)? yes

......
 * wpeng/Beta.Serial ... [DONE]
 * tech/Beta.ShortVideoWeb ... [DONE]
2021-11-17 10:29:40 +0800 -- done
2021-11-17 10:29:40 +0800 -- Restoring uploads ... 
2021-11-17 10:29:40 +0800 -- done
2021-11-17 10:29:40 +0800 -- Restoring builds ... 
2021-11-17 10:29:40 +0800 -- done
2021-11-17 10:29:40 +0800 -- Restoring artifacts ... 
2021-11-17 10:29:40 +0800 -- done
2021-11-17 10:29:40 +0800 -- Restoring pages ... 
2021-11-17 10:29:40 +0800 -- done
2021-11-17 10:29:40 +0800 -- Restoring lfs objects ... 
2021-11-17 10:29:41 +0800 -- done
This task will now rebuild the authorized_keys file.
You will lose any data stored in the authorized_keys file.
Do you want to continue (yes/no)? Do you want to continue (yes/no)? Do you want to continue (yes/no)? yes
Do you want to continue (yes/no)? yes

Deleting tmp directories ... done
done
done
done
done
done
done
done
Warning: Your gitlab.rb and gitlab-secrets.json files contain sensitive data 
and are not included in this backup. You will need to restore these files manually.
Restore task is done.


#GitLab 管理authorized_keys恢复：
[root@VM-16-9-centos .ssh]# mkdir ~/gitlab-check-backup-1637117003
[root@VM-16-9-centos .ssh]# sudo mv /var/opt/gitlab/.ssh/authorized_keys ~/gitlab-check-backup-1637117003
[root@VM-16-9-centos .ssh]# cp /tmp/gitlab/authorized_keys /var/opt/gitlab/.ssh/
[root@VM-16-9-centos .ssh]# chown git.git /var/opt/gitlab/.ssh/authorized_keys  #非常关键

#没有上面的授权，每次检查都会报访问不到密钥文件错误
[root@VM-16-9-centos .ssh]# gitlab-rake gitlab:check SANITIZE=true  #直到检查没有任何报错为止。
5/308 ... yes
Redis version >= 2.8.0? ... yes
Ruby version >= 2.5.3 ? ... yes (2.6.3)
Git version >= 2.22.0 ? ... yes (2.22.0)
Git user has default SSH configuration? ... yes
Active users: ... 106
Is authorized keys file accessible? ... no  
Trying to fix error automatically. ...Failed
  Try fixing it:
  sudo chmod 700 /var/opt/gitlab/.ssh
  touch /var/opt/gitlab/.ssh/authorized_keys
  sudo chmod 600 /var/opt/gitlab/.ssh/authorized_keys
  Please fix the error above and rerun the checks.

Checking GitLab App ... Finished


Checking GitLab subtasks ... Finished
```

> 报错：
>
> Is authorized keys file accessible? ... no  
> Trying to fix error automatically. ...Failed
>
> > 解决方法：
> >

`[root@VM-16-9-centos .ssh]# chown git.git /var/opt/gitlab/.ssh/authorized_keys` 

![image-20211117110316872](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031199.png)

> #使用之前的账号验证登录：

![image-20211117130846550](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031200.png)

> #查看日志没有任何的报错信息即可

`[root@VM-16-9-centos .ssh]# gitlab-ctl tail`



## 2.7 优化添加每天自动备份（建议在最后升级完成后操作）

```shell
#限制本地文件的备份生命周期（修剪旧备份）
编辑/etc/gitlab/gitlab.rb： 
## Limit backup lifetime to 7 days - 604800 seconds
gitlab_rails['backup_keep_time'] = 604800   

#设置备份文件的保存位置
编辑/etc/gitlab/gitlab.rb： 
gitlab_rails['backup_path'] = "/beta/gitlab/backup"

#备份存档权限
编辑/etc/gitlab/gitlab.rb：
gitlab_rails['backup_archive_permissions'] = 0644 # Makes the backup archives world-readable

为root用户编辑 crontab ：
sudo su -
crontab -e
在那里，添加以下行以安排每天凌晨 2 点的备份：
0 2 * * * /opt/gitlab/bin/gitlab-backup create CRON=1
CRON=1如果没有任何错误，环境设置会指示备份脚本隐藏所有进度输出。建议这样做以减少 cron 垃圾邮件。

#gitlab 12.1 及更早版本的备份方式
# edited by ouyang 20211117 添加定时任务，每天凌晨两点，执行gitlab备份
0  2    * * *   root    /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1

#重启cron服务
systemctl restart crond.service
```



# 三、开始正式版本升级

## 3.1 12.3.5 -> 12.10.14升级

```shell
升级前检查：
 ~]# rpm -qa gitlab-ce
gitlab-ce-12.3.5-ce.0.el7.x86_64
#查看服务状态
~]# gitlab-ctl status

~]# gitlab-rake gitlab:check SANITIZE=true --trace
~]# gitlab-rake gitlab:check
~]# gitlab-rake gitlab:check SANITIZE=true

#停止服务
~]# gitlab-ctl reconfigure
~]# gitlab-ctl stop 
~]# gitlab-ctl status

#卸载旧包
~]# rpm -evh `rpm -qa gitlab-ce`
Preparing...                          ################################# [100%]
Cleaning up / removing...
   1:gitlab-ce-12.3.5-ce.0.el7        ################################# [100%]

#安装完成后，不管成功或者失败，一定要淡定，慢慢解决问题，不要着急。
~]# rpm -ivh gitlab-ce-12.10.14-ce.0.el7.x86_64.rpm 
warning: gitlab-ce-12.10.14-ce.0.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID f27eab47: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:gitlab-ce-12.10.14-ce.0.el7      ##########                        ( 31%)

确认配置文件内容存在,如果不存在替换之前的备份文件：
]# cd /etc/gitlab/
]# egrep -v "^$|^#" gitlab.rb


#安装后可能会有部分服务启动失败，执行如下命令：
]# gitlab-ctl restart #全部启动启动成功

#刷新配置 
]# gitlab-ctl reconfigure


#确认当前版本及服务状态：
[root@VM-16-13-centos gitlab]# gitlab-ctl status 
run: alertmanager: (pid 20600) 623s; run: log: (pid 1564) 234492s
run: gitaly: (pid 20610) 622s; run: log: (pid 1581) 234492s
run: gitlab-exporter: (pid 21734) 517s; run: log: (pid 1563) 234492s
run: gitlab-workhorse: (pid 21715) 518s; run: log: (pid 1562) 234492s
run: grafana: (pid 21774) 515s; run: log: (pid 1574) 234492s
run: logrotate: (pid 20667) 621s; run: log: (pid 1560) 234492s
run: nginx: (pid 21727) 518s; run: log: (pid 1575) 234492s
run: node-exporter: (pid 20691) 620s; run: log: (pid 1571) 234492s
run: postgres-exporter: (pid 21756) 515s; run: log: (pid 1559) 234492s
run: postgresql: (pid 20712) 619s; run: log: (pid 1587) 234492s
run: prometheus: (pid 20714) 618s; run: log: (pid 1561) 234492s
run: redis: (pid 20743) 618s; run: log: (pid 1598) 234492s
run: redis-exporter: (pid 20748) 618s; run: log: (pid 1565) 234492s
run: sidekiq: (pid 21701) 520s; run: log: (pid 1580) 234492s
run: unicorn: (pid 22994) 354s; run: log: (pid 1586) 234492s

版本验证：
]# rpm -qa gitlab-ce
gitlab-ce-12.10.14-ce.0.el7.x86_64
]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
12.10.14
```

> 登录页面访问验证版本
>

![image-20211117134526520](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031201.png)

## 3.2  PostgreSQL升级到11

官方说明：

![image-20211117135601100](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031202.png)

> 12.8版本有更新说明，但是官方推荐我们升级到12.10

![image-20211117135908991](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031203.png)

> sudo gitlab-ctl pg-upgrade -V 11
>

![image-20211117141233314](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031204.png)

```shell
#查看当前postgresql版本
[root@VM-16-9-centos gitlab]# rpm -qa gitlab-ce
gitlab-ce-12.10.14-ce.0.el7.x86_64
[root@VM-16-9-centos gitlab]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
12.10.14
[root@VM-16-9-centos gitlab]# cat /var/opt/gitlab/postgresql/data/PG_VERSION
10
[root@VM-16-9-centos gitlab]# /opt/gitlab/embedded/bin/pg_ctl --version
pg_ctl (PostgreSQL) 10.12
[root@VM-16-9-centos gitlab]# /opt/gitlab/embedded/bin/psql --version
psql (PostgreSQL) 10.12

#升级postgresql版本到11.x
gitlab]# gitlab-ctl pg-upgrade -V 11
Checking for an omnibus managed postgresql: OK
Checking if postgresql['version'] is set: OK
Checking if we already upgraded: NOT OK
Checking for a newer version of PostgreSQL to install
Upgrading PostgreSQL to 11.7
Checking if PostgreSQL bin files are symlinked to the expected location: OK
Waiting 30 seconds to ensure tasks complete before PostgreSQL upgrade.
See https://docs.gitlab.com/omnibus/settings/database.html#upgrade-packaged-postgresql-server for details
If you do not want to upgrade the PostgreSQL server at this time, enter Ctrl-C and see the documentation for details

Please hit Ctrl-C now if you want to cancel the operation.
.................
Running handlers complete
Chef Client finished, 4/743 resources updated in 06 seconds
Running reconfigure: OK
Waiting for Database to be running.
Database upgrade is complete, running vacuumdb analyze
Toggling deploy page:rm -f /opt/gitlab/embedded/service/gitlab-rails/public/index.html
Toggling deploy page: OK
Toggling services:ok: run: alertmanager: (pid 21957) 0s
ok: run: gitaly: (pid 21969) 1s
ok: run: gitlab-exporter: (pid 21989) 0s
ok: run: grafana: (pid 21999) 1s
ok: run: logrotate: (pid 22019) 0s
ok: run: node-exporter: (pid 22026) 0s
ok: run: postgres-exporter: (pid 22032) 0s
ok: run: prometheus: (pid 22039) 0s
ok: run: redis-exporter: (pid 22127) 1s
ok: run: sidekiq: (pid 22133) 0s
Toggling services: OK
==== Upgrade has completed ====
Please verify everything is working and run the following if so
sudo rm -rf /var/opt/gitlab/postgresql/data.10
sudo rm -f /var/opt/gitlab/postgresql-version.old

#调用sudo gitlab-ctl reconfigure以进行所需的配置更改并启动新的数据库服务器
]# gitlab-ctl reconfigure
]# gitlab-ctl status


#运行ANALYZE以生成数据库统计信息
]# sudo gitlab-psql -c "SELECT relname, last_analyze, last_autoanalyze FROM pg_stat_user_tables WHERE last_analyze IS NULL AND last_autoanalyze IS NULL;"
]# sudo gitlab-psql -c 'SET statement_timeout = 0; ANALYZE VERBOSE;'

#删除部署页面
]# rm -f /opt/gitlab/embedded/service/gitlab-rails/public/index.html

#清理旧的数据库文件
[root@VM-16-9-centos gitlab]# rm -rf /var/opt/gitlab/postgresql/data.10
[root@VM-16-9-centos gitlab]# rm -f /var/opt/gitlab/postgresql-version.old

#选择退出自动 PostgreSQL升级，需要运行如下命令
]# touch /etc/gitlab/disable-postgresql-upgrade

#升级完成后版本及服务状态确认：
[root@VM-16-9-centos gitlab]# cat /var/opt/gitlab/postgresql/data/PG_VERSION
11
[root@VM-16-9-centos gitlab]# /opt/gitlab/embedded/bin/pg_ctl --version
pg_ctl (PostgreSQL) 11.7
[root@VM-16-9-centos gitlab]# /opt/gitlab/embedded/bin/psql --version
psql (PostgreSQL) 11.7
[root@VM-16-9-centos gitlab]# gitlab-ctl status
run: alertmanager: (pid 21957) 96s; run: log: (pid 624) 77903s
run: gitaly: (pid 21969) 96s; run: log: (pid 32090) 78015s
run: gitlab-exporter: (pid 21989) 95s; run: log: (pid 477) 77917s
run: gitlab-workhorse: (pid 14371) 1683s; run: log: (pid 32572) 77962s
run: grafana: (pid 21999) 95s; run: log: (pid 735) 77892s
run: logrotate: (pid 22019) 94s; run: log: (pid 375) 77929s
run: nginx: (pid 14380) 1683s; run: log: (pid 32619) 77955s
run: node-exporter: (pid 22026) 94s; run: log: (pid 437) 77923s
run: postgres-exporter: (pid 22032) 93s; run: log: (pid 684) 77897s
run: postgresql: (pid 21474) 110s; run: log: (pid 32253) 78011s
run: prometheus: (pid 22039) 93s; run: log: (pid 572) 77909s
run: redis: (pid 13259) 1808s; run: log: (pid 32039) 78018s
run: redis-exporter: (pid 22127) 93s; run: log: (pid 526) 77913s
run: sidekiq: (pid 22133) 92s; run: log: (pid 32529) 77967s
run: unicorn: (pid 15426) 1541s; run: log: (pid 32495) 77973s

#保险起见访问页面看看是否正常。
```



# 四、12.10.14 -> 13.0.14 升级：

```shell
升级前检查：
]# rpm -qa gitlab-ce
gitlab-ce-12.10.14-ce.0.el7.x86_64
## 查看服务状态 
gitlab-ctl reconfigure
gitlab-ctl status 

gitlab-rake gitlab:check SANITIZE=true --trace
gitlab-rake gitlab:check
gitlab-rake gitlab:check SANITIZE=true

#停止服务及确认服务状态
gitlab-ctl stop
gitlab-ctl status

#卸载旧包
]# rpm -qa gitlab-ce
gitlab-ce-12.10.14-ce.0.el7.x86_64

]# rpm -e `rpm -qa gitlab-ce`
Preparing...                          ################################# [100%]
Cleaning up / removing...
   1:gitlab-ce-12.10.14-ce.0.el7      ################################# [100%]


#安装过程时间较长，耐心等待：
]# rpm -ivh gitlab-ce-13.0.14-ce.0.el7.x86_64.rpm
warning: gitlab-ce-13.0.14-ce.0.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID f27eab47: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:gitlab-ce-13.0.14-ce.0.el7       ##############                    ( 43%)



#部分服务启动失败，等待5分钟后执行如下命令：
]# gitlab-ctl restart 

#加载配置：
]# gitlab-ctl reconfigure

#查看当前服务状态：
]# gitlab-ctl status
run: alertmanager: (pid 29168) 122s; run: log: (pid 624) 79418s
run: gitaly: (pid 29179) 121s; run: log: (pid 32090) 79530s
run: gitlab-exporter: (pid 29187) 121s; run: log: (pid 477) 79432s
run: gitlab-workhorse: (pid 29199) 120s; run: log: (pid 32572) 79477s
run: grafana: (pid 29224) 120s; run: log: (pid 735) 79407s
run: logrotate: (pid 29234) 119s; run: log: (pid 375) 79444s
run: nginx: (pid 29243) 119s; run: log: (pid 32619) 79470s
run: node-exporter: (pid 29262) 118s; run: log: (pid 437) 79438s
run: postgres-exporter: (pid 29267) 118s; run: log: (pid 684) 79412s
run: postgresql: (pid 29352) 118s; run: log: (pid 32253) 79526s
run: prometheus: (pid 29361) 117s; run: log: (pid 572) 79424s
run: puma: (pid 29373) 116s; run: log: (pid 28794) 200s
run: redis: (pid 29379) 116s; run: log: (pid 32039) 79533s
run: redis-exporter: (pid 29384) 116s; run: log: (pid 526) 79428s
run: sidekiq: (pid 29390) 115s; run: log: (pid 32529) 79482s

#验证当前版本
]# rpm -qa gitlab-ce
gitlab-ce-13.0.14-ce.0.el7.x86_64
]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
13.0.14
```

> 页面访问验证：
>

![image-20211117143914367](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031205.png)

# 五、13.0.14 -> 13.1.11 升级

```shell
升级前检查：
]# rpm -qa gitlab-ce
gitlab-ce-13.0.14-ce.0.el7.x86_64
## 查看服务状态 
gitlab-ctl reconfigure
gitlab-ctl status 

gitlab-rake gitlab:check SANITIZE=true --trace
gitlab-rake gitlab:check
gitlab-rake gitlab:check SANITIZE=true


#停止服务及确认服务状态
gitlab-ctl stop
gitlab-ctl status

#卸载旧包
]# rpm -qa gitlab-ce
gitlab-ce-13.0.14-ce.0.el7.x86_64

]# rpm -evh `rpm -qa gitlab-ce`
Preparing...                          ################################# [100%]
Cleaning up / removing...
   1:gitlab-ce-13.0.14-ce.0.el7       ################################# [100%]

#安装过程时间较长，耐心等待：
packages]# rpm -ivh gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm
warning: gitlab-ce-13.1.11-ce.0.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID f27eab47: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:gitlab-ce-13.1.11-ce.0.el7       ###############                   ( 46%)
中间内容忽略
...
Running handlers:
Running handlers complete
Chef Client finished, 100/1131 resources updated in 53 seconds
gitlab Reconfigured!
Restarting previously running GitLab services

     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/
  

Upgrade complete! If your GitLab server is misbehaving try running
  sudo gitlab-ctl restart
before anything else.
If you need to roll back to the previous version you can use the database
backup made during the upgrade (scroll up for the filename).

#重新启动服务
]# gitlab-ctl restart

#查看当前服务状态
]# gitlab-ctl status

#加载配置：
]# gitlab-ctl reconfigure

#加载配置后状态查看
]# gitlab-ctl status

#版本确认：
]# rpm -qa gitlab-ce
gitlab-ce-13.1.11-ce.0.el7.x86_64
]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
13.1.11
```

> 访问页面验证：
>

![image-20211117145305743](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031206.png)

# 六、13.1.11 -> 13.8.8 升级：

```shell
升级前检查：
]# rpm -qa gitlab-ce
gitlab-ce-13.1.11-ce.0.el7.x86_64
## 查看服务状态 
gitlab-ctl status 

gitlab-rake gitlab:check SANITIZE=true --trace
gitlab-rake gitlab:check
gitlab-rake gitlab:check SANITIZE=true


#停止服务及确认服务状态
gitlab-ctl stop
gitlab-ctl status


]# rpm -qa gitlab-ce
gitlab-ce-13.1.11-ce.0.el7.x86_64

#卸载
]# rpm -evh `rpm -qa gitlab-ce`
Preparing...                          ################################# [100%]
Cleaning up / removing...
   1:gitlab-ce-13.1.11-ce.0.el7       ################################# [100%]
   
#安装过程时间较长，耐心等待：
]# rpm -ivh gitlab-ce-13.8.8-ce.0.el7.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:gitlab-ce-13.8.8-ce.0.el7        ################################# [100%]
Checking PostgreSQL executables:
...
Running handlers:
Running handlers complete
Chef Infra Client finished, 68/801 resources updated in 01 minutes 05 seconds
gitlab Reconfigured!
Restarting previously running GitLab services

     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/
  

Upgrade complete! If your GitLab server is misbehaving try running
  sudo gitlab-ctl restart
before anything else.
If you need to roll back to the previous version you can use the database
backup made during the upgrade (scroll up for the filename).

=== INFO ===
You are currently running PostgreSQL 11.
Note that PostgreSQL 12 will become the minimum required PostgreSQL version in GitLab 14.0 (May 2021).
PostgreSQL 11 will be removed in GitLab 14.0. Please consider upgrading your PostgreSQL version soon.
To upgrade, please see: https://docs.gitlab.com/omnibus/settings/database.html#upgrade-packaged-postgresql-server
=== INFO ===
Help us improve the upgrade experience, let us know how we did with a 1 minute survey:
https://gitlab.fra1.qualtrics.com/jfe/form/SV_0Hwcx9ncPfygMfj?installation=omnibus&release=13-8
#当前版本做的越来越人性化了，升级后有提示说明


#重新启动服务
]# gitlab-ctl restart

#查看当前服务状态
]# gitlab-ctl status

#加载配置：
gitlab-ctl reconfigure

#加载配置后状态查看
gitlab-ctl status

#版本确认：
]# rpm -qa gitlab-ce
gitlab-ce-13.8.8-ce.0.el7.x86_64
]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
13.8.8
```

访问页面验证版本：

![image-20211117150619662](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031207.png)

## 6.1 PostgreSQL升级到12

> #上面部署完成后有链接说明：

![image-20211117151106187](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031208.png)

```shell
#升级前版本检查
[root@VM-16-9-centos gitlab]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
13.8.8[root@VM-16-9-centos gitlab]# cat /var/opt/gitlab/postgresql/data/PG_VERSION
11
[root@VM-16-9-centos gitlab]# /opt/gitlab/embedded/bin/pg_ctl --version
pg_ctl (PostgreSQL) 11.11
[root@VM-16-9-centos gitlab]# /opt/gitlab/embedded/bin/psql --version
psql (PostgreSQL) 11.11

#开始升级
]# gitlab-ctl pg-upgrade
Running handlers:
Running handlers complete
Chef Infra Client finished, 3/790 resources updated in 09 seconds
Running reconfigure: OK
Waiting for Database to be running.
Database upgrade is complete, running vacuumdb analyze
Toggling deploy page:rm -f /opt/gitlab/embedded/service/gitlab-rails/public/index.html
Toggling deploy page: OK
Toggling services:ok: run: alertmanager: (pid 22264) 1s
ok: run: gitaly: (pid 22272) 0s
ok: run: gitlab-exporter: (pid 22287) 0s
ok: run: grafana: (pid 22302) 1s
ok: run: logrotate: (pid 22311) 0s
ok: run: node-exporter: (pid 22318) 1s
ok: run: postgres-exporter: (pid 22324) 0s
ok: run: prometheus: (pid 22334) 1s
ok: run: redis-exporter: (pid 22421) 0s
ok: run: sidekiq: (pid 22427) 0s
Toggling services: OK
==== Upgrade has completed ====
Please verify everything is working and run the following if so
sudo rm -rf /var/opt/gitlab/postgresql/data.11
sudo rm -f /var/opt/gitlab/postgresql-version.old

#调用sudo gitlab-ctl reconfigure以进行所需的配置更改并启动新的数据库服务器
]# gitlab-ctl reconfigure
]# gitlab-ctl status

#运行ANALYZE以生成数据库统计信息
]# sudo gitlab-psql -c "SELECT relname, last_analyze, last_autoanalyze FROM pg_stat_user_tables WHERE last_analyze IS NULL AND last_autoanalyze IS NULL;"
]# sudo gitlab-psql -c 'SET statement_timeout = 0; ANALYZE VERBOSE;'

#删除部署页面
]# rm -f /opt/gitlab/embedded/service/gitlab-rails/public/index.html

#清理旧的数据库文件
sudo rm -rf /var/opt/gitlab/postgresql/data.11
sudo rm -f /var/opt/gitlab/postgresql-version.old

#选择退出自动 PostgreSQL升级，需要运行如下命令
]# touch /etc/gitlab/disable-postgresql-upgrade

#升级完成后版本及服务状态确认：
]# cat /var/opt/gitlab/postgresql/data/PG_VERSION
12
]# /opt/gitlab/embedded/bin/pg_ctl --version
pg_ctl (PostgreSQL) 12.6
]# /opt/gitlab/embedded/bin/psql --version
psql (PostgreSQL) 12.6

#确认当前服务状态正常
]# gitlab-ctl status


#保险起见访问页面看看是否正常。
```

# 七、13.8.8 -> 13.12.15 升级

```shell
升级前检查：
]# rpm -qa gitlab-ce
gitlab-ce-13.8.8-ce.0.el7.x86_64
## 查看服务状态 
gitlab-ctl status 

gitlab-rake gitlab:check SANITIZE=true --trace
gitlab-rake gitlab:check
gitlab-rake gitlab:check SANITIZE=true

#停止服务及确认服务状态
gitlab-ctl stop
gitlab-ctl status

#卸载
]# rpm -qa gitlab-ce
gitlab-ce-13.8.8-ce.0.el7.x86_64
]#  rpm -evh `rpm -qa gitlab-ce`
Preparing...                          ################################# [100%]
Cleaning up / removing...
   1:gitlab-ce-13.8.8-ce.0.el7        ################################# [100%]

#安装过程时间较长，耐心等待：
]# rpm -ivh gitlab-ce-13.12.15-ce.0.el7.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:gitlab-ce-13.12.15-ce.0.el7      ################################# [100%]
Checking PostgreSQL executables:Starting Chef Infra Client, version 15.14.0
resolving cookbooks for run list: ["gitlab::config", "postgresql::bin"]
Synchronizing Cookbooks:
  - gitlab (0.0.1)
  - postgresql (0.1.0)
...
中间内容忽略
Recipe: gitlab::gitlab-workhorse
  * runit_service[gitlab-workhorse] action restart (up to date)
Recipe: monitoring::gitlab-exporter
  * runit_service[gitlab-exporter] action restart (up to date)
Recipe: monitoring::prometheus
  * execute[reload prometheus] action run (skipped due to only_if)
Recipe: monitoring::grafana
  * runit_service[grafana] action restart (up to date)

Running handlers:
Running handlers complete
Chef Infra Client finished, 65/810 resources updated in 01 minutes 01 seconds
gitlab Reconfigured!
Restarting previously running GitLab services

     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/
  

Upgrade complete! If your GitLab server is misbehaving try running
  sudo gitlab-ctl restart
before anything else.
If you need to roll back to the previous version you can use the database
backup made during the upgrade (scroll up for the filename).

#重新启动服务
]# gitlab-ctl restart

#查看当前服务状态
]# gitlab-ctl status

#加载配置：
]# gitlab-ctl reconfigure

#加载配置后状态查看
]# gitlab-ctl status

#版本确认：
]# rpm -qa gitlab-ce
gitlab-ce-13.12.15-ce.0.el7.x86_64
]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
13.12.15
```

> 页面访问验证版本：
>

![image-20211117153703472](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031209.png)

# 八、13.12.15 -> 14.0.12 升级：

```shell
升级前检查：
]# rpm -qa gitlab-ce
gitlab-ce-13.12.15-ce.0.el7.x86_64
## 查看服务状态 
gitlab-ctl status 

gitlab-rake gitlab:check SANITIZE=true --trace
gitlab-rake gitlab:check
gitlab-rake gitlab:check SANITIZE=true


#停止服务及确认服务状态
gitlab-ctl stop
gitlab-ctl status

#卸载
]# rpm -qa gitlab-ce
gitlab-ce-13.12.15-ce.0.el7.x86_64

]# rpm -evh `rpm -qa gitlab-ce`
Preparing...                          ################################# [100%]
Cleaning up / removing...
   1:gitlab-ce-13.12.15-ce.0.el7      ################################# [100%]

#安装过程时间较长，耐心等待：
]# rpm -ihv gitlab-ce-14.0.12-ce.0.el7.x86_64.rpm 
warning: gitlab-ce-14.0.12-ce.0.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID f27eab47: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:gitlab-ce-14.0.12-ce.0.el7       ################################# [100%]
Checking PostgreSQL executables:Starting Chef Infra Client, version 15.14.0
resolving cookbooks for run list: ["gitlab::config", "postgresql::bin"]
Synchronizing Cookbooks:
  - gitlab (0.0.1)
....
Recipe: monitoring::prometheus
  * execute[reload prometheus] action run (skipped due to only_if)

Running handlers:
Running handlers complete
Chef Infra Client finished, 25/769 resources updated in 52 seconds
gitlab Reconfigured!
Restarting previously running GitLab services

     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/
  

Upgrade complete! If your GitLab server is misbehaving try running
  sudo gitlab-ctl restart
before anything else.
If you need to roll back to the previous version you can use the database
backup made during the upgrade (scroll up for the filename).

#重新启动服务
]# gitlab-ctl restart

#查看当前服务状态
]# gitlab-ctl status

#加载配置：
]# gitlab-ctl reconfigure

#加载配置后状态查看
]# gitlab-ctl status

#版本确认：
]# rpm -qa gitlab-ce
gitlab-ce-14.0.12-ce.0.el7.x86_64
]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
14.0.12
```

> 访问页面版本验证：
>

![image-20211117155910120](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031210.png)

## 8.1运行所有批处理后台迁移

> #官方对14.0版本有说明：

![image-20211117160122699](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031211.png)

```shell
对于有停机时间的部署
#要运行所有批处理后台迁移，可能需要大量时间，具体取决于 GitLab 安装的大小。

1.检查数据库中批处理后台迁移的状态，并使用适当的参数手动运行它们，直到状态查询不返回任何行。

2.当所有它们的状态都标记为完成时，重新运行安装的迁移。

3.从 GitLab 升级完成数据库迁移：
sudo gitlab-rake db:migrate

4.运行重新配置：
sudo gitlab-ctl reconfigure

5.完成安装的升级。
gitlab-ctl status

#验证页面访问是否有异常
```

# 九、**14.0.12 -> 14.4.2 升级：**

```shell
升级前检查：
]# rpm -qa gitlab-ce
gitlab-ce-14.0.12-ce.0.el7.x86_64

## 查看服务状态 
gitlab-ctl status 

gitlab-rake gitlab:check SANITIZE=true --trace
gitlab-rake gitlab:check
gitlab-rake gitlab:check SANITIZE=true

#停止服务及确认服务状态
gitlab-ctl stop
gitlab-ctl status

#卸载
]# rpm -qa gitlab-ce
gitlab-ce-14.0.12-ce.0.el7.x86_64

]# rpm -evh `rpm -qa gitlab-ce`
Preparing...                          ################################# [100%]
Cleaning up / removing...
   1:gitlab-ce-14.0.12-ce.0.el7       ################################# [100%]
   
#安装过程时间较长，耐心等待：
]# rpm -ivh gitlab-ce-14.4.2-ce.0.el7.x86_64.rpm 
warning: gitlab-ce-14.4.2-ce.0.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID f27eab47: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:gitlab-ce-14.4.2-ce.0.el7        ################################# [100%]
Checking PostgreSQL executables:Starting Chef Infra Client, version 15.17.4
resolving cookbooks for run list: ["gitlab::config", "postgresql::bin"]
Synchronizing Cookbooks:
  - gitlab (0.0.1)
...
Recipe: gitlab::gitlab-rails
  * execute[clear the gitlab-rails cache] action run
    - execute /opt/gitlab/bin/gitlab-rake cache:clear
Recipe: gitaly::enable
  * runit_service[gitaly] action restart (up to date)
  * runit_service[gitaly] action hup
    - send hup to runit_service[gitaly]

Running handlers:
There was an error running gitlab-ctl reconfigure:

rails_migration[gitlab-rails] (gitlab::database_migrations line 51) had an error: Mixlib::ShellOut::ShellCommandFailed: bash[migrate gitlab-rails database] (/opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/resources/rails_migration.rb line 16) had an error: Mixlib::ShellOut::ShellCommandFailed: Expected process to exit with [0], but received '1'
---- Begin output of "bash"  "/tmp/chef-script20211117-8745-1ngw6y0" ----
STDOUT: == 20210317210338 AddValidRunnerRegistrars: migrating =========================
-- add_column(:application_settings, :valid_runner_registrars, :string, {:array=>true, :default=>["project", "group"]})
   -> 0.0177s
== 20210317210338 AddValidRunnerRegistrars: migrated (0.0178s) ================

== 20210601132134 RemovePartialIndexForHashedStorageMigration: migrating ======
-- transaction_open?()
   -> 0.0000s
-- index_exists?(:projects, :id, {:name=>"index_on_id_partial_with_legacy_storage", :algorithm=>:concurrently})
   -> 0.0212s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- remove_index(:projects, {:name=>"index_on_id_partial_with_legacy_storage", :algorithm=>:concurrently, :column=>:id})
   -> 0.0257s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210601132134 RemovePartialIndexForHashedStorageMigration: migrated (0.0484s) 

== 20210602155056 AddMergeRequestDiffCommitUsers: migrating ===================
-- create_table(:merge_request_diff_commit_users, {:id=>:bigint})
-- quote_column_name(:name)
   -> 0.0000s
-- quote_column_name(:email)
   -> 0.0000s
   -> 0.0538s
-- quote_table_name("check_147358fc42")
   -> 0.0000s
-- quote_table_name("check_f5fa206cf7")
   -> 0.0000s
-- quote_table_name(:merge_request_diff_commit_users)
   -> 0.0000s
-- execute("ALTER TABLE \"merge_request_diff_commit_users\"\nADD CONSTRAINT \"check_147358fc42\" CHECK (char_length(\"name\") <= 512),\nADD CONSTRAINT \"check_f5fa206cf7\" CHECK (char_length(\"email\") <= 512)\n")
   -> 0.0024s
-- transaction_open?()
   -> 0.0000s
-- current_schema()
   -> 0.0002s
-- execute("ALTER TABLE merge_request_diff_commit_users\nADD CONSTRAINT merge_request_diff_commit_users_name_or_email_existence\nCHECK ( (COALESCE(name, '') != '') OR (COALESCE(email, '') != '') )\nNOT VALID;\n")
   -> 0.0005s
-- current_schema()
   -> 0.0001s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- execute("ALTER TABLE merge_request_diff_commit_users VALIDATE CONSTRAINT merge_request_diff_commit_users_name_or_email_existence;")
   -> 0.0011s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210602155056 AddMergeRequestDiffCommitUsers: migrated (0.0785s) ==========

== 20210602155110 AddMergeRequestDiffCommitUserColumns: migrating =============
-- add_column(:merge_request_diff_commits, :commit_author_id, :bigint)
   -> 0.0012s
-- add_column(:merge_request_diff_commits, :committer_id, :bigint)
   -> 0.0003s
== 20210602155110 AddMergeRequestDiffCommitUserColumns: migrated (0.0015s) ====

== 20210602164044 ScheduleLatestPipelineIdPopulation: migrating ===============
== 20210602164044 ScheduleLatestPipelineIdPopulation: migrated (0.0000s) ======

== 20210604032738 CreateDastSiteProfilesBuilds: migrating =====================
-- create_table(:dast_site_profiles_builds, {:primary_key=>[:dast_site_profile_id, :ci_build_id], :comment=>"{\"owner\":\"group::dynamic analysis\",\"description\":\"Join table between DAST Site Profiles and CI Builds\"}"})
   -> 0.0124s
== 20210604032738 CreateDastSiteProfilesBuilds: migrated (0.0125s) ============

== 20210604034158 AddCiBuildIdFkToDastSiteProfilesBuilds: migrating ===========
-- transaction_open?()
   -> 0.0000s
-- foreign_keys(:dast_site_profiles_builds)
   -> 0.0042s
-- execute("ALTER TABLE dast_site_profiles_builds\nADD CONSTRAINT fk_a325505e99\nFOREIGN KEY (ci_build_id)\nREFERENCES ci_builds (id)\nON DELETE CASCADE\nNOT VALID;\n")
   -> 0.0194s
-- execute("SET statement_timeout TO 0")
   -> 0.0002s
-- execute("ALTER TABLE dast_site_profiles_builds VALIDATE CONSTRAINT fk_a325505e99;")
   -> 0.0334s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210604034158 AddCiBuildIdFkToDastSiteProfilesBuilds: migrated (0.0604s) ==

== 20210604034354 AddDastSiteProfileIdFkToDastSiteProfilesBuilds: migrating ===
-- transaction_open?()
   -> 0.0000s
-- foreign_keys(:dast_site_profiles_builds)
   -> 0.0040s
-- execute("ALTER TABLE dast_site_profiles_builds\nADD CONSTRAINT fk_94e80df60e\nFOREIGN KEY (dast_site_profile_id)\nREFERENCES dast_site_profiles (id)\nON DELETE CASCADE\nNOT VALID;\n")
   -> 0.0026s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- execute("ALTER TABLE dast_site_profiles_builds VALIDATE CONSTRAINT fk_94e80df60e;")
   -> 0.0033s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210604034354 AddDastSiteProfileIdFkToDastSiteProfilesBuilds: migrated (0.0128s) 

== 20210604051330 CreateDastScannerProfilesBuilds: migrating ==================
-- create_table(:dast_scanner_profiles_builds, {:primary_key=>[:dast_scanner_profile_id, :ci_build_id], :comment=>"{\"owner\":\"group::dynamic analysis\",\"description\":\"Join table between DAST Scanner Profiles and CI Builds\"}"})
   -> 0.0076s
== 20210604051330 CreateDastScannerProfilesBuilds: migrated (0.0077s) =========

== 20210604051742 AddCiBuildIdFkToDastScannerProfilesBuilds: migrating ========
-- transaction_open?()
   -> 0.0000s
-- foreign_keys(:dast_scanner_profiles_builds)
   -> 0.0040s
-- execute("ALTER TABLE dast_scanner_profiles_builds\nADD CONSTRAINT fk_e4c49200f8\nFOREIGN KEY (ci_build_id)\nREFERENCES ci_builds (id)\nON DELETE CASCADE\nNOT VALID;\n")
   -> 0.0006s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- execute("ALTER TABLE dast_scanner_profiles_builds VALIDATE CONSTRAINT fk_e4c49200f8;")
   -> 0.0014s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210604051742 AddCiBuildIdFkToDastScannerProfilesBuilds: migrated (0.0087s) 

== 20210604051917 AddDastScannerProfileIdFkToDastScannerProfilesBuilds: migrating 
-- transaction_open?()
   -> 0.0000s
-- foreign_keys(:dast_scanner_profiles_builds)
   -> 0.0037s
-- execute("ALTER TABLE dast_scanner_profiles_builds\nADD CONSTRAINT fk_5d46286ad3\nFOREIGN KEY (dast_scanner_profile_id)\nREFERENCES dast_scanner_profiles (id)\nON DELETE CASCADE\nNOT VALID;\n")
   -> 0.0029s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- execute("ALTER TABLE dast_scanner_profiles_builds VALIDATE CONSTRAINT fk_5d46286ad3;")
   -> 0.0119s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210604051917 AddDastScannerProfileIdFkToDastScannerProfilesBuilds: migrated (0.0216s) 

== 20210604133651 ScheduleMergeRequestDiffUsersBackgroundMigration: migrating =
== 20210604133651 ScheduleMergeRequestDiffUsersBackgroundMigration: migrated (0.0000s) 

== 20210609202501 ScheduleBackfillDraftStatusOnMergeRequests: migrating =======
== 20210609202501 ScheduleBackfillDraftStatusOnMergeRequests: migrated (0.0000s) 

== 20210610042700 RemoveClustersApplicationsFluentdTable: migrating ===========
-- drop_table(:clusters_applications_fluentd)
   -> 0.0188s
== 20210610042700 RemoveClustersApplicationsFluentdTable: migrated (0.0188s) ==

== 20210610153556 DeleteLegacyOperationsFeatureFlags: migrating ===============
-- execute("DELETE FROM operations_feature_flags WHERE version = 1")
   -> 0.0034s
== 20210610153556 DeleteLegacyOperationsFeatureFlags: migrated (0.0034s) ======

== 20210611082822 AddPagesFileEntriesToPlanLimits: migrating ==================
-- add_column(:plan_limits, :pages_file_entries, :integer, {:default=>200000, :null=>false})
   -> 0.0026s
== 20210611082822 AddPagesFileEntriesToPlanLimits: migrated (0.0026s) =========

== 20210611101034 AddDevopsAdoptionSastDast: migrating ========================
-- add_column(:analytics_devops_adoption_snapshots, :sast_enabled_count, :integer)
   -> 0.0012s
-- add_column(:analytics_devops_adoption_snapshots, :dast_enabled_count, :integer)
   -> 0.0003s
== 20210611101034 AddDevopsAdoptionSastDast: migrated (0.0015s) ===============

== 20210614124111 AddDevopsAdoptionSastDastIndexes: migrating =================
-- transaction_open?()
   -> 0.0000s
-- index_exists?(:ci_job_artifacts, [:project_id, :created_at], {:where=>"file_type = 5", :name=>"index_ci_job_artifacts_sast_for_devops_adoption", :algorithm=>:concurrently})
   -> 0.0030s
-- execute("SET statement_timeout TO 0")
   -> 0.0002s
-- add_index(:ci_job_artifacts, [:project_id, :created_at], {:where=>"file_type = 5", :name=>"index_ci_job_artifacts_sast_for_devops_adoption", :algorithm=>:concurrently})
   -> 0.0069s
-- execute("RESET statement_timeout")
   -> 0.0002s
-- transaction_open?()
   -> 0.0000s
-- index_exists?(:ci_job_artifacts, [:project_id, :created_at], {:where=>"file_type = 8", :name=>"index_ci_job_artifacts_dast_for_devops_adoption", :algorithm=>:concurrently})
   -> 0.0261s
-- execute("SET statement_timeout TO 0")
   -> 0.0002s
-- add_index(:ci_job_artifacts, [:project_id, :created_at], {:where=>"file_type = 8", :name=>"index_ci_job_artifacts_dast_for_devops_adoption", :algorithm=>:concurrently})
   -> 0.0051s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210614124111 AddDevopsAdoptionSastDastIndexes: migrated (0.0435s) ========

== 20210614142311 AddRunningContainerScanningMaxSizeToPlanLimits: migrating ===
-- add_column(:plan_limits, :ci_max_artifact_size_running_container_scanning, :integer, {:null=>false, :default=>0})
   -> 0.0009s
== 20210614142311 AddRunningContainerScanningMaxSizeToPlanLimits: migrated (0.0009s) 

== 20210615064342 AddIssueIdToRequirement: migrating ==========================
-- add_column(:requirements, :issue_id, :bigint, {:null=>true})
   -> 0.0011s
== 20210615064342 AddIssueIdToRequirement: migrated (0.0022s) =================

== 20210615234935 FixBatchedMigrationsOldFormatJobArguments: migrating ========
== 20210615234935 FixBatchedMigrationsOldFormatJobArguments: migrated (0.0233s) 

== 20210616110748 AddIssueIndexToRequirement: migrating =======================
-- transaction_open?()
   -> 0.0000s
-- index_exists?(:requirements, :issue_id, {:name=>"index_requirements_on_issue_id", :unique=>true, :algorithm=>:concurrently})
   -> 0.0029s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- add_index(:requirements, :issue_id, {:name=>"index_requirements_on_issue_id", :unique=>true, :algorithm=>:concurrently})
   -> 0.0073s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210616110748 AddIssueIndexToRequirement: migrated (0.0112s) ==============

== 20210616111311 AddIssueRequirementForeignKey: migrating ====================
-- transaction_open?()
   -> 0.0000s
-- foreign_keys(:requirements)
   -> 0.0037s
-- execute("ALTER TABLE requirements\nADD CONSTRAINT fk_85044baef0\nFOREIGN KEY (issue_id)\nREFERENCES issues (id)\nON DELETE CASCADE\nNOT VALID;\n")
   -> 0.0038s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- execute("ALTER TABLE requirements VALIDATE CONSTRAINT fk_85044baef0;")
   -> 0.0309s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210616111311 AddIssueRequirementForeignKey: migrated (0.0413s) ===========

== 20210616134905 AddTimestampToSchemaMigration: migrating ====================
-- add_column(:schema_migrations, :finished_at, :timestamptz)
   -> 0.0019s
-- change_column_default(:schema_migrations, :finished_at, #<Proc:0x00007f088f879bc0 /opt/gitlab/embedded/service/gitlab-rails/db/migrate/20210616134905_add_timestamp_to_schema_migration.rb:9 (lambda)>)
   -> 0.0027s
== 20210616134905 AddTimestampToSchemaMigration: migrated (0.0046s) ===========

== 20210616145254 AddPartialIndexForCiBuildsToken: migrating ==================
-- transaction_open?()
   -> 0.0000s
-- index_exists?(:ci_builds, :token, {:unique=>true, :where=>"token IS NOT NULL", :name=>"index_ci_builds_on_token_partial", :algorithm=>:concurrently})
   -> 0.0086s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- add_index(:ci_builds, :token, {:unique=>true, :where=>"token IS NOT NULL", :name=>"index_ci_builds_on_token_partial", :algorithm=>:concurrently})
   -> 0.0204s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210616145254 AddPartialIndexForCiBuildsToken: migrated (0.0304s) =========

== 20210616154808 RemoveCiBuildProtectedIndex: migrating ======================
-- transaction_open?()
   -> 0.0000s
-- index_exists?(:ci_builds, :protected, {:name=>"index_ci_builds_on_protected", :algorithm=>:concurrently})
   -> 0.0094s
-- execute("SET statement_timeout TO 0")
   -> 0.0002s
-- remove_index(:ci_builds, {:name=>"index_ci_builds_on_protected", :algorithm=>:concurrently, :column=>:protected})
   -> 0.0118s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210616154808 RemoveCiBuildProtectedIndex: migrated (0.0225s) =============

== 20210616185947 AddMailgunSettingsToApplicationSetting: migrating ===========
-- add_column(:application_settings, :encrypted_mailgun_signing_key, :binary)
   -> 0.0017s
-- add_column(:application_settings, :encrypted_mailgun_signing_key_iv, :binary)
   -> 0.0005s
-- add_column(:application_settings, :mailgun_events_enabled, :boolean, {:default=>false, :null=>false})
   -> 0.0009s
== 20210616185947 AddMailgunSettingsToApplicationSetting: migrated (0.0031s) ==

== 20210617022324 CreateIncidentManagementPendingAlertEscalations: migrating ==
-- execute("\nCREATE TABLE incident_management_pending_alert_escalations (\n  id bigserial NOT NULL,\n  rule_id bigint,\n  alert_id bigint NOT NULL,\n  schedule_id bigint NOT NULL,\n  process_at timestamp with time zone NOT NULL,\n  created_at timestamp with time zone NOT NULL,\n  updated_at timestamp with time zone NOT NULL,\n  status smallint NOT NULL,\n  PRIMARY KEY (id, process_at)\n) PARTITION BY RANGE (process_at);\n\nCREATE INDEX index_incident_management_pending_alert_escalations_on_alert_id\n  ON incident_management_pending_alert_escalations USING btree (alert_id);\n\nCREATE INDEX index_incident_management_pending_alert_escalations_on_rule_id\n  ON incident_management_pending_alert_escalations USING btree (rule_id);\n\nCREATE INDEX index_incident_management_pending_alert_escalations_on_schedule_id\n  ON incident_management_pending_alert_escalations USING btree (schedule_id);\n\nALTER TABLE incident_management_pending_alert_escalations ADD CONSTRAINT fk_rails_fcbfd9338b\n  FOREIGN KEY (schedule_id) REFERENCES incident_management_oncall_schedules(id) ON DELETE CASCADE;\n\nALTER TABLE incident_management_pending_alert_escalations ADD CONSTRAINT fk_rails_057c1e3d87\n  FOREIGN KEY (rule_id) REFERENCES incident_management_escalation_rules(id) ON DELETE SET NULL;\n\nALTER TABLE incident_management_pending_alert_escalations ADD CONSTRAINT fk_rails_8d8de95da9\n  FOREIGN KEY (alert_id) REFERENCES alert_management_alerts(id) ON DELETE CASCADE;\n")
   -> 0.0075s
== 20210617022324 CreateIncidentManagementPendingAlertEscalations: migrated (0.0086s) 

== 20210617161348 CascadeDeleteFreezePeriods: migrating =======================
-- transaction_open?()
   -> 0.0000s
-- foreign_keys(:ci_freeze_periods)
   -> 0.0039s
-- execute("ALTER TABLE ci_freeze_periods\nADD CONSTRAINT fk_2e02bbd1a6\nFOREIGN KEY (project_id)\nREFERENCES projects (id)\nON DELETE CASCADE\nNOT VALID;\n")
   -> 0.0028s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- execute("ALTER TABLE ci_freeze_periods VALIDATE CONSTRAINT fk_2e02bbd1a6;")
   -> 0.0444s
-- execute("RESET statement_timeout")
   -> 0.0002s
-- foreign_keys(:ci_freeze_periods)
   -> 0.0039s
-- remove_foreign_key(:ci_freeze_periods, :projects, {:column=>:project_id, :name=>"fk_rails_2e02bbd1a6"})
   -> 0.0064s
== 20210617161348 CascadeDeleteFreezePeriods: migrated (0.0645s) ==============

== 20210617180131 MigrateUsagePingSidekiqQueue: migrating =====================
== 20210617180131 MigrateUsagePingSidekiqQueue: migrated (0.0009s) ============

== 20210621043337 RenameServicesToIntegrations: migrating =====================
-- execute("LOCK services IN ACCESS EXCLUSIVE MODE")
   -> 0.0002s
-- execute("DROP TRIGGER IF EXISTS trigger_has_external_wiki_on_insert ON services")
   -> 0.0034s
-- execute("DROP TRIGGER IF EXISTS trigger_has_external_wiki_on_update ON services")
   -> 0.0002s
-- execute("DROP TRIGGER IF EXISTS trigger_has_external_wiki_on_delete ON services")
   -> 0.0002s
-- execute("DROP TRIGGER IF EXISTS trigger_has_external_issue_tracker_on_insert ON services")
   -> 0.0013s
-- execute("DROP TRIGGER IF EXISTS trigger_has_external_issue_tracker_on_update ON services")
   -> 0.0002s
-- execute("DROP TRIGGER IF EXISTS trigger_has_external_issue_tracker_on_delete ON services")
   -> 0.0002s
-- transaction()
-- rename_table(:services, :integrations)
   -> 0.0103s
-- execute("CREATE VIEW services AS SELECT * FROM integrations")
   -> 0.0051s
   -> 0.0154s
-- execute("CREATE TRIGGER trigger_has_external_wiki_on_insert\nAFTER INSERT ON integrations\nFOR EACH ROW\nWHEN (NEW.active = TRUE AND NEW.type = 'ExternalWikiService' AND NEW.project_id IS NOT NULL)\nEXECUTE FUNCTION set_has_external_wiki();\n")rake aborted!
StandardError: An error has occurred, all later migrations canceled:

Expected batched background migration for the given configuration to be marked as 'finished', but it is 'active':       {:job_class_name=>"CopyColumnUsingBackgroundMigrationJob", :table_name=>"events", :column_name=>"id", :job_arguments=>[["id"], ["id_convert_to_bigint"]]}

Finalize it manualy by running

        sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,events,id,'[["id"]\, ["id_convert_to_bigint"]]']

For more information, check the documentation

        https://docs.gitlab.com/ee/user/admin_area/monitoring/background_migrations.html#database-migrations-failing-because-of-batched-background-migration-not-finished
/opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/database/migration_helpers.rb:1109:in `ensure_batched_background_migration_is_finished'
/opt/gitlab/embedded/service/gitlab-rails/db/post_migrate/20210622045705_finalize_events_bigint_conversion.rb:11:in `up'
/opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/database/migrations/lock_retry_mixin.rb:31:in `ddl_transaction'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/db.rake:61:in `block (3 levels) in <top (required)>'
/opt/gitlab/embedded/bin/bundle:23:in `load'
/opt/gitlab/embedded/bin/bundle:23:in `<main>'

Caused by:
Expected batched background migration for the given configuration to be marked as 'finished', but it is 'active':       {:job_class_name=>"CopyColumnUsingBackgroundMigrationJob", :table_name=>"events", :column_name=>"id", :job_arguments=>[["id"], ["id_convert_to_bigint"]]}

Finalize it manualy by running

        sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,events,id,'[["id"]\, ["id_convert_to_bigint"]]']

For more information, check the documentation

        https://docs.gitlab.com/ee/user/admin_area/monitoring/background_migrations.html#database-migrations-failing-because-of-batched-background-migration-not-finished
/opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/database/migration_helpers.rb:1109:in `ensure_batched_background_migration_is_finished'
/opt/gitlab/embedded/service/gitlab-rails/db/post_migrate/20210622045705_finalize_events_bigint_conversion.rb:11:in `up'
/opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/database/migrations/lock_retry_mixin.rb:31:in `ddl_transaction'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/db.rake:61:in `block (3 levels) in <top (required)>'
/opt/gitlab/embedded/bin/bundle:23:in `load'
/opt/gitlab/embedded/bin/bundle:23:in `<main>'
Tasks: TOP => db:migrate
(See full trace by running task with --trace)

   -> 0.0013s
-- execute("CREATE TRIGGER trigger_has_external_wiki_on_update\nAFTER UPDATE ON integrations\nFOR EACH ROW\nWHEN (NEW.type = 'ExternalWikiService' AND OLD.active != NEW.active AND NEW.project_id IS NOT NULL)\nEXECUTE FUNCTION set_has_external_wiki();\n")
   -> 0.0004s
-- execute("CREATE TRIGGER trigger_has_external_wiki_on_delete\nAFTER DELETE ON integrations\nFOR EACH ROW\nWHEN (OLD.type = 'ExternalWikiService' AND OLD.project_id IS NOT NULL)\nEXECUTE FUNCTION set_has_external_wiki();\n")
   -> 0.0003s
-- execute("CREATE OR REPLACE FUNCTION set_has_external_issue_tracker()\nRETURNS TRIGGER AS\n$$\nBEGIN\nUPDATE projects SET has_external_issue_tracker = (\n  EXISTS\n  (\n    SELECT 1\n    FROM integrations\n    WHERE project_id = COALESCE(NEW.project_id, OLD.project_id)\n      AND active = TRUE\n      AND category = 'issue_tracker'\n  )\n)\nWHERE projects.id = COALESCE(NEW.project_id, OLD.project_id);\nRETURN NULL;\n\nEND\n$$ LANGUAGE PLPGSQL\n")
   -> 0.0045s
-- execute("CREATE TRIGGER trigger_has_external_issue_tracker_on_insert\nAFTER INSERT ON integrations\nFOR EACH ROW\nWHEN (NEW.category = 'issue_tracker' AND NEW.active = TRUE AND NEW.project_id IS NOT NULL)\nEXECUTE FUNCTION set_has_external_issue_tracker();\n")
   -> 0.0003s
-- execute("CREATE TRIGGER trigger_has_external_issue_tracker_on_update\nAFTER UPDATE ON integrations\nFOR EACH ROW\nWHEN (NEW.category = 'issue_tracker' AND OLD.active != NEW.active AND NEW.project_id IS NOT NULL)\nEXECUTE FUNCTION set_has_external_issue_tracker();\n")
   -> 0.0003s
-- execute("CREATE TRIGGER trigger_has_external_issue_tracker_on_delete\nAFTER DELETE ON integrations\nFOR EACH ROW\nWHEN (OLD.category = 'issue_tracker' AND OLD.active = TRUE AND OLD.project_id IS NOT NULL)\nEXECUTE FUNCTION set_has_external_issue_tracker();\n")
   -> 0.0003s
== 20210621043337 RenameServicesToIntegrations: migrated (0.0290s) ============

== 20210621044000 RenameServicesIndexesToIntegrations: migrating ==============
-- execute("ALTER INDEX IF EXISTS \"index_services_on_project_and_type_where_inherit_null\" RENAME TO \"index_integrations_on_project_and_type_where_inherit_null\"\n")
   -> 0.0004s
-- execute("ALTER INDEX IF EXISTS \"index_services_on_project_id_and_type_unique\" RENAME TO \"index_integrations_on_project_id_and_type_unique\"\n")
   -> 0.0002s
-- execute("ALTER INDEX IF EXISTS \"index_services_on_template\" RENAME TO \"index_integrations_on_template\"\n")
   -> 0.0001s
-- execute("ALTER INDEX IF EXISTS \"index_services_on_type\" RENAME TO \"index_integrations_on_type\"\n")
   -> 0.0001s
-- execute("ALTER INDEX IF EXISTS \"index_services_on_type_and_instance_partial\" RENAME TO \"index_integrations_on_type_and_instance_partial\"\n")
   -> 0.0002s
-- execute("ALTER INDEX IF EXISTS \"index_services_on_type_and_template_partial\" RENAME TO \"index_integrations_on_type_and_template_partial\"\n")
   -> 0.0002s
-- execute("ALTER INDEX IF EXISTS \"index_services_on_type_id_when_active_and_project_id_not_null\" RENAME TO \"index_integrations_on_type_id_when_active_and_project_id_not_null\"\n")
   -> 0.0010s
-- execute("ALTER INDEX IF EXISTS \"index_services_on_unique_group_id_and_type\" RENAME TO \"index_integrations_on_unique_group_id_and_type\"\n")
   -> 0.0009s
== 20210621044000 RenameServicesIndexesToIntegrations: migrated (0.0034s) =====

== 20210621084632 AddSummaryToTimelogs: migrating =============================
-- add_column(:timelogs, :summary, :text)
   -> 0.0054s
== 20210621084632 AddSummaryToTimelogs: migrated (0.0055s) ====================

== 20210621090030 AddTextLimitToTimelogsSummary: migrating ====================
-- transaction_open?()
   -> 0.0000s
-- current_schema()
   -> 0.0002s
-- execute("ALTER TABLE timelogs\nADD CONSTRAINT check_271d321699\nCHECK ( char_length(summary) <= 255 )\nNOT VALID;\n")
   -> 0.0004s
-- current_schema()
   -> 0.0001s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- execute("ALTER TABLE timelogs VALIDATE CONSTRAINT check_271d321699;")
   -> 0.0013s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210621090030 AddTextLimitToTimelogsSummary: migrated (0.0059s) ===========

== 20210621091830 AddDevopsAdoptionSnapshotDependencyScanning: migrating ======
-- add_column(:analytics_devops_adoption_snapshots, :dependency_scanning_enabled_count, :integer)
   -> 0.0005s
== 20210621091830 AddDevopsAdoptionSnapshotDependencyScanning: migrated (0.0005s) 

== 20210621111747 AddCiArtifactsDevopsAdoptionIndex: migrating ================
-- transaction_open?()
   -> 0.0000s
-- index_exists?(:ci_job_artifacts, [:file_type, :project_id, :created_at], {:name=>"index_ci_job_artifacts_on_file_type_for_devops_adoption", :where=>"file_type IN (5,6,8,23)", :algorithm=>:concurrently})
   -> 0.0031s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- add_index(:ci_job_artifacts, [:file_type, :project_id, :created_at], {:name=>"index_ci_job_artifacts_on_file_type_for_devops_adoption", :where=>"file_type IN (5,6,8,23)", :algorithm=>:concurrently})
   -> 0.0050s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210621111747 AddCiArtifactsDevopsAdoptionIndex: migrated (0.0090s) =======

== 20210621155328 ReplaceProjectAuthorizationsProjectIdIndex: migrating =======
-- transaction_open?()
   -> 0.0000s
-- index_exists?(:project_authorizations, [:project_id, :user_id], {:name=>"index_project_authorizations_on_project_id_user_id", :algorithm=>:concurrently})
   -> 0.0012s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- add_index(:project_authorizations, [:project_id, :user_id], {:name=>"index_project_authorizations_on_project_id_user_id", :algorithm=>:concurrently})
   -> 0.0133s
-- execute("RESET statement_timeout")
   -> 0.0002s
-- transaction_open?()
   -> 0.0000s
-- indexes(:project_authorizations)
   -> 0.0013s
-- execute("SET statement_timeout TO 0")
   -> 0.0001s
-- remove_index(:project_authorizations, {:algorithm=>:concurrently, :name=>"index_project_authorizations_on_project_id"})
   -> 0.0034s
-- execute("RESET statement_timeout")
   -> 0.0002s
== 20210621155328 ReplaceProjectAuthorizationsProjectIdIndex: migrated (0.0215s) 

== 20210621164210 DropRemoveOnCloseFromLabels: migrating ======================
-- column_exists?(:labels, :remove_on_close)
   -> 0.0015s
-- remove_column(:labels, :remove_on_close)
   -> 0.0015s
== 20210621164210 DropRemoveOnCloseFromLabels: migrated (0.0041s) =============

== 20210621223000 StealBackgroundJobsThatReferenceServices: migrating =========
== 20210621223000 StealBackgroundJobsThatReferenceServices: migrated (0.0025s) 

== 20210621223242 FinalizeRenameServicesToIntegrations: migrating =============
-- transaction()
-- execute("DROP VIEW IF EXISTS services")
   -> 0.0006s
   -> 0.0006s
== 20210621223242 FinalizeRenameServicesToIntegrations: migrated (0.0007s) ====

== 20210622041846 FinalizePushEventPayloadsBigintConversion: migrating ========
== 20210622041846 FinalizePushEventPayloadsBigintConversion: migrated (0.0000s) 

== 20210622045705 FinalizeEventsBigintConversion: migrating ===================
STDERR: 
---- End output of "bash"  "/tmp/chef-script20211117-8745-1ngw6y0" ----
Ran "bash"  "/tmp/chef-script20211117-8745-1ngw6y0" returned 1

Running handlers complete
Chef Infra Client failed. 29 resources updated in 38 seconds
===
There was an error running gitlab-ctl reconfigure. Please check the output above for more
details.
===

warning: %posttrans(gitlab-ce-14.4.2-ce.0.el7.x86_64) scriptlet failed, exit status 1

#报错解决
#上面提示命令手动执行
]# sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,events,id,'[["id"]\, ["id_convert_to_bigint"]]']
Done.
]# sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,events,id,'[["id"]\, ["id_convert_to_bigint"]]']
Done.

gitlab-ctl reconfigure
gitlab-ctl restart
gitlab-rake db:migrate
gitlab-ctl reconfigure

#官方命令
]# sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,events,id,'[["id"]\, ["id_convert_to_bigint"]]']
]# sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,push_event_payloads,event_id,'[["event_id"]\, ["event_id_convert_to_bigint"]]']

#5分钟后执行gitlab-ctl reconfigure没有任何报错
]# gitlab-ctl reconfigure
   * ruby_block[Delete unmanaged env files for grafana service] action run (skipped due to only_if)
    * template[/opt/gitlab/sv/grafana/check] action create (skipped due to only_if)
    * template[/opt/gitlab/sv/grafana/finish] action create (skipped due to only_if)
    * directory[/opt/gitlab/sv/grafana/control] action create (up to date)
    * link[/opt/gitlab/init/grafana] action create (up to date)
    * file[/opt/gitlab/sv/grafana/down] action nothing (skipped due to action :nothing)
    * directory[/opt/gitlab/service] action create (up to date)
    * link[/opt/gitlab/service/grafana] action create (up to date)
    * ruby_block[wait for grafana service socket] action run (skipped due to not_if)
     (up to date)
Recipe: gitlab::database_reindexing_disable
  * crond_job[database-reindexing] action delete
    * file[/var/opt/gitlab/crond/database-reindexing] action delete (up to date)
     (up to date)

Running handlers:
Running handlers complete
Chef Infra Client finished, 3/769 resources updated in 22 seconds
gitlab Reconfigured!


#查看后台迁移
gitlab-psql
select job_class_name, table_name, column_name, job_arguments from batched_background_migrations where status <> 3;

gitlab-rails runner -e production 'puts Gitlab::BackgroundMigration.remaining'

#重新启动服务
]# gitlab-ctl restart

#查看当前服务状态
]# gitlab-ctl status

#加载配置：
]# gitlab-ctl reconfigure

#加载配置后状态查看
]# gitlab-ctl status

#版本确认：
]# rpm -qa gitlab-ce
gitlab-ce-14.0.12-ce.0.el7.x86_64
]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
14.0.12


```

> #手动运行这两条

![image-20211117161919448](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031212.png)

> #页面访问版本验证

![image-20211117165035728](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111181031214.png)

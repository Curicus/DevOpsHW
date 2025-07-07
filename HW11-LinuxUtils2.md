# Unix утилиты №2
## Task 1 Работа с файловой системой
### 1. Добавить к ВМ три дополнительных диска небольшого размера
```
user@Lab5:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   16G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 14.2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0    5G  0 disk
sdc                         8:32   0    5G  0 disk
sdd                         8:48   0    5G  0 disk
sr0                        11:0    1 1024M  0 rom
```
### 2. Собрать из двух дополнительных дисков raid-1 (имя устройства задать /dev/md1)
```
user@Lab5:~$ sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
user@Lab5:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                         8:0    0   16G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part  /boot
└─sda3                      8:3    0 14.2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm   /
sdb                         8:16   0    5G  0 disk
└─md0                       9:0    0    5G  0 raid1
sdc                         8:32   0    5G  0 disk
└─md0                       9:0    0    5G  0 raid1
sdd                         8:48   0    5G  0 disk
sr0                        11:0    1 1024M  0 rom
```
### 3. Из полученного массива /dev/md1 и третьего диска собрать volume group (lvm) с именем vg_raid
   ```
   user@Lab5:~$ sudo vgcreate vg_raid /dev/md0 /dev/sdd
   user@Lab5:~$ sudo pvs
   PV         VG        Fmt  Attr PSize   PFree
   /dev/md0   vg_raid   lvm2 a--    4.99g  4.99g
   /dev/sda3  ubuntu-vg lvm2 a--  <14.25g <4.25g
   /dev/sdd   vg_raid   lvm2 a--   <5.00g <5.00g
   ```
### 4. От созданной VG vg_raid отрезать logical volume размером 3Gb и именем mysql-app1; создать на разделе файловую систему xfs
   ```
   user@Lab5:~$ sudo lvcreate -L 3G -n mysql-app1 vg_raid
   Logical volume "mysql-app1" created.
   user@Lab5:~$ sudo mkfs.xfs /dev/vg_raid/mysql-app1
   meta-data=/dev/vg_raid/mysql-app1 isize=512    agcount=4, agsize=196608 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=1, sparse=1, rmapbt=1
            =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
   data     =                       bsize=4096   blocks=786432, imaxpct=25
            =                       sunit=0      swidth=0 blks
   naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
   log      =internal log           bsize=4096   blocks=16384, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   ```
### 5. Установить на ВМ mariadb-server `sudo apt install mariadb-server`
   
### 6. Вынести папку с данными mariadb-server на созданный logical volume mysql-app1. Обязательно проверить, что БД после всех манипуляций запустилась и живая.
> Тормознем службу `sudo systemctl stop mariadb.service`
> Создадим дирректорию для временного монтирования нашего volume и примонтируем его
```
user@Lab5:~$ sudo mkdir -p /mnt/mysql
user@Lab5:~$ sudo mount /dev/vg_raid/mysql-app1 /mnt/mysql
```
> Сделаем перенос всей дерриктории mysql и чекнем что там все хоошо
```
user@Lab5:~$ sudo rsync -avz /var/lib/mysql/
user@Lab5:~$ ls -l /mnt/mysql
total 111036
-rw-rw---- 1 mysql mysql    417792 Jul  7 10:11 aria_log.00000001
-rw-rw---- 1 mysql mysql        52 Jul  7 10:11 aria_log_control
-rw-r--r-- 1 root  root          0 Jul  7 08:01 debian-10.11.flag
-rw-rw---- 1 mysql mysql       910 Jul  7 10:11 ib_buffer_pool
-rw-rw---- 1 mysql mysql  12582912 Jul  7 08:01 ibdata1
-rw-rw---- 1 mysql mysql 100663296 Jul  7 08:01 ib_logfile0
-rw-rw---- 1 mysql mysql         0 Jul  7 08:01 multi-master.info
drwx------ 2 mysql mysql      8192 Jul  7 08:01 mysql
-rw-r--r-- 1 root  root         16 Jul  7 08:01 mysql_upgrade_info
drwx------ 2 mysql mysql        20 Jul  7 08:01 performance_schema
drwx------ 2 mysql mysql      8192 Jul  7 08:01 sys
```
> Перекинем в бэкап основнуюдирректорию базы `user@Lab5:~$ sudo mv /var/lib/mysql /var/lib/mysql.bak`
> Создадим основную точку монтирования нашего volume и отмонтируем от временной точки
```
user@Lab5:~$ sudo umount /mnt/mysql
user@Lab5:~$ sudo mkdir /var/lib/mysql
user@Lab5:~$ sudo blkid /dev/vg_raid/mysql-app1
/dev/vg_raid/mysql-app1: UUID="aba7cf19-38e6-4688-87fc-f50dc8e9e109" BLOCK_SIZE="512" TYPE="xfs" 
user@Lab5:~$ sudo vim /etc/fstab
```
> Вставим туда скопированный UUID `UUID=aba7cf19-38e6-4688-87fc-f50dc8e9e109 /var/lib/mysql xfs defaults 0 0` и перечитаем демона
`user@Lab5:~$ sudo systemctl daemon-reload`
> Проверим что наша база находится там где надо
```
user@Lab5:~$ df -h | grep mysql
/dev/mapper/vg_raid-mysql--app1    3.0G  203M  2.8G   7% /var/lib/mysql
``` 
> Запустим базу и проверим
```
user@Lab5:~$ sudo systemctl start mariadb
user@Lab5:~$ sudo systemctl status mariadb
● mariadb.service - MariaDB 10.11.13 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-07-07 11:06:14 UTC; 5s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 16014 ExecStartPre=/usr/bin/install -m 755 -o mysql -g root -d /var/run/mysqld (code=exited, status=0/SUCCESS)
    Process: 16016 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 16019 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`/usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-environment _WSREP_START_POSITION=$VAR || exit 1 (co>
    Process: 16093 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 16095 ExecStartPost=/etc/mysql/debian-start (code=exited, status=0/SUCCESS)
   Main PID: 16079 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 13 (limit: 46311)
     Memory: 78.8M (peak: 82.3M)
        CPU: 1.621s
     CGroup: /system.slice/mariadb.service
             └─16079 /usr/sbin/mariadbd
```

### 7. Увеличить размер mysql-app1 до 4G
> На vg_raid было всего 10GB, значит мы спокойно может еще 1 Gb откусить для нашего volume
```
user@Lab5:~$ sudo lvresize -L +1G /dev/vg_raid/mysql-app1
user@Lab5:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg_raid/mysql-app1
  LV Name                mysql-app1
  VG Name                vg_raid
  LV UUID                qMU0py-0wLP-rrK1-lEzJ-6jzH-COJ2-pjx2rd
  LV Write Access        read/write
  LV Creation host, time Lab5, 2025-07-07 07:46:25 +0000
  LV Status              available
  # open                 1
  LV Size                4.00 GiB
  Current LE             1024
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1
```
### 8. Оставшееся в volume group место отдать новому logical volume с именем app1 и файловой системой ext4, который смонтировать в /opt/data-app1
```
ser@Lab5:~$ sudo lvcreate -l 100%FREE -n app1 vg_raid
  Logical volume "app1" created.
user@Lab5:~$ sudo mkfs.ext4 /dev/vg_raid/app1
user@Lab5:~$ sudo mkdir -p /opt/data-app1
user@Lab5:~$ sudo mount /dev/vg_raid/app1 /opt/data-app1
```


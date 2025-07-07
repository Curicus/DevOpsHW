# Unix утилиты №2
## Task 1 Работа с файловой системой
1. Добавить к ВМ три дополнительных диска небольшого размера
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
2. Собрать из двух дополнительных дисков raid-1 (имя устройства задать /dev/md1)
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
3. Из полученного массива /dev/md1 и третьего диска собрать volume group (lvm) с именем vg_raid
   ```
   user@Lab5:~$ sudo vgcreate vg_raid /dev/md0 /dev/sdd
   user@Lab5:~$ sudo pvs
   PV         VG        Fmt  Attr PSize   PFree
   /dev/md0   vg_raid   lvm2 a--    4.99g  4.99g
   /dev/sda3  ubuntu-vg lvm2 a--  <14.25g <4.25g
   /dev/sdd   vg_raid   lvm2 a--   <5.00g <5.00g
   ```
4. От созданной VG vg_raid отрезать logical volume размером 3Gb и именем mysql-app1; создать на разделе файловую систему xfs
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
5. Установить на ВМ mariadb-server `sudo apt install mariadb-server`
   
7. Вынести папку с данными mariadb-server на созданный logical volume mysql-app1. Обязательно проверить, что БД после всех манипуляций запустилась и живая.






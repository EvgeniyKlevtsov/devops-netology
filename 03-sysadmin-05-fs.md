## Домашнее задание к занятию "3.5. Файловые системы"

### 2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
Нет, т.к. жесткая ссылка полностью аналогична файлу, на который создана и включает те же разрешения, что выставлены на оригинальном файле.

### 3. Сделайте vagrant destroy на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile содержимым из примера.
Готово.

### 4. Используя fdisk, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
Используем `lsblk` найдём нужные нам устройства, а потом при помощи `fdisk /dev/sdb` создадим разделы.
```
root@vagrant:~# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xeffb2ad2.

Command (m for help): g
Created a new GPT disklabel (GUID: 7E95D682-25C5-534D-A6D0-D755CE97A0F9).

Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-5242846, default 2048): 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242846, default 5242846): +2G	

Created a new partition 1 of type 'Linux filesystem' and of size 2 GiB.

Command (m for help): p
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 7E95D682-25C5-534D-A6D0-D755CE97A0F9

Device     Start     End Sectors Size Type
/dev/sdb1   2048 4196351 4194304   2G Linux filesystem

Command (m for help): n
Partition number (2-128, default 2): 2
First sector (4196352-5242846, default 4196352): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242846, default 5242846): 

Created a new partition 2 of type 'Linux filesystem' and of size 511 MiB.

Command (m for help): p
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 7E95D682-25C5-534D-A6D0-D755CE97A0F9

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 5242846 1046495  511M Linux filesystem

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

root@vagrant:~# 
```
### 5. Используя sfdisk, перенесите данную таблицу разделов на второй диск.
```
root@vagrant:~# sfdisk -d /dev/sdb | sfdisk /dev/sdc
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new GPT disklabel (GUID: 7E95D682-25C5-534D-A6D0-D755CE97A0F9).
/dev/sdc1: Created a new partition 1 of type 'Linux filesystem' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux filesystem' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: gpt
Disk identifier: 7E95D682-25C5-534D-A6D0-D755CE97A0F9

Device       Start     End Sectors  Size Type
/dev/sdc1     2048 4196351 4194304    2G Linux filesystem
/dev/sdc2  4196352 5242846 1046495  511M Linux filesystem

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
### 6. Соберите mdadm RAID1 на паре разделов 2 Гб.
Выведем информацию о устройствах:
```
root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk 
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part 
└─sda5                 8:5    0 63.5G  0 part 
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk 
├─sdb1                 8:17   0    2G  0 part 
└─sdb2                 8:18   0  511M  0 part 
sdc                    8:32   0  2.5G  0 disk 
├─sdc1                 8:33   0    2G  0 part 
└─sdc2                 8:34   0  511M  0 part 
root@vagrant:~# 
```
Создадим RAID1 из нужных дисков:
```
root@vagrant:~# mdadm -C -v /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
root@vagrant:~# 
```
### 7. Соберите mdadm RAID0 на второй паре маленьких разделов.
```
root@vagrant:~# mdadm -C -v /dev/md1 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
root@vagrant:~#
```
Посмотрим что получилось в итоге.
```
root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk  
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part  
└─sda5                 8:5    0 63.5G  0 part  
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk  
├─sdb1                 8:17   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdb2                 8:18   0  511M  0 part  
  └─md1                9:1    0 1017M  0 raid0 
sdc                    8:32   0  2.5G  0 disk  
├─sdc1                 8:33   0    2G  0 part  
│ └─md0                9:0    0    2G  0 raid1 
└─sdc2                 8:34   0  511M  0 part  
  └─md1                9:1    0 1017M  0 raid0 
root@vagrant:~# 
```
### 8. Создайте 2 независимых PV на получившихся md-устройствах.
```
root@vagrant:~# cd /home/vagrant/
root@vagrant:/home/vagrant# pvcreate /dev/md0
  Physical volume "/dev/md0" successfully created.
root@vagrant:/home/vagrant# pvcreate /dev/md1
  Physical volume "/dev/md1" successfully created.
```
### 9. Создайте общую volume-group на этих двух PV.
Вот что получилось в итоге:
```
root@vagrant:/home/vagrant# vgcreate Volume_group /dev/md0 /dev/md1
  Volume group "Volume_group" successfully created
root@vagrant:/home/vagrant# pvdisplay 
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               vgvagrant
  PV Size               <63.50 GiB / not usable 0   
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              16255
  Free PE               0
  Allocated PE          16255
  PV UUID               Mx3LcA-uMnN-h9yB-gC2w-qm7w-skx0-OsTz9z
   
  --- Physical volume ---
  PV Name               /dev/md0
  VG Name               Volume_group
  PV Size               <2.00 GiB / not usable 0   
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              511
  Free PE               511
  Allocated PE          0
  PV UUID               2P5B8u-O3Eg-nPAm-Xzt2-QENR-p307-KY5oR6
   
  --- Physical volume ---
  PV Name               /dev/md1
  VG Name               Volume_group
  PV Size               1017.00 MiB / not usable 0   
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              254
  Free PE               254
  Allocated PE          0
  PV UUID               pKTblS-4CXj-CdTB-L0sI-LkSS-H8Kr-2vO0PU  
```
### 10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
```
root@vagrant:/home/vagrant# lvcreate -n Logical_volume -L 100M Volume_group /dev/md1
  Logical volume "Logical_volume" created.
root@vagrant:/home/vagrant# 
```
### 11. Создайте mkfs.ext4 ФС на получившемся LV.
```
root@vagrant:/home/vagrant# mkfs.ext4 /dev/Volume_group/Logical_volume 
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```
### 12. Смонтируйте этот раздел в любую директорию, например, /tmp/new.
```
root@vagrant:/home/vagrant# mkdir /tmp/new
root@vagrant:/home/vagrant# mount /dev/Volume_group/Logical_volume  /tmp/new/
```
### 13. 1.  Поместите туда тестовый файл, например  `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
```
root@vagrant:/home/vagrant# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2021-12-01 19:59:22--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22731271 (22M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz             100%[==============================================>]  21.68M   332KB/s    in 67s     

2021-12-01 20:00:29 (332 KB/s) - ‘/tmp/new/test.gz’ saved [22731271/22731271]
```
### 14. Прикрепите вывод lsblk.
```
root@vagrant:/home/vagrant# lsblk 
NAME                              MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                                 8:0    0   64G  0 disk  
├─sda1                              8:1    0  512M  0 part  /boot/efi
├─sda2                              8:2    0    1K  0 part  
└─sda5                              8:5    0 63.5G  0 part  
  ├─vgvagrant-root                253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1              253:1    0  980M  0 lvm   [SWAP]
sdb                                 8:16   0  2.5G  0 disk  
├─sdb1                              8:17   0    2G  0 part  
│ └─md0                             9:0    0    2G  0 raid1 
└─sdb2                              8:18   0  511M  0 part  
  └─md1                             9:1    0 1017M  0 raid0 
    └─Volume_group-Logical_volume 253:2    0  100M  0 lvm   /tmp/new
sdc                                 8:32   0  2.5G  0 disk  
├─sdc1                              8:33   0    2G  0 part  
│ └─md0                             9:0    0    2G  0 raid1 
└─sdc2                              8:34   0  511M  0 part  
  └─md1                             9:1    0 1017M  0 raid0 
    └─Volume_group-Logical_volume 253:2    0  100M  0 lvm   /tmp/new
root@vagrant:/home/vagrant# 
```
### 15. Протестируйте целостность файла:
```
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
```
готово
### 16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
```
root@vagrant:/home/vagrant# pvmove /dev/md1 /dev/md0
  /dev/md1: Moved: 12.00%
  /dev/md1: Moved: 100.00%
root@vagrant:/home/vagrant#
```
### 17. Сделайте --fail на устройство в вашем RAID1 md.
```
root@vagrant:/home/vagrant# mdadm /dev/md0 --fail /dev/sdc1
mdadm: set /dev/sdc1 faulty in /dev/md0
root@vagrant:/home/vagrant# 
```
### 18. Подтвердите выводом dmesg, что RAID1 работает в деградированном состоянии.
```
[ 3265.786243]  sdb: sdb1 sdb2
[ 3492.299782]  sdc: sdc1 sdc2
[ 4412.175195] md/raid1:md0: not clean -- starting background reconstruction
[ 4412.175198] md/raid1:md0: active with 2 out of 2 mirrors
[ 4412.175222] md0: detected capacity change from 0 to 2144337920
[ 4412.178289] md: resync of RAID array md0
[ 4422.560652] md: md0: resync done.
[ 4648.236295] md1: detected capacity change from 0 to 1066401792
[ 6074.314212] EXT4-fs (dm-2): mounted filesystem with ordered data mode. Opts: (null)
[ 6074.314228] ext4 filesystem being mounted at /tmp/new supports timestamps until 2038 (0x7fffffff)
[ 6765.015975] md/raid1:md0: Disk failure on sdc1, disabling device.
               md/raid1:md0: Operation continuing on 1 devices.
root@vagrant:/home/vagrant# 
```

### 19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:
```
root@vagrant:/home/vagrant# gzip -t /tmp/new/test.gz
root@vagrant:/home/vagrant# echo $?
0
root@vagrant:/home/vagrant# 
```
### 20. Погасите тестовый хост,  `vagrant destroy`.
Готово.

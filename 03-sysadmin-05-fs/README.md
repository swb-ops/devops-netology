### 3.5. Файловые системы

---
1. Узнайте о sparse (разряженных) файлах.  
Прочитал.

---
2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

Не могут, так как ссылаются на один и тот же индексный дескриптор.
```
vshchepkin@vshchepkin:~/link$ ln link_orig link_hard1
vshchepkin@vshchepkin:~/link$ ln link_orig link_hard2
vshchepkin@vshchepkin:~/link$ ll
-rw-rw-r--  3 vshchepkin vshchepkin     8 Jul  8 18:00 link_hard1
-rw-rw-r--  3 vshchepkin vshchepkin     8 Jul  8 18:00 link_hard2
-rw-rw-r--  3 vshchepkin vshchepkin     8 Jul  8 18:00 link_orig
vshchepkin@vshchepkin:~/link$ chown vshchepkin:www-data link_hard2
vshchepkin@vshchepkin:~/link$ ll
-rw-rw-r--  3 vshchepkin www-data       8 Jul  8 18:00 link_hard1
-rw-rw-r--  3 vshchepkin www-data       8 Jul  8 18:00 link_hard2
-rw-rw-r--  3 vshchepkin www-data       8 Jul  8 18:00 link_orig
```

---
3. Создать ВМ  двумя дополнительными неразмеченными дисками по 2.5 Гб.

Имею ВМ на VmWare, подключил два доп диска:
```
root@vshchepkin:/# fdisk -l | grep sd
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
/dev/sda1     2048      4095      2048   1M BIOS boot
/dev/sda2     4096   2101247   2097152   1G Linux filesystem
/dev/sda3  2101248 104855551 102754304  49G Linux filesystem
Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
```

---
4. Используя fdisk, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

Разбил с помощью fdisk /dev/sdb на 2 раздела
```
root@vshchepkin:/#fdisk -l | grep sd
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
/dev/sda1     2048      4095      2048   1M BIOS boot
/dev/sda2     4096   2101247   2097152   1G Linux filesystem
/dev/sda3  2101248 104855551 102754304  49G Linux filesystem
Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
/dev/sdb1          2048 4196351 4194304    2G 8e Linux LVM
/dev/sdb2       4196352 5242879 1046528  511M 8e Linux LVM
```

---
5. Используя sfdisk, перенесите данную таблицу разделов на второй диск.
```
root@vshchepkin:/# sfdisk -d /dev/sdb > pert_table_sdb1 
root@vshchepkin:/# sfdisk /dev/sdc < pert_table_sdb1

root@vshchepkin:/# fdisk -l | grep 'sd.*'
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
/dev/sda1     2048      4095      2048   1M BIOS boot
/dev/sda2     4096   2101247   2097152   1G Linux filesystem
/dev/sda3  2101248 104855551 102754304  49G Linux filesystem
Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
/dev/sdc1          2048 4196351 4194304    2G 8e Linux LVM
/dev/sdc2       4196352 5242879 1046528  511M 8e Linux LVM
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
/dev/sdb1          2048 4196351 4194304    2G 8e Linux LVM
/dev/sdb2       4196352 5242879 1046528  511M 8e Linux LVM
```

---
6. Соберите mdadm RAID1 на паре разделов 2 Гб.
```
root@vshchepkin:/# mdadm --zero-superblock --force /dev/sd{b1,c1}
mdadm: Unrecognised md component device - /dev/sdb1
mdadm: Unrecognised md component device - /dev/sdc1
root@vshchepkin:/# wipefs --all --force /dev/sd{b1,c1}
root@vshchepkin:/# mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sd{b1,c1}
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

root@vshchepkin:/# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop  /snap/core18/2074
loop1                       7:1    0 55.4M  1 loop  /snap/core18/2066
loop2                       7:2    0 69.9M  1 loop  /snap/lxd/19188
loop3                       7:3    0 67.6M  1 loop  /snap/lxd/20326
loop4                       7:4    0 32.3M  1 loop  /snap/snapd/12398
loop5                       7:5    0 32.3M  1 loop  /snap/snapd/12159
sda                         8:0    0   50G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   49G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0   49G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md0                     9:0    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md0                     9:0    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
```

---
7. Соберите mdadm RAID0 на второй паре маленьких разделов.

```
root@vshchepkin:/# mdadm --zero-superblock --force /dev/sd{b2,c2}
mdadm: Unrecognised md component device - /dev/sdb2
mdadm: Unrecognised md component device - /dev/sdc2
root@vshchepkin:/# wipefs --all --force /dev/sd{b2,c2}
root@vshchepkin:/# mdadm --create --verbose /dev/md1 -l 0 -n 2 /dev/sd{b2,c2}  
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
root@vshchepkin:/# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop  /snap/core18/2074
loop1                       7:1    0 55.4M  1 loop  /snap/core18/2066
loop2                       7:2    0 69.9M  1 loop  /snap/lxd/19188
loop3                       7:3    0 67.6M  1 loop  /snap/lxd/20326
loop4                       7:4    0 32.3M  1 loop  /snap/snapd/12398
loop5                       7:5    0 32.3M  1 loop  /snap/snapd/12159
sda                         8:0    0   50G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   49G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0   49G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md0                     9:0    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md1                     9:1    0 1018M  0 raid0 
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md0                     9:0    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md1                     9:1    0 1018M  0 raid0
```

---
8. Создайте 2 независимых PV на получившихся md-устройствах.
```
root@vshchepkin:/# pvcreate /dev/md0
root@vshchepkin:/# pvcreate /dev/md1
root@vshchepkin:/# pvs
  PV         VG        Fmt  Attr PSize    PFree   
  /dev/md0             lvm2 ---    <2.00g   <2.00g
  /dev/md1             lvm2 ---  1018.00m 1018.00m
```

---
9. Создайте общую volume-group на этих двух PV.
```
root@vshchepkin:/# vgcreate vg1 /dev/md0 /dev/md1
  Volume group "vg1" successfully created
root@vshchepkin:/# vgs
  VG        #PV #LV #SN Attr   VSize   VFree 
  ubuntu-vg   1   1   0 wz--n- <49.00g     0 
  vg1         2   0   0 wz--n-  <2.99g <2.99g
```

---
10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
```
root@vshchepkin:/# lvcreate -L 100M -n lv_raid0 vg1 /dev/md1
  Logical volume "lv_raid0" created.
root@vshchepkin:/# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao---- <49.00g                                                    
  lv_raid0  vg1       -wi-a----- 100.00m
```

---
11. Создайте mkfs.ext4 ФС на получившемся LV.
```
root@vshchepkin:/# mkfs.ext4 /dev/vg1/lv_raid0
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```
---
12. Смонтируйте этот раздел в любую директорию, например, /tmp/new.
```
root@vshchepkin:/# mount /dev/vg1/lv_raid0 /tmp/new
```
---
13. Поместите туда тестовый файл, например wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz.
```
root@vshchepkin:/# ls /tmp/new/
lost+found  test.gz
```
---
14. Прикрепите вывод lsblk.
```
root@vshchepkin:/# lsblk 
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop  /snap/core18/2074
loop1                       7:1    0 55.4M  1 loop  /snap/core18/2066
loop2                       7:2    0 69.9M  1 loop  /snap/lxd/19188
loop3                       7:3    0 67.6M  1 loop  /snap/lxd/20326
loop4                       7:4    0 32.3M  1 loop  /snap/snapd/12398
loop5                       7:5    0 32.3M  1 loop  /snap/snapd/12159
sda                         8:0    0   50G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   49G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0   49G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md0                     9:0    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md1                     9:1    0 1018M  0 raid0 
    └─vg1-lv_raid0        253:1    0  100M  0 lvm   /tmp/new
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md0                     9:0    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md1                     9:1    0 1018M  0 raid0 
    └─vg1-lv_raid0        253:1    0  100M  0 lvm   /tmp/new
sr0                        11:0    1  1.1G  0 rom   
```
---
15. Протестируйте целостность файла:
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
```
root@vshchepkin:/# gzip -t /tmp/new/test.gz
root@vshchepkin:/# echo $?
0
```
---
16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
```
root@vshchepkin:/# pvmove /dev/md1 /dev/md0
  /dev/md1: Moved: 12.00%
  /dev/md1: Moved: 100.00%
root@vshchepkin:/# lsblk 
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop  /snap/core18/2074
loop1                       7:1    0 55.4M  1 loop  /snap/core18/2066
loop2                       7:2    0 69.9M  1 loop  /snap/lxd/19188
loop3                       7:3    0 67.6M  1 loop  /snap/lxd/20326
loop4                       7:4    0 32.3M  1 loop  /snap/snapd/12398
loop5                       7:5    0 32.3M  1 loop  /snap/snapd/12159
sda                         8:0    0   50G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   49G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0   49G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md0                     9:0    0    2G  0 raid1 
│   └─vg1-lv_raid0        253:1    0  100M  0 lvm   /tmp/new
└─sdb2                      8:18   0  511M  0 part  
  └─md1                     9:1    0 1018M  0 raid0 
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md0                     9:0    0    2G  0 raid1 
│   └─vg1-lv_raid0        253:1    0  100M  0 lvm   /tmp/new
└─sdc2                      8:34   0  511M  0 part  
  └─md1                     9:1    0 1018M  0 raid0 
sr0                        11:0    1  1.1G  0 rom   
```
---
17. Сделайте --fail на устройство в вашем RAID1 md.
```
root@vshchepkin:/# mdadm /dev/md0 --fail /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md0
```
---
18. Подтвердите выводом dmesg, что RAID1 работает в деградированном состоянии.
```
root@vshchepkin:/# dmesg -T | tail -n 10
[Fri Jul  9 17:27:42 2021] md0: detected capacity change from 0 to 2144337920
[Fri Jul  9 17:27:42 2021] md: resync of RAID array md0
[Fri Jul  9 17:28:34 2021] md: md0: resync done.
[Fri Jul  9 17:31:22 2021] md1: detected capacity change from 0 to 1067450368
[Sat Jul 10 17:34:17 2021] EXT4-fs (dm-1): mounted filesystem with ordered data mode. Opts: (null)
[Sat Jul 10 17:34:17 2021] ext4 filesystem being mounted at /tmp/new supports timestamps until 2038 (0x7fffffff)
[Sat Jul 10 17:47:38 2021] EXT4-fs (dm-1): mounted filesystem with ordered data mode. Opts: (null)
[Sat Jul 10 17:47:38 2021] ext4 filesystem being mounted at /tmp/new supports timestamps until 2038 (0x7fffffff)
[Sat Jul 10 17:51:00 2021] md/raid1:md0: Disk failure on sdb1, disabling device.
                           md/raid1:md0: Operation continuing on 1 devices.
```                                                      
---
19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
```
root@vshchepkin:/# gzip -t /tmp/new/test.gz                                         
root@vshchepkin:/# echo $?
0
```

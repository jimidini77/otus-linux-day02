# otus-linux-day02
Работа с `mdadm`

# **Содержание ДЗ**

* добавление в Vagrantfile дисков
* сборка RAID5
* запись конфигурации RAID в конфиг для сборки массива при загрузке
* вывод из строя дисков в RAID и починка
* создание GPT-раздела, 5 партиций и монтирование их

# **Выполнение**

В Vagrantfile добавлены диски

```
	:disks => {
		:sata1 => {
			:dfile => './sata1.vdi', #файл диска лежит в каталоге с Vagrantfile
			:size => 100,            #размер 100 мегабайт
			:port => 1               #номер порта
		},
		:sata2 => {
                        :dfile => './sata2.vdi',
                        :size => 100, 
			:port => 2
		},
                :sata3 => {
                        :dfile => './sata3.vdi',
                        :size => 100,
                        :port => 3
                },
                :sata4 => {
                        :dfile => './sata4.vdi',
                        :size => 100, 
                        :port => 4
                },
                :sata5 => {
                        :dfile => './sata5.vdi',
                        :size => 100, 
                        :port => 5
                },
                :sata6 => {
                        :dfile => './sata6.vdi',
                        :size => 100, 
                        :port => 6
                }
	}
```

На четырёх дисках создан RAID-5, добавлены 2 spare-диска

```
mdadm --create /dev/md0 -l 5 -n 4 /dev/sd{b,c,d,e}
mdadm /dev/md0 --add /dev/sd{f,g}
```
Создан конфигурационный файл mdadm.conf
```
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
```

На собранном массиве создана GPT таблица разделов с пятью разделами размерами по 20% от общей емкости массива
```                                                          
parted -s /dev/md0 mklabel gpt
for i in $(seq 0 20 80); do parted -s /dev/md0 mkpart primary ext4 $i% $((i+20))%; done 
```

Созданные разделы отформатированы в ext4 и примонтированы к каталогам
```
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
mkdir -p /raid/part{1,2,3,4,5}
for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
```

Имитация поломки диска пометкой одного диска как сбойного

```
[vagrant@otuslinux ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sat May 14 15:19:43 2022
        Raid Level : raid5
        Array Size : 301056 (294.00 MiB 308.28 MB)
     Used Dev Size : 100352 (98.00 MiB 102.76 MB)
      Raid Devices : 4
     Total Devices : 6
       Persistence : Superblock is persistent

       Update Time : Sat May 14 15:21:27 2022
             State : clean
    Active Devices : 4
   Working Devices : 6
    Failed Devices : 0
     Spare Devices : 2

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 761768fc:7f6ff61d:6d353f24:14e1c4a5
            Events : 26

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde

       5       8       80        -      spare   /dev/sdf
       6       8       96        -      spare   /dev/sdg
```

После выполнения `mdadm /dev/md0 --fail /dev/sdb` один из spare-дисков задействован вместо сбойного
```
[vagrant@otuslinux ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sat May 14 15:19:43 2022
        Raid Level : raid5
        Array Size : 301056 (294.00 MiB 308.28 MB)
     Used Dev Size : 100352 (98.00 MiB 102.76 MB)
      Raid Devices : 4
     Total Devices : 6
       Persistence : Superblock is persistent

       Update Time : Sat May 14 16:13:20 2022
             State : clean
    Active Devices : 4
   Working Devices : 5
    Failed Devices : 1
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 761768fc:7f6ff61d:6d353f24:14e1c4a5
            Events : 45

    Number   Major   Minor   RaidDevice State
       6       8       96        0      active sync   /dev/sdg
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde

       0       8       16        -      faulty   /dev/sdb
       5       8       80        -      spare   /dev/sdf
```

Vagrantfile содержит inline-script, выполняющий при провижининге 
- создание массива
- запись конфигурационного файла
- создание GPT-таблицы
- создание разделов, их форматирование, монтирование и запись в fstab конфига, для автоматического монтирования при загрузке системы

## **Issues**

При провижининге у создаваемых в `/etc/` файлов конфигов временно менялись права доступа, т.к. у юзера `vagrant` Не хватало прав на запись


# **Результаты**

Полученный в ходе работы Vagrantfile помещен в публичный репозиторий:
- **GitHub** - https://github.com/jimidini77/otus-linux-day02

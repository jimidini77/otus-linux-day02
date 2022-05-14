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

Vagrantfile содержит inline-script, выполняющий при провижининге 
- создание массива
- запись конфигурационного файла
- создание GPT-таблицы
- создание разделов, их форматирование, монтирование и запись в fstab конфига, для автоматического монтирования при загрузке системы

## **Issues**

При провижининге у создаваемых в `/etc/` файлов конфигов временно менялись права доступа, т.к. у юзера `vagrant` Не хватало прав на запись


# **Результаты**

Полученные в ходе работы vagrant box и конфигурационные файлы помещены в публичные репозитории:
- **GitHub** - https://github.com/jimidini77/manual_kernel_update/
- **Vagrant Cloud** - https://app.vagrantup.com/jimidini77/boxes/centos-7-5



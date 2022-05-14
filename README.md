# otus-linux-day02
������ � `mdadm`

# **���������� ��**

* ���������� � Vagrantfile ������
* ������ RAID5
* ������ ������������ RAID � ������ ��� ������ ������� ��� ��������
* ����� �� ����� ������ � RAID � �������
* �������� GPT-�������, 5 �������� � ������������ ��

# **����������**

� Vagrantfile ��������� �����

```
	:disks => {
		:sata1 => {
			:dfile => './sata1.vdi', #���� ����� ����� � �������� � Vagrantfile
			:size => 100,            #������ 100 ��������
			:port => 1               #����� �����
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

�� ������ ������ ������ RAID-5, ��������� 2 spare-�����

```
mdadm --create /dev/md0 -l 5 -n 4 /dev/sd{b,c,d,e}
mdadm /dev/md0 --add /dev/sd{f,g}
```
������ ���������������� ���� mdadm.conf
```
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
```

�� ��������� ������� ������� GPT ������� �������� � ����� ��������� ��������� �� 20% �� ����� ������� �������
```                                                          
parted -s /dev/md0 mklabel gpt
for i in $(seq 0 20 80); do parted -s /dev/md0 mkpart primary ext4 $i% $((i+20))%; done 
```

��������� ������� ��������������� � ext4 � �������������� � ���������
```
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
mkdir -p /raid/part{1,2,3,4,5}
for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
```

�������� ������� ����� �������� ������ ����� ��� ��������

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

����� ���������� `mdadm /dev/md0 --fail /dev/sdb` ���� �� spare-������ ������������ ������ ��������
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

Vagrantfile �������� inline-script, ����������� ��� ������������ 
- �������� �������
- ������ ����������������� �����
- �������� GPT-�������
- �������� ��������, �� ��������������, ������������ � ������ � fstab �������, ��� ��������������� ������������ ��� �������� �������

## **Issues**

��� ������������ � ����������� � `/etc/` ������ �������� �������� �������� ����� �������, �.�. � ����� `vagrant` �� ������� ���� �� ������


# **����������**

���������� � ���� ������ Vagrantfile ������� � ��������� �����������:
- **GitHub** - https://github.com/jimidini77/otus-linux-day02



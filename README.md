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

Vagrantfile �������� inline-script, ����������� ��� ������������ 
- �������� �������
- ������ ����������������� �����
- �������� GPT-�������
- �������� ��������, �� ��������������, ������������ � ������ � fstab �������, ��� ��������������� ������������ ��� �������� �������

## **Issues**

��� ������������ � ����������� � `/etc/` ������ �������� �������� �������� ����� �������, �.�. � ����� `vagrant` �� ������� ���� �� ������


# **����������**

���������� � ���� ������ vagrant box � ���������������� ����� �������� � ��������� �����������:
- **GitHub** - https://github.com/jimidini77/manual_kernel_update/
- **Vagrant Cloud** - https://app.vagrantup.com/jimidini77/boxes/centos-7-5



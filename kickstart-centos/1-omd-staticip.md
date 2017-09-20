## File kickstart cài đặt CentOS 7

### Mục đích sử dụng:

- Cài đặt IP tĩnh cho 2 card mạng
	- Card PXE: 172.16.1.101
	- Card LAN2: 192.168.100.145

	- **Chú ý**: 
		- Card PXE phải ra được Internet để cài đặt Check_MK
		- Card LAN2, không được cắm dây trước khi cài nếu dải đó có DHCP Server.
		
- Cài đặt Check_MK
- Cài đặt LVM:
	- Sử dụng ổ sda
	- Tạo Physical Volume tên `pv.20` dung lượng bằng với `sda`
	- /boot : 500MB
	- Tạo Volume Group tên `centos`
	- Tạo Logical Volume tên `root`, dung lượng tối thiểu là 1G, tối đa là 50GB, dung lượng giãn nở (--grow) định dạng "xfs" được mount vào `/`
	- Tạo Logical Volume tên `swap` dung lượng 2GB để làm SWAP 

### File cấu hình

```
version=RHEL7
# System authorization information
auth --enableshadow --passalgo=sha512

# Use network installation
url --url="http://172.16.1.254/images/centos7_x64"
install

%post
#!/bin/sh
NICs=$(cat /proc/net/dev | awk {'print $1'} | grep -Ev "^lo|^face|^Inter|^$" | cut -d ":" -f1)

# Khai bao IP PREFIX GATEWAY
NIC1="192.168.100.145 24 192.168.100.1"
NIC0="172.16.1.101 24 172.16.1.254"
i=0
for x in $NICs
do
	NIC=($NICs)
	eval DATA=( \${NIC$i[@]})	
	echo -e "TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
NAME=${NIC[i]}
DEVICE=${NIC[i]}
ONBOOT=yes
IPADDR=${DATA[0]}
PREFIX=${DATA[1]}
GATEWAY=${DATA[2]}
DNS1=8.8.8.8" > /etc/sysconfig/network-scripts/ifcfg-${NIC[i]}
i=$(expr $i + 1)
done
## Setup OMD
yum install -y wget
cd /opt
wget https://raw.githubusercontent.com/hoangdh/meditech-ghichep-omd/master/scripts/setup-omd-c7.sh
bash /opt/setup-omd-c7.sh
%end

# Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=sda

# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'

# System language
lang en_US.UTF-8

# Network information
# network  --bootproto=static --ip=172.16.2.15 --netmask=255.255.255.0 --device=ens4 --activate
network  --hostname=localhost.localdomain

# Root password
rootpw "meditech2017"

# System services
services --enabled="chronyd"

# System timezone
timezone Asia/Ho_Chi_Minh --isUtc

# System bootloader configuration
bootloader --location=mbr --boot-drive=sda

# Partition clearing information
clearpart --none --initlabel

# Disk partitioning information
part pv.20 --fstype="lvmpv" --ondisk=sda --size=1 --grow
part /boot --fstype="xfs" --ondisk=sda --size=500
volgroup centos --pesize=4096 pv.20
logvol /  --fstype="xfs" --grow --maxsize=51200 --size=1024 --name=root --vgname=centos
logvol swap  --fstype="swap" --size=2048 --name=swap01 --vgname=centos

%packages
#@compat-libraries
@core
#wget
#net-tools
#chrony
%end

reboot
```
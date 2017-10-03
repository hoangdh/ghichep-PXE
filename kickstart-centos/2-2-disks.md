## Cấu hình Kickstart sử dụng LVM với 2 disk

```
# Run the Setup Agent on first boot
firstboot --enable
# ignoredisk --only-use=sda,sdb

# Partition clearing information
clearpart --none --initlabel

# System bootloader configuration
bootloader --location=mbr --boot-drive=sda

# Disk partitioning information
part pv.10 --fstype="lvmpv" --ondisk=sda --size=1 --grow
part pv.20 --fstype="lvmpv" --ondisk=sdb --size=1 --grow
part /boot --fstype="xfs" --ondisk=sda --size=512
volgroup vg-hoangdh --pesize=4096 pv.10 pv.20 
logvol /  --fstype="xfs" --grow  --size=5120 --name=lv-root --vgname=vg-hoangdh
logvol swap  --fstype="swap" --size=2048 --name=lv-swap01 --vgname=vg-hoangdh
```

- Giải thích:
	- Cấu hình boot MBR trên disk `sda`
	- Tạo PV `pv.10` từ `sda` và` pv.20` từ `sdb`
	- Tạo phân vùng /boot với dung lượng 512MB từ `sda`
	- Tạo VG `vg-hoangdh` từ 2 PV `pv.10` và `pv.20`
	- Tạo `lv-swap01` cho SWAP 2GB
	- Tạo `lv-root` cho `/` với dung lượng còn lại
	
### Ghi chép về ổ cứng

- IDE controllers device: – hda/hdb/hdc…
- SCSI controllers device: – sda/sdb/sdc…

### Pre-script tự động detech ổ cứng (Không sử dụng cho SCSI)

```
!/bin/bash
#DISKs=$(ls /sys/block/*/device/* | grep block | grep -Ev "sr0|^$" | cut -d "/" -f4 | sort -u)

DISKs=$(lsblk -D | awk {'print $1'} | grep -Ev "[0-9]$|-|^$|^NAME")
DATA=($DISKs)

echo -e "part /boot --fstype=\"xfs\" --ondisk=${DATA[0]} --size=512
bootloader --location=mbr --boot-drive=${DATA[0]}" > /tmp/disk-include
i=$(echo $DISKs | wc -w)
VG=""
if [ $i = 1 ]
then
	echo -e "part pv.$DISKs --fstype=\"lvmpv\" --ondisk=$DISKs --size=1 --grow" >> /tmp/disk-include
	VG="pv.$DISKs"
else
	
	for d in $DISKs
	do
		echo -e "part pv.$d --fstype=\"lvmpv\" --ondisk=$d --size=1 --grow" >> /tmp/disk-include
		VG=$(echo $VG pv.$d)
	done
fi

echo -e "volgroup vg-hoangdh --pesize=4096 $VG
logvol /  --fstype=\"xfs\" --grow --size=1 --name=lv-root --vgname=vg-hoangdh
logvol /home --fstype=\"ext4\" --percent=10 --name=lv-home --vgname=vg-hoangdh
logvol swap  --fstype=\"swap\" --size=2048 --name=lv-swap01 --vgname=vg-hoangdh" >> /tmp/disk-include
```
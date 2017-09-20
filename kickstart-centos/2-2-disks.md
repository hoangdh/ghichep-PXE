## Cấu hình Kickstart sử dụng LVM với 2 disk

```
# Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=sda,sdb

# System bootloader configuration
bootloader --location=mbr --boot-drive=sda

# Partition clearing information
clearpart --none --initlabel

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
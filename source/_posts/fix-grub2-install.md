title: fix grub2 install
date: 2016-10-19 15:57:49
tags:
---

手里一台机器突然死机，重启后grub失灵了，于是费了一番周折恢复grub2。
先用光盘进入系统，然后打开terminal。

```
sudo su -
mkdir /mnt
mount /dev/sda4 /mnt
mount /dev/sda2 /mnt/boot
mount -t proc proc /mnt/proc
mount --bind /sys /mnt/sys
mount --bind /dev /mnt/dev
chroot /mnt

```
进入后，首先安装grub2时发现域名不能解析，设置了一下 /etc/resolv.conf，比照外面环境设置即可。

```
echo "nameserver 127.0.1.1" > /dev/resolv.conf
```

然后执行

```
apt install grub2
parted /dev/sda set 1 bios_grub on
grub-install --target=i386-pc /dev/sda
```

执行正常没有报错。重启后进入了久违的grub界面。但是又有了新问题，进不去系统了。卡在了systemd的dev-disk服务。
systemd的该服务超时，进入维护模式，后来想到是不是/etc/fstab由于分区的修改导致uuid发生了变化，所以修改了/etc/fstab，不使用uuid了，直接用/dev/sdaX来标示分区。
重启，解决问题！回到了我的系统。


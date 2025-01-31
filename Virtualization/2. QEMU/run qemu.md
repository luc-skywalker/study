apt install libepoxy-dev libglib2.0-dev libpixman-1-dev ninja-build libaio-dev

./configure --enable-kvm --disable-xen --disable-strip --disable-sdl --enable-vhost-net --disable-debug-tcg --target-list=x86_64-softmmu

# 简单直接启动

> sudo /home/sdp/workspace/codes/qemu/build/x86_64-softmmu/qemu-system-x86_64 -name ubuntu -accel kvm -cpu host,host-cache-info=true -m 16G -smp 8 -hda /home/sdp/workspace/images/ubuntu-22.04.raw -netdev user,id=hostnet0 -device rtl8139,netdev=hostnet0,id=net0,mac=52:54:00:36:32:aa,bus=pci.0,addr=0x5 -nographic -full-screen

# 自定义kernel, 且作为IDE设备

-kernel /home/sdp/workspace/images/vmlinuz-5.18.0 -append "root=/dev/sda1 ro console=tty1 console=ttyS0 intel_iommu=sm_on,on" 

将 file 镜像文件作为客户机中的**第 1 个 IDE 设备**(序号0), 而在**客户机**中表现为:

* `/dev/hda` 设备, 若客户机中使用 `PIIX_IDE` 驱动, IDE硬盘;
* `/dev/sda` 设备, 若客户机中使用 `ata_piix` 驱动, SATA硬盘(目前 kernel 使用的).

## 镜像文件不用任何参数

可以不加 `-hda`, `/home/sdp/workspace/images/ubuntu-22.04.raw`

> sudo /home/sdp/workspace/codes/qemu/build/x86_64-softmmu/qemu-system-x86_64 -name ubuntu -accel kvm -cpu host,host-cache-info=true -m 16G -smp 8 /home/sdp/workspace/images/ubuntu-22.04.raw -netdev user,id=hostnet0 -device rtl8139,netdev=hostnet0,id=net0,mac=52:54:00:36:32:aa,bus=pci.0,addr=0x5 -kernel /home/sdp/workspace/images/vmlinuz-5.18.0 -append "root=/dev/sda1 ro console=tty1 console=ttyS0 intel_iommu=sm_on,on" -nographic -full-screen

## 指定hda参数

`-hda /home/sdp/workspace/images/ubuntu-22.04.raw`

> sudo /home/sdp/workspace/codes/qemu/build/x86_64-softmmu/qemu-system-x86_64 -name ubuntu -accel kvm -cpu host,host-cache-info=true -m 16G -smp 8 -hda /home/sdp/workspace/images/ubuntu-22.04.raw -netdev user,id=hostnet0 -device rtl8139,netdev=hostnet0,id=net0,mac=52:54:00:36:32:aa,bus=pci.0,addr=0x5 -kernel /home/sdp/workspace/images/vmlinuz-5.18.0 -append "root=/dev/sda1 ro console=tty1 console=ttyS0 intel_iommu=sm_on,on" -nographic -full-screen

## 使用drive参数

`-drive file=/home/sdp/workspace/images/ubuntu-22.04.raw,format=raw,if=ide`

> sudo /home/sdp/workspace/codes/qemu/build/x86_64-softmmu/qemu-system-x86_64 -name ubuntu -accel kvm -cpu host,host-cache-info=true -m 16G -smp 8 -drive file=/home/sdp/workspace/images/ubuntu-22.04.raw,format=raw,if=ide -netdev user,id=hostnet0 -device rtl8139,netdev=hostnet0,id=net0,mac=52:54:00:36:32:aa,bus=pci.0,addr=0x5 -kernel /home/sdp/workspace/images/vmlinuz-5.18.0 -append "root=/dev/sda1 ro console=tty1 console=ttyS0 intel_iommu=sm_on,on" -nographic -full-screen

# 作为virtio-blk设备

```
CONFIG_VIRTIO=y
CONFIG_VIRTIO_PCI=y
CONFIG_VIRTIO_BLK=y
```

guest 中就是 /dev/vda 设备

`-drive file=/home/sdp/workspace/images/ubuntu-22.04.raw,format=raw,if=none,id=drive-virtio-disk0,cache=none,aio=native -object iothread,id=iothread0 -device virtio-blk-pci,iothread=iothread0,scsi=off,drive=drive-virtio-disk0,id=virtio-disk0,bootindex=1` 和 `root=/dev/vda1`

> sudo /home/sdp/workspace/codes/qemu/build/x86_64-softmmu/qemu-system-x86_64 -name ubuntu -accel kvm -cpu host,host-cache-info=true -m 16G -smp 8 -drive file=/home/sdp/workspace/images/ubuntu-22.04.raw,format=raw,if=none,id=drive-virtio-disk0,cache=none,aio=native -object iothread,id=iothread0 -device virtio-blk-pci,iothread=iothread0,scsi=off,drive=drive-virtio-disk0,id=virtio-disk0,bootindex=1 -netdev user,id=hostnet0 -device rtl8139,netdev=hostnet0,id=net0,mac=52:54:00:36:32:aa,bus=pci.0,addr=0x5 -kernel /home/sdp/workspace/images/vmlinuz-5.18.0 -append "root=/dev/vda1 ro console=tty1 console=ttyS0 intel_iommu=sm_on,on" -nographic -full-screen
> 
> 


# monitor

加上下面的就可以启用 monitor

```
-chardev socket,id=montest,server=on,wait=off,path=/tmp/mon_test -mon chardev=montest,mode=readline
```

但是 `Ctrl+C` 会退出 vm, 信息如下

```
qemu-system-x86_64: terminating on signal 2
```

根据 https://stackoverflow.com/questions/49716931/how-to-run-qemu-with-nographic-and-monitor-but-still-be-able-to-send-ctrlc-to, 添加下面参数就可以解决

```
-serial mon:stdio
```

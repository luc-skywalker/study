
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. 网络传输](#1-网络传输)
- [2. 共享文件](#2-共享文件)
  - [2.1. 环境配置](#21-环境配置)
  - [2.2. 在host上创建共享文件夹](#22-在host上创建共享文件夹)
  - [2.3. 启动虚拟机](#23-启动虚拟机)
  - [2.4. Guest上mount共享文件夹](#24-guest上mount共享文件夹)
- [3. qemu-nbd方式](#3-qemu-nbd方式)
- [4.](#4)
- [5. virt工具](#5-virt工具)
- [6. 参考](#6-参考)

<!-- /code_chunk_output -->

# 1. 网络传输

虚拟机与宿主机之间, 使用网络来进行文件传输. 这个需要先在宿主机上配置网络桥架, 在qemu-kvm启动配置网卡就可以实现文件传输. 

# 2. 共享文件

方式是qemu实现的**Plan 9 folder sharing over Virtio**

## 2.1. 环境配置

要宿主机需要在内核中配置了9p选项, 即: 

```conf
CONFIG_NET_9P=y
CONFIG_net_9P_VIRTIO=y
CONFIG_NET_9P_DEBUG=y (可选项)
CONFIG_9P_FS=y
CONFIG_9P_FS_POSIX_ACL=y
```

另外, qemu在编译时需要支持ATTR/XATTR. 

## 2.2. 在host上创建共享文件夹

在Host上建立一个共享文件夹: 

```
# mkdir /tmp/shared_host
```

## 2.3. 启动虚拟机

在Host上启动虚拟机qemu: 注意最后的mount_tag

```
# qemu-system-x86_64 -smp 2 -m 4096 -enable-kvm -drive file=/home/test/ubuntu.img,if=virtio -vnc :2 \
-fsdev local,security_model=passthrough,id=fsdev-fs0,path=/tmp/shared_host -device virtio-9p-pci,id=fs0,fsdev=fsdev-fs0,mount_tag=test_mount
```

在Host上启动虚拟机libvirt: 

```xml
<devices>
    <filesystem type='mount' accessmode='passthrough'>
      <source dir='/tmp/shared_host'/>
      <target dir='test_mount'/>
    </filesystem>
</devices>
```


```
<devices>
    <filesystem type='mount' accessmode='passthrough'>
      <source dir='/data/shared'/>
      <target dir='test_mount'/>
    </filesystem>
</devices>
```

## 2.4. Guest上mount共享文件夹

在Guest上mount共享文件夹: 

```
# mkdir /tmp/shared_guest
# mount -t 9p -o trans=virtio test_mount /tmp/shared_guest/ -oversion=9p2000.L,posixacl,cache=loose
```

```
mkdir /root/shared
mount -t 9p -o trans=virtio test_mount /root/shared/ -oversion=9p2000.L,posixacl,cache=loose
```

现在就可在Host的/tmp/shared_host和Guest的/tmp/shared_guest/之间进行文件的共享了. 

# 3. qemu-nbd方式

查看\<qemu-nbd方式挂载镜像>

# 4. 

使用dd创建一个文件, 作为虚拟机和宿主机之间传输桥梁

```
dd if=/dev/zero of=/opt/share.img bs=1M count=200
```

格式化share.img文件

```
mkfs.ext4 /opt/share.img
```

在宿主机上创建一个文件夹, 
   
```
mkdir /tmp/share
mount -o loop /opt/share.img /tmp/share
```

这样, 在宿主机上把需要传输给虚拟机的文件放到/tmp/share 下即可. 

启动qemu-kvm虚拟机, 添加上/opt/share.img文件. 

在虚拟机中 mount上添加的一块硬盘. 即可以获得宿主机上放在/tmp/share文件夹下的文件

该方法的缺点: 
     
宿主机和虚拟机文件传输不能实时传输. 如果需要传输新文件, 需要重启虚拟机. 


# 5. virt工具

virt-copy-in/out(libguestfs-tools包)

参考: http://libguestfs.org/virt-copy-in.1.html

# 6. 参考

https://sjt157.github.io/2018/12/12/VM%E4%B8%8EHOST%E4%B9%8B%E9%97%B4%E4%BC%A0%E8%BE%93%E6%96%87%E4%BB%B6/


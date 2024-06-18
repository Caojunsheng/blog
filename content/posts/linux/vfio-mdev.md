---
title: "vfio-mdev使用"
date: 2024-06-18T16:59:28+08:00
description: "Virtio PCI配置空间详解"
draft: false
tags: ["virtio"]
categories: ["linux"]
series: ["linux"]
---

使用vfio-mdev

  

modprobe vfio_pci

  

cd /home/cjs/usr/src/linux-5.10.0-60.18.0.50.r1064_55.hce2.x86_64/

make modules

  

make M=/home/cjs/usr/src/linux-5.10.0-60.18.0.50.r1064_55.hce2.x86_64/drivers/vfio/ -C /home/cjs/usr/src/linux-5.10.0-60.18.0.50.r1064_55.hce2.x86_64 -j10 CONFIG_VFIO_MDEV=m

insmod mdev.ko

insmod vfio_mdev.ko

  

make M=/home/cjs/usr/src/linux-5.10.0-60.18.0.50.r1064_55.hce2.x86_64/samples/vfio-mdev/ -C /home/cjs/usr/src/linux-5.10.0-60.18.0.50.r1064_55.hce2.x86_64 -j10 CONFIG_SAMPLE_VFIO_MDEV_MTTY=m

  
  

echo "83b8f4f2-509f-382f-3c1e-e6bfe0fa1001" > /sys/devices/virtual/mtty/mtty/mdev_supported_types/mtty-2/create

  

qemu-kvm -machine q35,accel=kvm -cpu host -smp 8 -m 16G -drive if=none,id=root,file=./centos7.2_cn.qcow2_par -device virtio-blk-pci,drive=root,disable-legacy=on -vga std -vnc :66 -device vfio-pci,addr=05.0,sysfsdev=/sys/bus/mdev/devices/83b8f4f2-509f-382f-3c1e-e6bfe0fa1001 -daemonize
> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY2NjI1NjM2Ml19
-->
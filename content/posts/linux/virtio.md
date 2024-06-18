---
title: "virtio_net设备初始化流程"
date: 2024-06-18T16:29:28+08:00
description: "virtio_net设备初始化"
draft: false
tags: ["virtio"]
categories: ["linux"]
series: ["linux"]
---

worker_thread --> process_one_work --> pciehp_power_thread -->

`pciehp_ctrl.c`

pciehp_enable_slot --> board_added -->

`pciehp_pci.c`

pciehp_configure_device -->

`pci/bus.c`

pci_bus_add_devices --> pci_bus_add_device -->

`dd.c`

device_attach --> __device_attach --> bus_for_each_drv --> __device_attach_driver --> driver_probe_device --> really_probe -->call_driver_probe -->

`pci-driver.c`

pci_device_probe --> pci_call_probe --> local_pci_probe -->

`virtio_pci_common.c`

virtio_pci_probe -->

`virtio.c`

register_virtio_device -->

`core.c`

device_add -->

`base/bus.c`

bus_probe_device -->

`dd.c`

device_initial_probe --> __device_attach --> bus_for_each_drv --> __device_attach_driver --> driver_probe_device --> really_probe -->call_driver_probe -->

`virtio.c`

virtio_dev_probe -->

`virtio_net.c`

virtnet_probe (网卡设备驱动加载的入口)

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM4NzA3MjQyMV19
-->
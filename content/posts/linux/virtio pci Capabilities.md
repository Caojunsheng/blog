---
title: "Virtio PCI设备配置空间详解"
date: 2024-05-29T15:59:28+08:00
description: "Virtio PCI配置空间详解"
draft: false
tags: ["PCIe", "virtio"]
categories: ["linux"]
series: ["linux"]
---
```shell

```
```c
// PCI Capabilities ID枚举
#define  PCI_CAP_ID_PM		0x01	/* Power Management */
#define  PCI_CAP_ID_AGP		0x02	/* Accelerated Graphics Port */
#define  PCI_CAP_ID_VPD		0x03	/* Vital Product Data */
#define  PCI_CAP_ID_SLOTID	0x04	/* Slot Identification */
#define  PCI_CAP_ID_MSI		0x05	/* Message Signalled Interrupts */
#define  PCI_CAP_ID_CHSWP	0x06	/* CompactPCI HotSwap */
#define  PCI_CAP_ID_PCIX	0x07	/* PCI-X */
#define  PCI_CAP_ID_HT		0x08	/* HyperTransport */
#define  PCI_CAP_ID_VNDR	0x09	/* Vendor-Specific */
#define  PCI_CAP_ID_DBG		0x0A	/* Debug port */
#define  PCI_CAP_ID_CCRC	0x0B	/* CompactPCI Central Resource Control */
#define  PCI_CAP_ID_SHPC	0x0C	/* PCI Standard Hot-Plug Controller */
#define  PCI_CAP_ID_SSVID	0x0D	/* Bridge subsystem vendor/device ID */
#define  PCI_CAP_ID_AGP3	0x0E	/* AGP Target PCI-PCI bridge */
#define  PCI_CAP_ID_SECDEV	0x0F	/* Secure Device */
#define  PCI_CAP_ID_EXP		0x10	/* PCI Express */
#define  PCI_CAP_ID_MSIX	0x11	/* MSI-X */
#define  PCI_CAP_ID_SATA	0x12	/* SATA Data/Index Conf. */
#define  PCI_CAP_ID_AF		0x13	/* PCI Advanced Features */
#define  PCI_CAP_ID_EA		0x14	/* PCI Enhanced Allocation */

// virtio_pci_cap结构体定义
struct virtio_pci_cap {
	__u8 cap_vndr;		/* Generic PCI field: PCI_CAP_ID_VNDR */
	__u8 cap_next;		/* Generic PCI field: next ptr. */
	__u8 cap_len;		/* Generic PCI field: capability length */
	__u8 cfg_type;		/* Identifies the structure. */
	__u8 bar;		/* Where to find it. */
	__u8 id;		/* Multiple capabilities of the same type */
	__u8 padding[2];	/* Pad to full dword. */
	__le32 offset;		/* Offset within bar. */
	__le32 length;		/* Length of the structure, in bytes. */
};

// cfg_type枚举类型
/* Common configuration */
#define VIRTIO_PCI_CAP_COMMON_CFG	1
/* Notifications */
#define VIRTIO_PCI_CAP_NOTIFY_CFG	2
/* ISR access */
#define VIRTIO_PCI_CAP_ISR_CFG		3
/* Device specific configuration */
#define VIRTIO_PCI_CAP_DEVICE_CFG	4
/* PCI configuration access */
#define VIRTIO_PCI_CAP_PCI_CFG		5
/* Additional shared memory capability */
#define VIRTIO_PCI_CAP_SHARED_MEMORY_CFG 8
```

![](http://image.huawei.com/tiny-lts/v1/images/0a60c0ca41e348844beda2c20ec2d279_486x326.png)


以c8起始的Capabilities为例，逐个解析
Capabilities: [c8] Vendor Specific Information: VirtIO: Notify
		BAR=1 offset=00000ff0 size=00000004 multiplier=00000000

c0: 00 0f 00 00 38 00 00 00 <span style="background-color:rgb(233,30,77)">09 dc 14 02 01 00 00 00</span>
d0: <span style="background-color:rgb(233,30,77)">f0 0f 00 00 04 00 00 00</span> 00 00 00 00 09 ec 10 03

| 配置空间值  | 对应字段        | 实际含义                         |
|-------------|-----------------|----------------------------------|
| 09          | __u8 cap_vndr   | PCI_CAP_ID_VNDR：Vendor-Specific |
| dc          | __u8 cap_next   | 指向下一个指针起始位置           |
| 14          | __u8 cap_len    | Capabilities长度                 |
| 02          | __u8 cfg_type   | VIRTIO_PCI_CAP_NOTIFY_CFG        |
| 01          | __u8 bar        | BAR=1                            |
| 00          | __u8 id         | 相同类型的Capabilities的id标识   |
| 00 00       | __u8 padding[2] |                                  |
| f0 0f 00 00 | __le32 offset   | 偏移位置                         |
| 04 00 00 00 | __le32 length   | 大小                             |

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM5MTg2MDE5NF19
-->
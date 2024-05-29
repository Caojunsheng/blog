---
title: "Virtio PCI设备配置空间详解"
date: 2024-05-29T15:59:28+08:00
description: "Virtio PCI配置空间详解"
draft: false
tags: ["PCIe", "virtio"]
categories: ["linux"]
series: ["linux"]
---
以virtio_blk设备的配置空间为例
```shell
# lspci -vvvvs 31:00.7
31:00.7 Class fe01: Virtio: Virtio block device (prog-if 30)
	Subsystem: Virtio: Device 0002
	Physical Slot: 2
	Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr+ Stepping- SERR+ FastB2B- DisINTx+
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx+
	Latency: 0, Cache Line Size: 32 bytes
	NUMA node: 0
	Region 0: Memory at 9c816000 (32-bit, prefetchable) [size=8K]
	Region 1: Memory at 9c821000 (32-bit, non-prefetchable) [size=4K]
	Region 2: Memory at 9c7e8000 (32-bit, prefetchable) [size=32K]
	Region 3: Memory at 9c730000 (32-bit, non-prefetchable) [size=64K]
	Region 4: Memory at d2ff4800000 (64-bit, prefetchable) [size=4M]
	Expansion ROM at 9bf00000 [disabled] [size=1M]
	Capabilities: [40] Express (v2) Endpoint, MSI 00
		DevCap:	MaxPayload 512 bytes, PhantFunc 0, Latency L0s unlimited, L1 unlimited
			ExtTag+ AttnBtn- AttnInd- PwrInd- RBE+ FLReset+ SlotPowerLimit 0.000W
		DevCtl:	CorrErr+ NonFatalErr+ FatalErr+ UnsupReq-
			RlxdOrd+ ExtTag+ PhantFunc- AuxPwr- NoSnoop+ FLReset-
			MaxPayload 512 bytes, MaxReadReq 4096 bytes
		DevSta:	CorrErr+ NonFatalErr- FatalErr- UnsupReq+ AuxPwr- TransPend-
		LnkCap:	Port #0, Speed 16GT/s, Width x8, ASPM not supported
			ClockPM- Surprise- LLActRep- BwNot- ASPMOptComp+
		LnkCtl:	ASPM Disabled; RCB 64 bytes Disabled- CommClk-
			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
		LnkSta:	Speed 16GT/s (ok), Width x8 (ok)
			TrErr- Train- SlotClk- DLActive- BWMgmt- ABWMgmt-
		DevCap2: Completion Timeout: Not Supported, TimeoutDis-, NROPrPrP-, LTR-
			 10BitTagComp-, 10BitTagReq-, OBFF Not Supported, ExtFmt+, EETLPPrefix-
			 EmergencyPowerReduction Not Supported, EmergencyPowerReductionInit-
			 FRS-, TPHComp-, ExtTPHComp-
			 AtomicOpsCap: 32bit- 64bit- 128bitCAS-
		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis-, LTR-, OBFF Disabled
			 AtomicOpsCtl: ReqEn+
		LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete-, EqualizationPhase1-
			 EqualizationPhase2-, EqualizationPhase3-, LinkEqualizationRequest-
	Capabilities: [80] MSI: Enable- Count=1/2 Maskable+ 64bit+
		Address: 0000000000000000  Data: 0000
		Masking: 00000000  Pending: 00000000
	Capabilities: [98] Vital Product Data
		Product Name:  
		Read-only fields:
			[PN] Part number: 970, NIC, 2X100GE
		End
	Capabilities: [a0] MSI-X: Enable+ Count=2 Masked-
		Vector table: BAR=2 offset=00000000
		PBA: BAR=2 offset=00004000
	Capabilities: [b0] Power Management version 3
		Flags: PMEClk- DSI- D1- D2- AuxCurrent=0mA PME(D0+,D1+,D2+,D3hot+,D3cold+)
		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
	Capabilities: [b8] Vendor Specific Information: VirtIO: CommonCfg
		BAR=1 offset=00000f00 size=00000038
	Capabilities: [c8] Vendor Specific Information: VirtIO: Notify
		BAR=1 offset=00000ff0 size=00000004 multiplier=00000000
	Capabilities: [dc] Vendor Specific Information: VirtIO: ISR
		BAR=1 offset=00000f3c size=00000004
	Capabilities: [ec] Vendor Specific Information: VirtIO: DeviceCfg
		BAR=1 offset=00000f40 size=00000050
	Capabilities: [100 v2] Advanced Error Reporting
		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
		UEMsk:	DLP- SDES- TLP+ FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
		AERCap:	First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap+
		HeaderLog: 00000000 00000000 00000000 00000000
	Capabilities: [150 v1] Alternative Routing-ID Interpretation (ARI)
		ARICap:	MFVC- ACS-, Next Function: 8
		ARICtl:	MFVC- ACS-, Function Group: 0
	Capabilities: [200 v1] Single Root I/O Virtualization (SR-IOV)
		IOVCap:	Migration-, Interrupt Message Number: 000
		IOVCtl:	Enable+ Migration- Interrupt- MSE+ ARIHierarchy-
		IOVSta:	Migration-
		Initial VFs: 255, Total VFs: 255, Number of VFs: 255, Function Dependency Link: 07
		VF offset: 1144, stride: 1, Device ID: 1001
		Supported Page Size: 00000553, System Page Size: 00000001
		Region 0: Memory at 00000d2ffd70c000 (64-bit, prefetchable)
		Region 2: Memory at 00000d2ff97f0000 (64-bit, prefetchable)
		Region 4: Memory at 00000d2ffd310000 (64-bit, prefetchable)
		VF Migration: offset: 00000000, BIR: 0
	Capabilities: [2a0 v1] Transaction Processing Hints
		Device specific mode supported
		No steering table available
	Capabilities: [4e0 v1] Device Serial Number ff-ff-ff-ff-ff-ff-ff-ff
	Capabilities: [630 v1] Access Control Services
		ACSCap:	SrcValid- TransBlk- ReqRedir- CmpltRedir- UpstreamFwd- EgressCtrl- DirectTrans-
		ACSCtl:	SrcValid- TransBlk- ReqRedir- CmpltRedir- UpstreamFwd- EgressCtrl- DirectTrans-
	Capabilities: [700 v1] Data Link Feature <?>
	Kernel driver in use: virtio-pci
	Kernel modules: virtio_pci



```
lspci -xxxs 31:00.7
31:00.7 Class fe01: Virtio: Virtio block device
00: f4 1a 01 10 46 05 18 00 00 30 01 fe 08 00 80 00
10: 08 60 81 9c 00 10 82 9c 08 80 7e 9c 00 00 73 9c
20: 0c 00 80 f4 2f 0d 00 00 00 00 00 00 f4 1a 02 00
30: 00 00 f0 9b 40 00 00 00 00 00 00 00 ff 00 00 00
40: 10 80 02 00 e2 8f 00 10 57 59 09 00 84 f0 43 00
50: 00 00 84 00 00 00 00 00 00 00 00 00 00 00 00 00
60: 00 00 00 00 00 00 10 00 40 00 00 00 3e 3e 80 01
70: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
80: 05 98 82 01 00 00 00 00 00 00 00 00 00 00 00 00
90: 00 00 00 00 00 00 00 00 03 a0 18 80 30 47 45 78
a0: 11 b0 01 80 02 00 00 00 02 40 00 00 00 00 00 00
b0: 01 b8 03 f8 08 00 00 00 09 c8 10 01 01 00 00 00
c0: 00 0f 00 00 38 00 00 00 09 dc 14 02 01 00 00 00
d0: f0 0f 00 00 04 00 00 00 00 00 00 00 09 ec 10 03
e0: 01 00 00 00 3c 0f 00 00 04 00 00 00 09 00 10 04
f0: 01 00 00 00 40 0f 00 00 50 00 00 00 00 00 00 00

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
eyJoaXN0b3J5IjpbMTE1NjAyMTU0Nl19
-->
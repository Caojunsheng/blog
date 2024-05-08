---
title: "PCIe AER寄存器值查看和修改"
date: 2024-05-08T15:59:28+08:00
description: "如何查看PCIe AER寄存器值和修改寄存器值"
draft: false
tags: ["PCIe"]
categories: ["linux"]
series: ["Hugo Guide"]
---

https://blog.csdn.net/zsmcdut/article/details/120151896
```shell
Capabilities: [100 v2] Advanced Error Reporting
        UESta:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
        UEMsk:  DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
        UESvrt: DLP+ SDES+ TLP+ FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
        CESta:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
        CEMsk:  RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
        AERCap: First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
            MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap+
        HeaderLog: 00000000 00000000 00000000 00000000
```

 - UESta: Uncorrectable Error Status，表示不可纠正错误状态，用于指示发生不可纠正错误的状态。
 - UEMsk: Uncorrectable Error Mask，表示不可纠正错误掩码，用于掩盖或屏蔽不可纠正错误。
 - UESvrt: Uncorrectable Error Severity，表示不可纠正错误严重性，用于指示不可纠正错误的严重程度。
 - CESta: Correctable Error Status，表示可纠正错误状态，用于指示发生可纠正错误的状态。
 - CEMsk: Correctable Error Mask，表示可纠正错误掩码，用于掩盖或屏蔽可纠正错误。 
Uncorrectable Error

DLP: Data Link Protocol Error，数据链路协议错误，表示数据链路协议错误。
SDES: Surprise Down Error Status，意外下降错误状态，表示设备在意外下降时的错误状态。
TLP: Transaction Layer Packet，事务层包，表示事务层包错误。
FCP: Flow Control Protocol Error，流控制协议错误，表示流控制协议错误。
CmpltTO: Completion Timeout，完成超时，表示完成操作超时。
CmpltAbrt: Completion Abort，完成中止，表示完成操作中止。
UnxCmplt: Unsupported Completion，不支持的完成，表示不支持的完成操作。
RxOF: Receiver Overflow，接收器溢出，表示接收器溢出错误。
MalfTLP: Malformed TLP，格式错误的TLP，表示格式错误的事务层包。
ECRC: ECRC Error，ECRC 错误，表示ECRC校验错误。
UnsupReq: Unsupported Request，不支持的请求，表示设备不支持的请求。
ACSViol: ACS Violation，ACS 违规，表示ACS规则违反。
Correctable Error

RxErr: Receive Error，表示接收数据时发生错误。
BadTLP: Bad TLP，表示传输层包（TLP）格式错误或无效。
BadDLLP: Bad DLLP，表示数据链路层包（DLLP）格式错误或无效。
Rollover: 溢出，表示发生了溢出错误。
Timeout: 超时，表示操作超时。
AdvNonFatalErr: 高级非致命错误，表示发生了高级非致命错误。
AERCap

ECRCGenCap 和 ECRCGenEn 表示设备是否支持和启用 ECRC 生成（ECRC generation）功能。
ECRCChkCap 和 ECRCChkEn 表示设备是否支持和启用 ECRC 检查（ECRC checking）功能。
MultHdrRecCap 和 MultHdrRecEn 表示设备是否支持和启用多头记录（Multiple Header Record）功能。
TLPPfxPres 表示设备是否支持 TLP 前缀保留（TLP Prefix Preservation）功能。
HdrLogCap 表示设备是否支持头部日志（Header Logging）功能。

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbODY3ODM1Nzk2LC0yMTE5MDA1NDkzXX0=
-->
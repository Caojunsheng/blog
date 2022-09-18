---
title: "golang chan recv源码分析"
date: 2021-09-17T17:11:28+08:00
description: "golang chan recv源码分析"
draft: false
tags: ["golang", "chan", "源码阅读"]
categories: ["golang chan"]
series: ["golang"]
---

### 2、chan读取源码分析
chan的读取源码入口是如下两个函数：
```go
// 读取的数据放在elem里面，两种读取的方式，第一种直接返回值，第二种返回一个bool值，判断chan是否关闭 
func chanrecv1(c *hchan, elem unsafe.Pointer) {  
   chanrecv(c, elem, true)  
}  

func chanrecv2(c *hchan, elem unsafe.Pointer) (received bool) {  
   _, received = chanrecv(c, elem, true)  
   return  
}
```
`chanrecv1`不返回ok，`chanrecv2`返回ok，两个最终都是调用`chanrecv`函数
```go
// src/runtime/chan.go:454
// chanrecv 函数接收 channel c 的元素并将其写入 ep 所指向的内存地址。
// 如果 ep 是 nil，说明忽略了接收值。
// 如果 block == false，即非阻塞型接收，在没有数据可接收的情况下，返回 (false, false)
// 否则，如果 c 处于关闭状态，将 ep 指向的地址清零，返回 (true, false)
// 否则，用返回值填充 ep 指向的内存地址。返回 (true, true)
// 如果 ep 非空，则应该指向堆或者函数调用者的栈
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	...
    // 如果chan是nil的话
	if c == nil {
	    // 非阻塞调用，则直接返回false, false 
		if !block {
			return
		}
		// 阻塞调用，一直等待接收nil的chan，goroutine挂起
		gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
		throw("unreachable")
	}

	// 如果是非阻塞且chan是空的
	if !block && empty(c) {
	    // 如果chan是未关闭的，直接返回false,false
		if atomic.Load(&c.closed) == 0 {
			return
		}
		// chan已经关闭，并且为空，老实说。这段代码感觉有点多余，下面也处理了这种情况
		if empty(c) {
			// The channel is irreversibly closed and empty.
			if raceenabled {
				raceacquire(c.raceaddr())
			}
			if ep != nil {
			    // 对于已关闭的chan执行接收，不忽略返回值的情况下，会受到该类型的零值，清理ep的内存
				typedmemclr(c.elemtype, ep)
			}
			// 返回selected为true
			return true, false
		}
	}

	var t0 int64
	if blockprofilerate > 0 {
		t0 = cputicks()
	}

	lock(&c.lock)
    // chan已经关闭，且缓存中无数据，直接返回该类型的零值
	if c.closed != 0 && c.qcount == 0 {
		if raceenabled {
			raceacquire(c.raceaddr())
		}
		unlock(&c.lock)
		if ep != nil {
			typedmemclr(c.elemtype, ep)
		}
		return true, false
	}

	if sg := c.sendq.dequeue(); sg != nil {
		// 如果sender中有等待发送，那么可以分为两种情况
		// 1、非缓冲队列，即同步chan，则直接从sender中接收值。
		// 2、缓冲队列，即异步chan，从缓冲队列的头部拷贝到接收者，拷贝发送队列的值到缓冲队列末尾
		recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
		return true, true
	}

    // 缓冲型chan，buf里面有元素，直接从buf里面拿
	if c.qcount > 0 {
		// Receive directly from queue
		qp := chanbuf(c, c.recvx)
		if raceenabled {
			racenotify(c, c.recvx, nil)
		}
		// 代码里面需要接收值，则需要拷贝值，比如接收是`val<-ch`，而不是`<-ch`，需要把chan的值拷贝到val
		if ep != nil {
			typedmemmove(c.elemtype, ep, qp)
		}
		// 清空掉原来buf中对应位置的值
		typedmemclr(c.elemtype, qp)
		// 接收index+1
		c.recvx++
		// 如果接收索引已经到末尾，重新移到队首
		if c.recvx == c.dataqsiz {
			c.recvx = 0
		}
		// 缓冲区大小减一
		c.qcount--
		// 解锁
		unlock(&c.lock)
		return true, true
	}

	if !block {
	    // 非阻塞接收，解锁，返回false,false
		unlock(&c.lock)
		return false, false
	}

	// 无发送者，这个接收值需要被阻塞.
	gp := getg()
	mysg := acquireSudog()
	mysg.releasetime = 0
	if t0 != 0 {
		mysg.releasetime = -1
	}
	// 构造一个接收数据的sudog.
	mysg.elem = ep
	mysg.waitlink = nil
	gp.waiting = mysg
	mysg.g = gp
	mysg.isSelect = false
	mysg.c = c
	gp.param = nil
	// 放入接受者队列中
	c.recvq.enqueue(mysg)
	// 将goroutine挂起
	atomic.Store8(&gp.parkingOnChan, 1)
	gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanReceive, traceEvGoBlockRecv, 2)

	// someone woke us up
	if mysg != gp.waiting {
		throw("G waiting list is corrupted")
	}
	gp.waiting = nil
	gp.activeStackChans = false
	if mysg.releasetime > 0 {
		blockevent(mysg.releasetime-t0, 2)
	}
	success := mysg.success
	gp.param = nil
	mysg.c = nil
	releaseSudog(mysg)
	return true, success
}

```
empty源码分析
 1. 如果是非缓冲型，且sendq中无goroutine

 2. 缓冲型，但是buf里面没有元素

```go
func empty(c *hchan) bool {
	// c.dataqsiz is immutable.
	if c.dataqsiz == 0 {
		return atomic.Loadp(unsafe.Pointer(&c.sendq.first)) == nil
	}
	return atomic.Loaduint(&c.qcount) == 0
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTEzMDUzNDRdfQ==
-->

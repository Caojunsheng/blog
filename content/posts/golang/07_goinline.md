---
title: "golang内联判断"
date: 2023-01-14T15:15:28+08:00
description: "golang如何判断函数是否被内联"
draft: false
tags: ["golang", "inline"]
categories: ["golang"]
series: ["golang"]
---
### 一、什么是函数内联
 内联优化是什么？
内联(inlining)是编程语言编译器常用的优化手段，其优化的对象为函数，也称为函数内联。如果某函数F支持内联，则意味着编译器可以用F的函数体/函数定义替换掉对函数F进行调用的代码，以消除函数调用带来的额外开销，这个过程如下图所示：
![image.png](https://tonybai.com/wp-content/uploads/understand-go-inlining-optimisations-by-example-2.png)
但是内联并不是只有优点而无缺点，因为内联，其实就是将一个函数调用原地展开，替换成这个函数的实现。当该函数被多次调用，就会被多次展开，这会增加编译后二进制文件的大小。而非内联函数，只需要保存一份函数体的代码，然后进行调用。所以，在空间上，一般来说使用内联函数会导致生成的可执行文件变大。

### 如何判断函数是否支持内联
我们先看下面的一段代码，testInline1、testInline2、testInline3是否能够被内联，我们猜想的是testInline1能够被内联，testInline2和testInline3不能够被内联。
```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println(testInline1(1, 2))
	fmt.Println(testInline2(1, 2))
	fmt.Println(testInline3(1, 2))
}

func testInline1(a, b int) int {
	if a > b {
		return a
	}
	return b
}

//go:noinline
func testInline2(a, b int) int {
	if a > b {
		return a
	}
	return b
}

func testInline3(a, b int) int {
	for i := 0; i < 10; i++ {
		fmt.Println(i)
	}
	if a > b {
		return a
	}
	return b
}
```
#### 方法1：使用go tool compile
我们可以使用go tool compile查看函数是否能够被内联：
```bash
go tool compile -m=2 test.go
```
- testInline1符合我们预期，是不能被内联的
- testInline2由于被打上注释“//go:noinline”，所以也不会被内联
- testInline3则是因为函数过于复杂，超过阈值80了，所以也不能被内联
```bash
$ go tool compile -m=2 test.go
test.go:15:6: can inline testInline1 with cost 8 as: func(int, int) int { if a > b { return a }; return b }
test.go:23:6: cannot inline testInline2: marked go:noinline
test.go:30:6: cannot inline testInline3: function too complex: cost 96 exceeds budget 80
test.go:32:14: inlining call to fmt.Println func(...interface {}) (int, error) { var fmt..autotmp_3 int; fmt..autotmp_3 = <nil>; var fmt..autotmp_4 error; fmt..autotmp_4 = <nil>; fmt..autotmp_3, fmt..autotmp_4 = fmt.Fprintln(io.Writer(os.Stdout), fmt.a...); return fmt..autotmp_3, fmt..autotmp_4 }
test.go:9:6: cannot inline main: function too complex: cost 359 exceeds budget 80
test.go:10:25: inlining call to testInline1 func(int, int) int { if a > b { return a }; return b }
test.go:10:13: inlining call to fmt.Println func(...interface {}) (int, error) { var fmt..autotmp_3 int; fmt..autotmp_3 = <nil>; var fmt..autotmp_4 error; fmt..autotmp_4 = <nil>; fmt..autotmp_3, fmt..autotmp_4 = fmt.Fprintln(io.Writer(os.Stdout), fmt.a...); return fmt..autotmp_3, fmt..autotmp_4 }
test.go:11:13: inlining call to fmt.Println func(...interface {}) (int, error) { var fmt..autotmp_3 int; fmt..autotmp_3 = <nil>; var fmt..autotmp_4 error; fmt..autotmp_4 = <nil>; fmt..autotmp_3, fmt..autotmp_4 = fmt.Fprintln(io.Writer(os.Stdout), fmt.a...); return fmt..autotmp_3, fmt..autotmp_4 }
test.go:12:13: inlining call to fmt.Println func(...interface {}) (int, error) { var fmt..autotmp_3 int; fmt..autotmp_3 = <nil>; var fmt..autotmp_4 error; fmt..autotmp_4 = <nil>; fmt..autotmp_3, fmt..autotmp_4 = fmt.Fprintln(io.Writer(os.Stdout), fmt.a...); return fmt..autotmp_3, fmt..autotmp_4 }

```

我们把该方法落到我们实际代码库上再试一下，我们以session库上smc/exec/do/internal/domain/objects/feature/perf/perfrole/perf5g/perf5gconcentrate/perfbuilder_ismf.go文件中`isFailedByRejUnspecidiedRecvByIsmf`为例。

![image.png](http://image.huawei.com/tiny-lts/v1/images/e3c38b9c091eac0bf3326f95dc03ae37_1119x212.png@900-0-90-f.png)
我们到perf5gconcentrate目录下，执行go tool compile命令，发现有导入包的报错，go tool compile仅支持单个文件，所以对于我们代码库上函数，建议使用方法2
![image.png](http://image.huawei.com/tiny-lts/v1/images/396adce310b9777dbb56f68a28ae9e10_1655x114.png@900-0-90-f.png)

#### 方法2：使用go build
使用go build编译一个目录perf5gconcentrate查询函数是否被内联
执行如下命令：
```bash
go build -gcflags="-m=2" .
```
![image.png](http://image.huawei.com/tiny-lts/v1/images/6f53b53753415b4d232e895ced294295_1898x101.png@900-0-90-f.png)
查看整个包的编译结果，我们发现函数`isFailedByRejUnspecidiedRecvByIsmf`无法被内联。我们在热补丁中可修改该函数。
![image.png](http://image.huawei.com/tiny-lts/v1/images/123b39a288b8d9c7a691e7a9198ef3f5_1548x140.png@900-0-90-f.png)




> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTkwMzk4NTY0OCwtMTgwNDcxNjQ2Nl19
-->
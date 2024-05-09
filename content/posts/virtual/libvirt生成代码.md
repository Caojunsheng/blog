

>背景：当我们需要修改xxx_protocol.x文件新增请求参数或者返回值参数时，又或者是需要修改自动生成出的代码时（gendispatch.pl），想先本地看看生成出的代码是否符合预期时，可以先在本地执行生成代码的操作，避免上环境编译才能得到生成代码。

### 1、RPC文件定义

当前存在下面三种RPC文件定义

 - src/remote/remote_protocol.x
 - src/remote/lxc_protocol.x
 - src/remote/qemu_protocol.x

```protobuf
// remote_protocol.x
……
// 一些常量定义和结构体定义
const REMOTE_STRING_MAX = 4194304;
typedef string remote_nonnull_string<REMOTE_STRING_MAX>;
typedef remote_nonnull_string *remote_string;
……
// 请求携带的参数
struct remote_connect_supports_feature_args {
    int feature;
};
// 返回值携带的参数
struct remote_connect_supports_feature_ret {
    int supported;
};
……
// 远程调用方法的序号
enum remote_procedure {
    REMOTE_PROC_CONNECT_OPEN = 1,
    REMOTE_PROC_CONNECT_CLOSE = 2,
    ……
}
```

### 2、perl生成代码文件

src/rpc/gendispatch.pl支持生成client和server两种代码
```perl
# Bodies for dispatch functions ("remote_dispatch.h").
elsif ($mode eq "server") 
# Bodies for client functions ("remote_client_bodies.h").
elsif ($mode eq "client")
```

通过Makefile自动生成client和server代码

src/remote/Makefile.inc.am
```
remote/remote_client_bodies.h: $(srcdir)/rpc/gendispatch.pl \
		$(REMOTE_PROTOCOL) Makefile.am
	$(AM_V_GEN)$(PERL) -w $(srcdir)/rpc/gendispatch.pl --mode=client \
	  remote REMOTE $(REMOTE_PROTOCOL) \
	  > remote/remote_client_bodies.h

remote/lxc_client_bodies.h: $(srcdir)/rpc/gendispatch.pl \
		$(LXC_PROTOCOL) Makefile.am
	$(AM_V_GEN)$(PERL) -w $(srcdir)/rpc/gendispatch.pl --mode=client \
	  lxc LXC $(LXC_PROTOCOL) \
	  > remote/lxc_client_bodies.h

remote/qemu_client_bodies.h: $(srcdir)/rpc/gendispatch.pl \
		$(QEMU_PROTOCOL) Makefile.am
	$(AM_V_GEN)$(PERL) -w $(srcdir)/rpc/gendispatch.pl --mode=client \
	  qemu QEMU $(QEMU_PROTOCOL) \
	  > remote/qemu_client_bodies.h

remote/remote_daemon_dispatch_stubs.h: $(srcdir)/rpc/gendispatch.pl \
		$(REMOTE_PROTOCOL) Makefile.am
	$(AM_V_GEN)$(PERL) -w $(top_srcdir)/src/rpc/gendispatch.pl \
	  --mode=server remote REMOTE $(REMOTE_PROTOCOL) \
	  > remote/remote_daemon_dispatch_stubs.h

remote/lxc_daemon_dispatch_stubs.h: $(srcdir)/rpc/gendispatch.pl \
		$(LXC_PROTOCOL) Makefile.am
	$(AM_V_GEN)$(PERL) -w $(top_srcdir)/src/rpc/gendispatch.pl \
	  --mode=server lxc LXC $(LXC_PROTOCOL) \
	  > remote/lxc_daemon_dispatch_stubs.h

remote/qemu_daemon_dispatch_stubs.h: $(srcdir)/rpc/gendispatch.pl \
		$(QEMU_PROTOCOL) Makefile.am
	$(AM_V_GEN)$(PERL) -w $(top_srcdir)/src/rpc/gendispatch.pl \
	  --mode=server qemu QEMU $(QEMU_PROTOCOL) \
	  > remote/qemu_daemon_dispatch_stubs.h

```

上面可以再Makefile里面生成，自然也可以通过perl在本地生成。

生成文件前需要再本地安装perl，perl安装外网可搜一篇通用安装教程。

参考上面的Makefile，可以得到下面的生成命令

```shell
# 生成remote_client_bodies.h文件
perl -w gendispatch.pl --mode=client remote REMOTE ../remote/remote_protocol.x  > ../remote/remote_client_bodies.h
# 生成lxc_client_bodies.h文件
perl -w gendispatch.pl --mode=client lxc LXC ../remote/lxc_protocol.x  > ../remote/lxc_client_bodies.h
# 生成qemu_client_bodies.h文件
perl -w gendispatch.pl --mode=client qemu QEMU ../remote/qemu_protocol.x  > ../remote/qemu_client_bodies.h
# 生成remote_daemon_dispatch_stubs.h文件
perl -w gendispatch.pl --mode=server remote REMOTE ../remote/remote_protocol.x  > ../remote/remote_daemon_dispatch_stubs.h
# 生成lxc_daemon_dispatch_stubs.h文件
perl -w gendispatch.pl --mode=server lxc LXC ../remote/lxc_protocol.x  > ../remote/lxc_daemon_dispatch_stubs.h
# 生成qemu_daemon_dispatch_stubs.h文件
perl -w gendispatch.pl --mode=server qemu QEMU ../remote/qemu_protocol.x  > ../remote/qemu_daemon_dispatch_stubs.h
```

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0MDkzNzI1MjJdfQ==
-->
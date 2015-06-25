title: coding-style
date: 2015-06-25 14:47:20
tags: 代码风格
---

在写代码的过程中发现自己存在不少问题，虽然已经在加强这方面，但还是有不少空间可以改进，现在把这些记录下来

##类型检查
函数参数合法性检查：
```c
int func(unsigned char *byte_val, unsigned int byte_index)
if (byte_val == NULL)
	return -1;
```

##保持整体代码风格一致
原来代码中有一段这样的数据结构：
```c
struct regs {
	u32 cfg;

	u32 ctl;

	u32 cnt;

	u32 read_dat;

	u32 dat[6];
};
```
在访问寄存器时最好使用该数据结构，不要再加上类似于下面的这些宏定义：
```c
/* reg addr */
#define EFUSE_CFG 0x0	
#define EFUSE_CTL 0x4	
#define EFUSE_CNT 0x8	
#define EFUSE_RDATA 0xc	
#define EFUSE_OUT0 0x10	
#define EFUSE_OUT1 0x14	
#define EFUSE_OUT2 0x18	
#define EFUSE_OUT3 0x1c	
#define EFUSE_OUT4 0x20	
#define EFUSE_OUT5 0x24
```
而是应该使用类似下面风格的代码：
```c
/* assign base address */
struct regs *_regs = 0xXXXXXXXX;
readl(&_regs->cfg);
```

##变量命名
变量命名之前应该好好斟酌名字是否会造成歧义，虽然之前看过《The art of readable code》，但真正自己去命名时才发现要区取一个好名字还是很不容易的。

##规则检查
内核源码中有一个很不错的coding style检查脚本，位置是./script/checkpatch.pl，即使是应用程序也可以用这个工具检查，用法自行google吧。

##添加patch
关于如何提交patch，内核中有专门的文档讲这个，位置是./doc/Documentation/SubmittingPatches，目前对我最有用的是
####Describe your changes
Describe user-visible impact.  Straight up crashes and lockups are
pretty convincing, but not all bugs are that blatant.  Even if the
problem was spotted during code review, describe the impact you think
it can have on users.  Keep in mind that the majority of Linux
installations run kernels from secondary stable trees or
vendor/product-specific trees that cherry-pick only specific patches
from upstream, so include anything that could help route your change
downstream: provoking circumstances, excerpts from dmesg, crash
descriptions, performance regressions, latency spikes, lockups, etc.

Quantify optimizations and trade-offs.  If you claim improvements in
performance, memory consumption, stack footprint, or binary size,
include numbers that back them up.  But also describe non-obvious
costs.  Optimizations usually aren't free but trade-offs between CPU,
memory, and readability; or, when it comes to heuristics, between
different workloads.  Describe the expected downsides of your
optimization so that the reviewer can weigh costs against benefits.

####Seperate your changes
Separate each _logical change_ into a separate patch.

For example, if your changes include both bug fixes and performance
enhancements for a single driver, separate those changes into two
or more patches.  If your changes include an API update, and a new
driver which uses that new API, separate those into two patches.

##关于如何选择开源的库
这个问题也是我看了free electrons的training material才知道的，主要的思路就是看它的contributions，对各个平台的支持，有时候看看sourceforge上的下载量也能反映一些问题



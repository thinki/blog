title: Linux字符设备驱动笔记
date: 2015-06-25 15:28:47
tags: 字符设备
---

最近在给Soc的字符设备添加一些功能，记录下遇到的问题：

##ioctl方法
很多资料都不推荐通过ioctl来控制设备，虽然至今我还没有明白这是为什么，不过好歹我知道可以用sysfs做一些简单的控制。
这里引用内核文档的内容/linux/Documentation/ioctl/ioctl-number.txt，If you are adding new ioctl's to the kernel, you should use the _IO macros defined in <linux/ioctl.h>，其他的注意事项也很详细的写在里面，虽然上面的很多ioctl号有很久没有更新了。

##void指针
driver常常需要操作寄存器，一次不经意间把int *指针的递增与void *的递增混淆了，我误认为int *指针递增1实际地址也是递增1，这是非常严重的错误，还好后来发现了囧

[在ANSIC标准中，不允许对void指针进行算术运算如pvoid++或pvoid+=1等，而在GNU中则允许，因为在缺省情况下，GNU认为void *与char *一样。sizeof(*pvoid)==sizeof(char)](http://blog.csdn.net/geekcome/article/details/6249151)

##内核与用户空间传递参数
###大小端
考虑到不同平台的可移植性，可以预先定义好传递的规则，比如用户空间强制把数据转换成大端，然后内核再强制把大端转换成自己体系结构的大小端：

用户空间
```c
uint8_t buf[4];	
*(uint32_t *)buf = htobe32(addr);

ioctl(fd, IOC_IOREAD, buf);

return be32toh(*(uint32_t *)buf);
```
内核空间
```c
u32 reg_val, reg_addr;
u8 buf[4];

copy_from_user(buf, (char *)arg, 4);

reg_addr = be32_to_cpu(*(u32 *)buf);
....
/* process data */
....

*(u32 *)buf = cpu_to_be32(reg_val);
copy_to_user((char *)arg, buf, 4);
```

###ioctl args
字符设备驱动中ioctl的args参数常常用作内核与用户空间通信的桥梁，在内核中读写该参数时切记要用copy_from_user或者copy_to_user，这2个函数其实是老生长谈了，主要原因还是因为[用户空间传过来的指针是在虚拟地址空间上的，它指向的虚拟地址空间很可能还没有真正映射到实际的物理页面上](http://blog.csdn.net/ce123_zhouwei/article/details/8454226)。

##读写延时
一般在字符设备读写过程中可能会遇到不同程度的delay时序要求，但最好不要delay太久，比如delay 10ms算是一个比较大的时延了，如果是在允许睡眠的上下文中，换用msleep也不失为一种好的代替方案。

##Kconfig中depends on和select的区别
写完字符设备以后就要添加Kconfig规则，当遇到依赖关系时有两种选择：depends和select
这两个的区别在于depends on的目录项只有在依赖的宏被选上的情况下才会出现，而select是当选上该项以后会默认也勾上select选中的宏。
>* 某个程序依赖于某个library时，使用select

>* 小的应用依赖于大的应用时，使用depends on
（比如某个audio driver依赖于alsa框架，这个时候应该用depends on）


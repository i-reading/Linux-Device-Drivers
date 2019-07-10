# CH04:Linux内核模块
一个Linux内核模块主要由以下几部分组成：
* 模块加载函数：通过insmod命令加载内核模块时，模块的加载函数会自动被内核执行，完成本模块的相关初始化工作。
* 模块卸载函数：通过rmmod命令卸载模块时，模块的卸载函数会自动被内核执行，完成与模块加载函数相反的功能。
* 模块许可证声明：许可证(LICENSE)声明描述内核模块的许可权限，，如果不声明LICENSE，模块被加载时，将收到内核被污染(Kernel Tainted)的警告。
* 模块参数：模块参数是模块被加载的时候可以传递给它的值，它本身对应模块内部的全局变量。
* 模块导出符号：内核模块可以导出的符号(symbol，对应于函数或变量)，若导出，其他模块则可以使用本模块的变量或函数。
* 模块作者等信息声明

## 模块加载函数
一般以` __init `标识声明，典型的加载函数的形式代码如下：
```
static int __init initialization_func(void)
{
	/*初始化代码*/
}
module_init(initialization_func);

```
模块加载函数以` module_init(函数名) `的形式被指定，返回整型值，若初始化成功，应返回0，初始化失败则返回错误编码（在Linux内核里，错误编码是一个接近于0的负值，在` <linux/errno.h> `中定义，包含` -ENODEV、-ENOMEM `之类的符号值）。

在Linux中，所以标识为` __init `的函数如果直接编译进内核，成为内核镜像的一部分，在链接的时候都会放在` .init.text `这个区段内。
```
#define __init __attribute__ ((__section__(".init.text")))

```
所有的` __init `函数在区段` .initcall.init `中还保存了一份函数指针，在初始化时内核会通过这些函数指针调用这些` __init `函数，并在初始化完成后，释放` init `区段的内存。
除了函数以外，数据也可以被定义为` __initdata `，对于只是初始化阶段需要的数据，内核在初始化完成后，也可以释放它们占用的内存。例如，下面代码将` hello_data `定义为` __initdata `:
```
static int hello_data __initdata = 1;

static int __init hello_init(void)
{
	printk(KERN_INFO "hello,world %d\n,hello_data");
	return 0;
}
module_init(hello_init);

static void __exit hello_exit(void)
{
	printk(KERN_INFO "Goodbye,world\n");
}
module_exit(hello_exit);
```

---
本文内容摘自[Linux设备驱动开发详解-基于最新的Linux 4.0内核](http://www.cmpbook.com/stackroom.php?id=41234)，在nVIDIA AGX-Xavier平台上运行。
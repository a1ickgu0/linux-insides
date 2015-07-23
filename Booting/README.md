# 内核引导过程（Kernel boot process）

This chapter describes the linux kernel boot process. You will see here a
couple of posts which describe the full cycle of the kernel loading process:

这章节描述 Linux 内核的引导过程，在这个部分你会看到几篇描述内核完整加载过程的文章：

* [From the bootloader to kernel](http://0xax.gitbooks.io/linux-insides/content/Booting/linux-bootstrap-1.html) - describes all stages from turning on the computer to before the first instruction of the kernel;
* [First steps in the kernel setup code](http://0xax.gitbooks.io/linux-insides/content/Booting/linux-bootstrap-2.html) - describes first steps in the kernel setup code. You will see heap initialization, querying of different parameters like EDD, IST and etc...
* [Video mode initialization and transition to protected mode](http://0xax.gitbooks.io/linux-insides/content/Booting/linux-bootstrap-3.html) - describes video mode initialization in the kernel setup code and transition to protected mode.
* [Transition to 64-bit mode](http://0xax.gitbooks.io/linux-insides/content/Booting/linux-bootstrap-4.html) - describes preparation for transition into 64-bit mode and transition into it.
* [Kernel Decompression](http://0xax.gitbooks.io/linux-insides/content/Booting/linux-bootstrap-5.html) - describes preparation before kernel decompression and directly decompression.


* [从 Bootloader 加载内核](https://github.com/a1ickgu0/linux-insides/blob/master/Booting/linux-bootstrap-1.md) - 介绍计算机上电到执行内核的第一条指令之前的所有过程。
* [内核引导的第一个步骤](https://github.com/a1ickgu0/linux-insides/blob/master/Booting/linux-bootstrap-2.md) - 介绍内核引导的第一个步骤，你将看到堆初始化，加载不同参数，如 EDD，IST 等等。
* [视频模式初始化并进入保护模式](https://github.com/a1ickgu0/linux-insides/blob/master/Booting/linux-bootstrap-3.md) - 介绍内核引导的视频模式初始，以及进入保护模式。
* [进入 64位 模式](https://github.com/a1ickgu0/linux-insides/blob/master/Booting/linux-bootstrap-4.md) - 介绍内核向 64位 模式转换过程。
* [内核解压](https://github.com/a1ickgu0/linux-insides/blob/master/Booting/linux-bootstrap-5.md) - 介绍内核的解压准备以及内核解压过程。



可执行可链接文件格式（Executable and Linkable Format）
================================================================================

ELF (Executable and Linkable Format) is a standard file format for executable files and shared libraries. Linux as many UNIX-like operating systems uses this format. Let's look on structure of the ELF-64 Object File Format  and some defintions in the linux kernel source code related with it.

ELF（Executable and Linkable Format，可执行可链接文件格式）是可执行文件以及动态链接库的文件格式标准。Linux 以及许多的 Unix-Like 操作系统都采用这种文件格式。这篇文章我们将了解到 ELF-64 对象文件格式（Object File Format）的结构以及 Linux 内核源代码中与之相关的部分定义内容。

An ELF object file consists of the following parts:

ELF 对象文件包括以下几个部分：

* ELF header - describes the main characteristics of the object file: type, CPU architecture, the virtual address of the entry point, the size and offset the remaining parts, etc...;
* Program header table - listing the available segments and their attributes. Program header table need loaders for placing sections of the file as virtual memory segments;
* Section header table - contains description of the sections.

* ELF 头 （ELF header）- 描述对象文件的主要特征，包括：文件类型、CPU架构、入口虚地址（entry point）、文件大小以及剩余各部分偏移等等。
* 程序头表（Program header table）- 保存有效段（segments）以及其属性信息，程序头表被加载器（Loader）所使用。
* 节头表（Section header table）-  保存各节（section） 的描述信息。

Now let's look closer on these components.

现在我们来深入看一下这几个组成部分的具体内容。

**ELF header**

**ELF 头**

It's located in the beginning of the object file. It's main point is to locate all other parts of the object file. File header contains following fields:

ELF 头存储在对象文件的起始位置，其主要作用是定位对象文件各组成部分，ELF 头主要包括以下内容：

* ELF identification - array of bytes which helps to identify the file as an ELF object file and also provides information about general object file characteristic;
* Object file type - identifies the object file type. This field can describe that ELF file is relocatable object file, executable file, etc...;
* Target architecture;
* Version of the object file format;
* Virtual address of the program entry point;
* File offset of the program header table;
* File offset of the section header table;
* Size of an ELF header;
* Size of a program header table entry;
* and other fields...

* ELF 标识：一组字节信息（幻数，Magic Number）用于标识文件为 ELF 对象文件，同时还包括此文件的一些基本信息。
* 对象文件类型：标识文件类型，说明对象文件：为可重定位的对象文件 （relocatable object file），可执行的对象文件（executable object file），等等。
*  CPU 架构信息。
* 对象文件格式的版本。
* 入口虚地址。
* 程序头表的偏移地址。
* 节头表的偏移地址。
* ELF 头长度。
* 程序头表长度。
* 以及其他信息域。

You can find `elf64_hdr` structure which presents ELF64 header in the linux kernel source code:

在 Linux 源码中，你可以找到 ELF64 文件头定义的结构体 `elf64_hdr`，内容如下：

```C
typedef struct elf64_hdr {
	unsigned char	e_ident[EI_NIDENT];
	Elf64_Half e_type;
	Elf64_Half e_machine;
	Elf64_Word e_version;
	Elf64_Addr e_entry;
	Elf64_Off e_phoff;
	Elf64_Off e_shoff;
	Elf64_Word e_flags;
	Elf64_Half e_ehsize;
	Elf64_Half e_phentsize;
	Elf64_Half e_phnum;
	Elf64_Half e_shentsize;
	Elf64_Half e_shnum;
	Elf64_Half e_shstrndx;
} Elf64_Ehdr;
```

This structure defined in the [elf.h](https://github.com/torvalds/linux/blob/master/include/uapi/linux/elf.h)

结构体定义在源文件  [elf.h](https://github.com/torvalds/linux/blob/master/include/uapi/linux/elf.h) 中

**Sections**

**Section 部分**


All data stores in a sections in an Elf object file. Sections identified by index in the section header table. Section header contains following fields:

ELF 对象文件的所有数据存储在不同的 Section 中，各个 Section 通过 Section Header Table 中的索引标识，Section 头中包括以下字段：

* Section name;
* Section type;
* Section attributes;
* Virtual address in memory;
* Offset in file;
* Size of section;
* Link to other section;
* Miscellaneous information;
* Address alignment boundary;
* Size of entries, if section has table;

* Section 名字。
* Section 类型。
* Section 属性。
* 内存虚地址。
* 文件内偏移。
* Section的长度。
* 到其它 Section 的链接。
* 扩展的 Section 信息。
* 地址对齐边界。
* 表成员数量（如果 Section 中包括 Table 的话）。

And presented with the following `elf64_shdr` structure in the linux kernel:

具体的可以参看 Linux 源码中的 `elf64_shdr` 结构体定义：

```C
typedef struct elf64_shdr {
	Elf64_Word sh_name;
	Elf64_Word sh_type;
	Elf64_Xword sh_flags;
	Elf64_Addr sh_addr;
	Elf64_Off sh_offset;
	Elf64_Xword sh_size;
	Elf64_Word sh_link;
	Elf64_Word sh_info;
	Elf64_Xword sh_addralign;
	Elf64_Xword sh_entsize;
} Elf64_Shdr;
```

**Program header table**

**程序头表**

All sections are grouped into segments in an executable or shared object file. Program header is an array of structures which describe every segment. It looks like:

可执行对象文件与动态链接对象文件的各个部分都以段（segment）的方式组织，程序头表是一组描述各个段属性的结构，具体如下：

```C
typedef struct elf64_phdr {
	Elf64_Word p_type;
	Elf64_Word p_flags;
	Elf64_Off p_offset;
	Elf64_Addr p_vaddr;
	Elf64_Addr p_paddr;
	Elf64_Xword p_filesz;
	Elf64_Xword p_memsz;
	Elf64_Xword p_align;
} Elf64_Phdr;
```

in the linux kernel source code.

`elf64_phdr` defined in the same [elf.h](https://github.com/torvalds/linux/blob/master/include/uapi/linux/elf.h).

Linux 源码中 `elf64_phdr` 也同样是在源文件 [elf.h](https://github.com/torvalds/linux/blob/master/include/uapi/linux/elf.h) 中定义。

And ELF object file also contains other fields/structures which you can find in the [Documentation](http://downloads.openwatcom.org/ftp/devel/docs/elf-64-gen.pdf). Better let's look on the `vmlinux`.

ELF 对象文件同时还包括一些其他的域与结构体，你可以在这个[文档](http://downloads.openwatcom.org/ftp/devel/docs/elf-64-gen.pdf)中找到。 

接下来我们来看一下 `vmlinux` 文件。

vmlinux
--------------------------------------------------------------------------------

`vmlinux` is relocatable ELF object file too. So we can look on it with the `readelf` util. First of all let's look on a header:


`vmlinux` 也是个可重定位的对象 ELF 文件（relocatable object file），因此我们可以用使用 `readelf` 工具查看其结构，首先看一下文件的头信息：

```
$ readelf -h  vmlinux
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1000000
  Start of program headers:          64 (bytes into file)
  Start of section headers:          381608416 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         5
  Size of section headers:           64 (bytes)
  Number of section headers:         73
  Section header string table index: 70
```

Here we can see that `vmlinux` is 64-bit executable file.

We can read from the [Documentation/x86/x86_64/mm.txt](https://github.com/torvalds/linux/blob/master/Documentation/x86/x86_64/mm.txt):

这里可以看到，`vmlinux` 是个 64 位的可执行文件，我们可以从  [Documentation/x86/x86_64/mm.txt](https://github.com/torvalds/linux/blob/master/Documentation/x86/x86_64/mm.txt) 查看到地址定义：
```
ffffffff80000000 - ffffffffa0000000 (=512 MB)  kernel text mapping, from phys 0
```

So we can find it in the `vmlinux` with:

接下来我们查看一下 `vmlinux` 的符号信息：

```
readelf -s vmlinux | grep ffffffff81000000
     1: ffffffff81000000     0 SECTION LOCAL  DEFAULT    1 
 65099: ffffffff81000000     0 NOTYPE  GLOBAL DEFAULT    1 _text
 90766: ffffffff81000000     0 NOTYPE  GLOBAL DEFAULT    1 startup_64
```

Note that here is address of the `startup_64` routine is not `ffffffff80000000`, but `ffffffff81000000` and now i'll explain why.

注意，这里我们看到的 `startup_64` 地址是 `ffffffff81000000` 而不是 `ffffffff80000000` ，接下来我们看为什么是这样：

We can see following definition in the [arch/x86/kernel/vmlinux.lds.S](https://github.com/torvalds/linux/blob/master/arch/x86/kernel/vmlinux.lds.S):

我们可以从 [arch/x86/kernel/vmlinux.lds.S](https://github.com/torvalds/linux/blob/master/arch/x86/kernel/vmlinux.lds.S) 看到如下定义：


```
    . = __START_KERNEL;
	...
	...
	..
	/* Text and read-only data */
	.text :  AT(ADDR(.text) - LOAD_OFFSET) {
		_text = .;
		...
		...
		...
	}
```

Where `__START_KERNEL` is:

其中的 `__START_KERNEL` 定义如下：

```
#define __START_KERNEL		(__START_KERNEL_map + __PHYSICAL_START)
```

`__START_KERNEL_map` is the value from documentation - `ffffffff80000000` and `__PHYSICAL_START` is `0x1000000`. That's why address of the `startup_64` is `ffffffff81000000`.

`__START_KERNEL_map` 的值为 `ffffffff80000000` ，同时 `__PHYSICAL_START` 值是 `0x1000000`，这个就是为什么 `startup_64` 地址是  `ffffffff81000000 。

And the last we can get program headers from `vmlinux` with the following command:

最后，我们可以通过以下命令读取 `vmlinux` 的程序头表信息：

```
readelf -l vmlinux

Elf file type is EXEC (Executable file)
Entry point 0x1000000
There are 5 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  LOAD           0x0000000000200000 0xffffffff81000000 0x0000000001000000
                 0x0000000000cfd000 0x0000000000cfd000  R E    200000
  LOAD           0x0000000001000000 0xffffffff81e00000 0x0000000001e00000
                 0x0000000000100000 0x0000000000100000  RW     200000
  LOAD           0x0000000001200000 0x0000000000000000 0x0000000001f00000
                 0x0000000000014d98 0x0000000000014d98  RW     200000
  LOAD           0x0000000001315000 0xffffffff81f15000 0x0000000001f15000
                 0x000000000011d000 0x0000000000279000  RWE    200000
  NOTE           0x0000000000b17284 0xffffffff81917284 0x0000000001917284
                 0x0000000000000024 0x0000000000000024         4

 Section to Segment mapping:
  Segment Sections...
   00     .text .notes __ex_table .rodata __bug_table .pci_fixup .builtin_fw
          .tracedata __ksymtab __ksymtab_gpl __kcrctab __kcrctab_gpl
		  __ksymtab_strings __param __modver 
   01     .data .vvar 
   02     .data..percpu 
   03     .init.text .init.data .x86_cpu_dev.init .altinstructions
          .altinstr_replacement .iommu_table .apicdrivers .exit.text
		  .smp_locks .data_nosave .bss .brk
```

Here we can see five segments with sections list. All of these sections you can find in the generated linker script at - `arch/x86/kernel/vmlinux.lds`.

我们可以看到 `vmlinux` 一共包含 5个段，各个段的具体定义可参看 Linux 源代码 `arch/x86/kernel/vmlinux.lds.S` 文件。

That's all. Of course it's not a full description of ELF object format, but if you are interesting in it, you can find documentation - [here](http://downloads.openwatcom.org/ftp/devel/docs/elf-64-gen.pdf)

本文到这里就结束了，但这个并不是 ELF 对象文件的全部内容，如果你有兴趣对此进一步了解，你可以看 [这份](http://downloads.openwatcom.org/ftp/devel/docs/elf-64-gen.pdf) 文档。



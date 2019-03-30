---
title: CTF | Return-to-dl-resolve
date: 2018-09-17 20:40:21
tags: 
---

首先需要理解 ELF 文件格式以及动态链接的过程。

### 相关结构

其中Tag对应着每个节。比如JMPREL对应着.rel.plt
32 位：
```c
typedef struct
{
  unsigned char        e_ident[EI_NIDENT];        /* Magic number and other info */
  Elf32_Half        e_type;                        /* Object file type */
  Elf32_Half        e_machine;                /* Architecture */
  Elf32_Word        e_version;                /* Object file version */
  Elf32_Addr        e_entry;                /* Entry point virtual address */
  Elf32_Off        e_phoff;                /* Program header table file offset */
  Elf32_Off        e_shoff;                /* Section header table file offset */
  Elf32_Word        e_flags;                /* Processor-specific flags */
  Elf32_Half        e_ehsize;                /* ELF header size in bytes */
  Elf32_Half        e_phentsize;                /* Program header table entry size */
  Elf32_Half        e_phnum;                /* Program header table entry count */
  Elf32_Half        e_shentsize;                /* Section header table entry size */
  Elf32_Half        e_shnum;                /* Section header table entry count */
  Elf32_Half        e_shstrndx;                /* Section header string table index */
} Elf32_Ehdr;
```

64 位：
```c
typedef struct
{
  unsigned char        e_ident[EI_NIDENT];        /* Magic number and other info */
  Elf64_Half        e_type;                        /* Object file type */
  Elf64_Half        e_machine;                /* Architecture */
  Elf64_Word        e_version;                /* Object file version */
  Elf64_Addr        e_entry;                /* Entry point virtual address */
  Elf64_Off        e_phoff;                /* Program header table file offset */
  Elf64_Off        e_shoff;                /* Section header table file offset */
  Elf64_Word        e_flags;                /* Processor-specific flags */
  Elf64_Half        e_ehsize;                /* ELF header size in bytes */
  Elf64_Half        e_phentsize;                /* Program header table entry size */
  Elf64_Half        e_phnum;                /* Program header table entry count */
  Elf64_Half        e_shentsize;                /* Section header table entry size */
  Elf64_Half        e_shnum;                /* Section header table entry count */
  Elf64_Half        e_shstrndx;                /* Section header string table index */
} Elf64_Ehdr;
```

`.rel.plt`节是用于函数重定位，`.rel.dyn`节是用于变量重定位

```c
typedef struct
{
  Elf32_Addr        r_offset;                /* Address */
  Elf32_Word        r_info;                        /* Relocation type and symbol index */
} Elf32_Rel;
```
`.got`节保存全局变量偏移表，`.got.plt`节保存全局函数偏移表。.got.plt对应着Elf32_Rel结构中r_offset的值。

`.symtab` 符号表，是一个数组，每个元素都是一个结构体。
`.dynsym`节包含了动态链接符号表。`Elf32_Sym[num]`中的`num`对应着`ELF32_R_SYM(Elf32_Rel->r_info)`。根据定义

- st_name :保存符号在 dynstr 的偏移


`strtab` 该节区描述默认的字符串表，包含了一系列的以 NULL 结尾的字符串。ELF 文件使用这些字符串来存储程序中的符号名，包括变量名,函数名。首尾都是`\0`


`.dynstr`节包含了动态链接的字符串。这个节以\x00作为开始和结尾，中间每个字符串也以\x00间隔。

`.plt`节是过程链接表。过程链接表把位置独立的函数调用重定向到绝对位置。

重定位：
.dynstr 就会包含对应函数名称的字符串，`.dynsym` 中就会包含一个具有相应名称的动态字符串表的符号（Elf_Sym），在 `rel.dyn` 中就会包含一个指向这个符号的的重定位表项。


```c
typedef struct {
    Elf32_Word      st_name;
    Elf32_Addr      st_value;
    Elf32_Word      st_size;
    unsigned char   st_info;
    unsigned char   st_other;
    Elf32_Half      st_shndx;
} Elf32_Sym;
```

### 解析过程：
1. 根据 reloc_index 计算相应的重定位表项
2. 根据重定位表项的 r->info 得到在符号表的索引
3. 继而得到对应的符号
4. 判断符号是否为 ELF_MACHINE_JMP_SLOT 
5. 判断符号是否解析过
6. 得到符号对应的版本
7. 根据name 来寻找相应函数在库中的位置 `strtab + sym->st_name`


read -r 查看 .rel.plt 信息
延迟绑定过程
1. `jmp plt`,`call .got.plt`
2. 如果是第一次调用，会 `jmp plt`, 根据函数 `push offset`
3. 再一次 `jmp` ,把 `linkmap` 的地址入栈，再用` _dl_runtime_resolve` 找到 `.rel.plt` 中的结构体
4. 使用目标结构体中的 `r_info` ，以 `r_info >> 8*16` 的偏移，在符号表找到对应的结构体
5. 使用 `st_name` 成员作为偏移量,在 `dynstr` 中寻找对应函数的名称字符串
6. 在 `libc` 中搜索到这个函数的真实地址,并填写到 `.got.plt` 中
7. 调用目标函数



got[1]: link_map
got[2]: _dl_runtime_resolve

plt[0] 
### 漏洞利用方式
1.控制eip为PLT[0]的地址，只需传递一个index_arg参数
2.控制index_arg的大小，使reloc的位置落在可控地址内
3.伪造reloc的内容，使sym落在可控地址内
4.伪造sym的内容，使name落在可控地址内
5.伪造name为任意库函数，如system


`_dl_runtime_resolve`:根据传入的参数获取导出函数（这里是puts）的地址，然后放到相应的GOT表，并且调用它。而这个函数的地址也是从GOT表取并且jmp [xxx]过去的

`_dl_fixup`函数：




http://pwn4.fun/2016/11/09/Return-to-dl-resolve/
https://veritas501.space/2017/10/07/ret2dl_resolve%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/

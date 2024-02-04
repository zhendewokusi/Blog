---
title: DPL CPL RPL
date: 2024-02-04 00:21:51
tags:
---

# 前言

- 实验环境:Archlinux + Bochs x86 Emulator 2.7
- 编译器: nasm

# DPL

DPL全称是`Descriptor Privilege Level`,描述符特权级.这里是在保护模式设置段描述符的时候进行确认的,比如内核的特权级别是0,用户程序的特别级别是3,这是在设置描述符的时候就确定的.我还是在此解释一下什么是
描述符.

## 1. 什么是描述符

要解释描述符就要解释什么是实模式什么是保护模式,真是一路修到发电厂...

### 1.1 实模式与保护模式

#### 实模式为什么危险
计算机的世界只有0和1,而这些0和1被人为划定为指令和数据,CPU是比较傻的,它并不知道指令和数据的区别,它就像一个完全服从的士兵一样,你想让它干什么它就干什么.即使你在实模式下一不小心让CPU执行本该属于数据的地方,它也会自己说服自己,将数据强行理解成指令.也许你听起来比较懵,事实的确如此,我们简单来写一个汇编demo,不要害怕,即使你没有学习过汇编语言,相信在我的注释下你也能看懂,放码:
```asm
; 文件名: mbr.S
MBR_BASE_ADDR equ 0x7c00	; 类似于C语言的define宏定义,编译期间进行替换
org	MBR_BASE_ADDR			; org 指令告诉汇编器将代码放置在指定的内存地址处
    ; jmp code				; jmp 翻译:跳,跳到code那里运行
data:
    _var1 dw 0x55			; 定义了一个名为 _var1 的 16 位（即 2 字节）数据，初始值为 0x55
    _var2 dw 0x00			; 同理
code:
    mov ax,0				; 下面都是具体命令,我们此时不关心具体是什么指令
    mov ds,ax
    mov ss,ax
    mov esp,0x7c00
    push ax
    jmp $
    times 510 - ($ - $$) db 0 ; 由于运行环境是bochs的主引导程序,额外需要,此处不关心
    db 0x55,0xaa
```
代码从头开始运行(宏定义肯定没了),从第三行开始运行,我们来看调试过程:
```
<bochs:1> b 0x7c00
<bochs:2> c
(0) Breakpoint 1, 0x00007c00 in ?? ()
Next at t=17178866
(0) [0x000000007c00] 0000:7c00 (unk. ctxt): push bp                   ; 55
<bochs:3> n
Next at t=17178867
(0) [0x000000007c01] 0000:7c01 (unk. ctxt): add byte ptr ds:[bx+si], al ; 0000
<bochs:4> n
Next at t=17178868
(0) [0x000000007c03] 0000:7c03 (unk. ctxt): add byte ptr ds:[bx+si], bh ; 00b80000
<bochs:5> n
Next at t=17178869
(0) [0x000000007c07] 0000:7c07 (unk. ctxt): mov ds, ax                ; 8ed8
<bochs:6> n
Next at t=17178870
(0) [0x000000007c09] 0000:7c09 (unk. ctxt): mov ss, ax                ; 8ed0
<bochs:7> n
Next at t=17178871
(0) [0x000000007c0b] 0000:7c0b (unk. ctxt): mov esp, 0x00007c00       ; 66bc007c0000
<bochs:8> n
Next at t=17178872
(0) [0x000000007c11] 0000:7c11 (unk. ctxt): push ax                   ; 50
<bochs:9> n
Next at t=17178873
(0) [0x000000007c12] 0000:7c12 (unk. ctxt): jmp .-2  (0x00007c12)     ; ebfe
```
相信大家看到了,这里多出来了我们没有写的指令`push bp`,`add byte ptr ds:[bx+si], al`和`add byte ptr ds:[bx+si], bh`,而且少了`mov ax,0`,再看调试过程最右侧的机器码,很明显,我们的`_var1`和`_var2`被当成了指令来执行,而这不符合我们代码的预期.
将注释的`jmp code`语句加上后,我们再来调试一下:
```
<bochs:1> b 0x7c00
<bochs:2> c
(0) Breakpoint 1, 0x00007c00 in ?? ()
Next at t=17178866
(0) [0x000000007c00] 0000:7c00 (unk. ctxt): jmp .+4  (0x00007c06)     ; eb04
<bochs:3> n
Next at t=17178867
(0) [0x000000007c06] 0000:7c06 (unk. ctxt): mov ax, 0x0000            ; b80000
<bochs:4> 
Next at t=17178868
(0) [0x000000007c09] 0000:7c09 (unk. ctxt): mov ds, ax                ; 8ed8
<bochs:5> 
Next at t=17178869
(0) [0x000000007c0b] 0000:7c0b (unk. ctxt): mov ss, ax                ; 8ed0
<bochs:6> 
Next at t=17178870
(0) [0x000000007c0d] 0000:7c0d (unk. ctxt): mov esp, 0x00007c00       ; 66bc007c0000
<bochs:7> 
Next at t=17178871
(0) [0x000000007c13] 0000:7c13 (unk. ctxt): push ax                   ; 50
<bochs:8> 
Next at t=17178872
(0) [0x000000007c14] 0000:7c14 (unk. ctxt): jmp .-2  (0x00007c14)     ; ebfe
```
此时就完全正确了,在早期实模式下,更可怕的是没有限制的自由,你想访问哪儿都可以,想改变指定内存的数据都是可以的,程序A的数值被程序B就可以随意更改,甚至更改程序A的代码运行逻辑,这是很可怕的,所以就有了保护模式。

#### 保护模式

当时急需对电脑中的程序有特权等级的划分,以及代码段,数据段等等的限制,段属性,段基址,段界限,为了提高电脑运行时候的安全性:如果CPU不小心运行到数据段就报错,用户程序试图修改不属于自己的段信息就会报错,这样安全程度就会大大提高,上述需求的实现均基于一种数据结构:段描述符。段描述符通常由一组16字节的数据表示，包含了内存段的各种属性和控制信息。
<!-- 此处来个描述符的图[]() -->

我们来看看具体的描述符初始化的环境:
```asm
; 文件名 : test.S 
 DA_DRW		equ	92h	; 存在的可读写数据段属性值
 DA_DPL3		equ	  60h	; DPL = 3
 ; 描述符的宏实现
 ; usage: Descriptor Base, Limit, Attr
 ;        Base:  dd
 ;        Limit: dd (low 20 bits available)
 ;        Attr:  dw (lower 4 bits of higher byte are always 0)
 %macro Descriptor 3
 	dw	%2 & 0FFFFh				; 段界限 1				(2 字节)
 	dw	%1 & 0FFFFh				; 段基址 1				(2 字节)
 	db	(%1 >> 16) & 0FFh			; 段基址 2				(1 字节)
 	dw	((%2 >> 8) & 0F00h) | (%3 & 0F0FFh)	; 属性 1 + 段界限 2 + 属性 2		(2 字节)
 	db	(%1 >> 24) & 0FFh			; 段基址 3				(1 字节)
 %endmacro ; 共 8 字节
 ; 下面来看一个具体的使用
 LABEL_DESC_DATA:		Descriptor	           0,             0FFFFh, DA_DRW + DA_DPL3					; Data
```
动手强的可以来编译看一看:

## 访问者的特权标签

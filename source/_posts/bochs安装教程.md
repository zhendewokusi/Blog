---
title: bochs安装教程
date: 2024-01-17 19:49:39
tags: [bochs] [真相还原]
---



```shell
CONFIGURE_ARGS="--enable-gdb-stub"
CONFIGURE_ARGS="--enable-debugger --enable-readline"
```

先改`.conf.linux`，里面的
```config
CONFIGURE_ARGS="--enable-gdb-stub"
```

再使用：
```shell
sh .conf.linux 
```
然后在Makefile里面操作:

我是在vscode中,Ctrl+F，将所有的`bochs`更改成`bochs_gdb`，接着将`bochs_gdb.h`更改为`bochs.h`,`wxbochs_gdb.rc`更改为`wxbochs.rc`，后面这个应该没影响，是和win有关系的，但是保险起见还是改了吧，然后进行`make -j16 && sudo make install`。

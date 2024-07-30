```S
.macro tlb_do_page_fault, write
SYM_CODE_START(tlb_do_page_fault_\write)
	SAVE_ALL
	csrrd	a2, LOONGARCH_CSR_BADV	# 读取发生页面错误的地址，放入 a2 寄存器
	move	a0, sp			# 将堆栈指针复制到 a0 寄存器
	REG_S	a2, sp, PT_BVADDR	# 将寄存器 a2 的值存储到堆栈上的 PT_BVADDR 偏移位置
	li.w	a1, \write		# 将\write加载到寄存器a1中，判断是读操作引起的还是写操作引起的
     // bl	do_page_fault
	RESTORE_ALL_AND_RET		# 恢复所有寄存器信息
SYM_CODE_END(tlb_do_page_fault_\write)
.endm
```
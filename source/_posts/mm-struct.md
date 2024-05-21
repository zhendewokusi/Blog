---
title: mm_struct
date: 2024-05-18 22:21:58
tags:
---

static inline void
__vma_link_list(struct mm_struct *mm, struct vm_area_struct *vma,
		struct vm_area_struct *prev, struct rb_node *rb_parent)
{
	if (prev) {
		vma->vm_next = prev->vm_next;
		prev->vm_next = vma;
	} else {
		mm->mmap = vma;
		if (rb_parent)
			vma->vm_next = rb_entry(rb_parent,
					struct vm_area_struct, vm_rb);
		else
			vma->vm_next = NULL;
	}
}

load_elf_binary ->setup_arg_pages -> insert_vm_struct -> vma_link ->  
load_elf_binary -> elf_map -> do_mmap -> do_mmap_pgoff -> get_unmapped_area
setup_arg_pages -> insert_vm_struct -> vma_link-> __vma_link
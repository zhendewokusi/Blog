---
title: slab 分配器
date: 2024-06-04 20:27:35
tags: [内核]
---
## 前言

本文是阅读 kernel 文档中关于 Slab 分配器的文档所写的笔记。

slab分配器背后的基本思想：让常用对象的缓存保持在初始化状态供给内核使用。

## slab 分配器的三个主要目标

1. 分配小内存块有助于消除由伙伴系统引起的内部碎片。slab内部维护了两组小内存缓冲区的缓存，范围从32字节到131072（2的17次方）字节。
2. 缓存常用的对象，使系统不会浪费时间分配、初始化和销毁对象。当创建一个新的slab时，将很多对象进行打包并且进行初始化；当对象被释放，它保持其初始化状态，以便于下一次的对象分配。
3. 通过将对象与L1与L2缓存对齐，更好利用硬件缓存。这是slab的最本质的任务，如果一个对象打包进slab后还有剩余空间，slab就将其进行着色，使其与CPU L1缓存对齐。

## 主体

 slabs_full 的所有对象都在使用中。 slabs_partial 中有空闲对象，因此是分配对象的主要候选者。 slabs_free 没有分配的对象，是slab 销毁的主要候选者。

```c
// The slab lists for all objects.
struct kmem_cache_node {
	spinlock_t list_lock;

#ifdef CONFIG_SLAB
	struct list_head slabs_partial;	/* partial list first, better asm code */
	struct list_head slabs_full;
	struct list_head slabs_free;
	unsigned long total_slabs;	/* length of all slab lists */
	unsigned long free_slabs;	/* length of free slab list only */
	unsigned long free_objects;
	unsigned int free_limit;
	unsigned int colour_next;	/* Per-node cache coloring */
	struct array_cache *shared;	/* shared per node */
	struct alien_cache **alien;	/* on other nodes */
	unsigned long next_reap;	/* updated without locking */
	int free_touched;		/* updated without locking */
#endif

#ifdef CONFIG_SLUB
	unsigned long nr_partial;
	struct list_head partial;
#ifdef CONFIG_SLUB_DEBUG
	atomic_long_t nr_slabs;
	atomic_long_t total_objects;
	struct list_head full;
#endif
#endif

};
```
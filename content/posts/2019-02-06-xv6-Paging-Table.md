---
title: "xv6 Basics - Page Tables"
date: 2019-02-06
draft: false
---

**Paging tables are the mechanism through which the operating system controls what memory address means.** They allow xv6 to multiplex the address spaces of different processes onto a single physical memory, and to protect the memories of different processes. The level of indirection provided by page tables allows many neat tricks. xv6 uses page tables primarily to multiplex address spaces and to protect memory.

<!--more-->

It also uses a few simple page-table tricks:

* Mapping the same memory (the kernel) in several address spaces.
* Mapping the same memory more than once in one address space. (Each user page is also mapped into the kernel's physical view of memory)
* Guarding a user stack with an ummapped page.

# Paging hardware

As a reminder, x86 instructions (both user and kernel) manipulate virtual addresses. The mechine's RAM, or physical memory, is indexed with physical addresses.

The x86 page table hardware connects these two kinds of addresses, by mapping each virtual address to a physical address.

```
   10      10          12          |     PPN     | Offset |  
|-------|---------|----------|            ^
|  Dir  |  Table  |  Offset  |        20  |          12
|-------|----|----|----------|     |------|------|-------|
    |        |-------------------->|     PPN     | Flags |
    |                              |-------------|-------|
    |         20       10          |             |       |
    |     |---------|-------|      |-------------|-------|
    |     |         |       |      |             |       |
    |     |---------|-------|      |-------------|-------|
    |---->|   PPN   | Flags |------->     Page Table
          |---------|-------|         2^10(1024) entries
          |         |       |              (PTEs)
          |---------|-------|
 |CR3|--->   Page Directory
           2^10(1024) entries
```

According to CSAPP, xv6 uses paging harware that support 2-level paging.

We use first top 10 bits to index page directory, and fetch the 20 bits PPN to index the Page table, and use the second 10 bits to index PPN in page table.

An x86 page table is logically an array of 2^20 (1,048,576) page table entries. Thus a page table gives table gives the operating system control over virtual-to-physical address translations at the granularity of aligned chunks of 4096 (2^12) bytes. Such a chunk is called a page.

Each PTE contains **flag bits** that tell the paging hardware how the associated virtual address is allowed to be used:

```
|31                    |12   |8   |6  |5  |4   |3   |2  |1  |0  |
|----------------------|-----|----|---|---|----|----|---|---|---|
| Physical Page Number | AVL |    | D | A | CD | WT | U | W | P |
                          |         |   |   |    |    |    |   |---- P - Present
                          |         |   |   |    |    |    |-------- W - Writable
                          |         |   |   |    |    |------------- U - User
                          |         |   |   |    |----------------- WT - 1=Write-through, 0=Write-back
                          |         |   |   |---------------------- CD - Cache Disabled
                          |         |   |--------------------------- A - Accessed
                          |         |------------------------------- D - Dirty (0 in page directory)
                          |--------------------------------------- AVL - Available for system use

Page table and page directory entries are identical except for the D bits
```

* `PTE_P` indicates whether the PTE is present: if it is not set, a reference to the page causes a fault.
* `PTE_W` controls whether instructions are allows to issue writes to the page; if not set, only reads and instructions fetches are allowed.
* `PTE_U` controls whether the user programs are allowed to use the page; if clear, only kernel is allowed to use the page.

The flags and all other page hardware related structures are defined in `mmu.h`.

# Process address space

 [The `entry.S` provided a primal memory layout for the kernel](https://xiahua.blog/2019/01/29/2019-1-29-paging/). The two entries we created map virtual address `0:0x400000` to physical address `0:0x400000` as well as `KERNBASE:KERNBASE+0x400000` to `0:0x400000`. It is quite elaborately designed because the bootloader can only get access to physical addresses, whereas the kernel only uses virtual addresses. To prevent the `entry.S` program breaking down, we assigned two different virtual address pages to the same physical address page. So the `jmp` need to be an indirect jump.

![Figure 2-2. Layout of the virtual address space of a process and the layout of the physical address space. Note that if a machine has more than 2 Gbyte of physical memory, xv6 can use only the memory that fits between KERNBASE and 0xFE00000.](memory_layout.png)

```c
#define KERNBASE 0x80000000
```

But once we start the process, a more elaborate plan for describing process address space is required. And this turns to be the standard memory layout of the xv6 system. As we can see in the Figure 2-2.

## 0:KERNBASE

This area which addressed up to 2 gigabytes is reserved for the user process. That means one process can use 2 gigabytes of memory for the most. The structure of this area is familiar to the most programmers, including user text, user data, user stack, etc. The user programs use 
them to do tasks on computer.

This area is not mapped to physical memory at first because the entries are dynamically created, which means only when the program needs to use memory, the corresponding entries are created by kernel and linking the virtual pages with vacant physical pages. At first, there is no process being, so there is no entry in this area.

## KERNBASE:0xFFFFFFFF

Above `KERNBASE` sits the kernel. And in this area, entries are all created with a simple mapping rule, which shown in `memlayout.h`:

```c
#define V2P_WO(x) ((x) - KERNBASE)    // same as V2P, but without casts
```

That means, in this area, virtual pages and physical pages are 1:1 mapped except for an offset `KERNBASE`. Why?

The advantage of this is that the kernel can manipulate the physical memory easily. The kernal can see the whole world including all processes' page tables. 

Xv6 does not set the PTE_U flag in the PTEs above KERNBASE, so only the kernel can use them.

Having every process’s page table contain mappings for both user memory and the entire kernel is convenient when switching from user code to kernel code during system calls and interrupts: such switches do not require page table switches. For the most part the kernel does not have its own page table; it is almost always borrowing some process’s page table.

To review, xv6 ensures that each process can use only its own memory. And, each process sees its memory as having contiguous virtual addresses starting at zero, while the process’s physical memory can be non-contiguous. xv6 implements the first by setting the PTE_U bit only on PTEs of virtual addresses that refer to the process’s own memory. It implements the second using the ability of page tables to translate successive virtual addresses to whatever physical pages happen to be allocated to the process.

`PHYSTOP` in the physical memory show the maximum address of the physical address. However, the memory-mapped I/O devices occupy the addresses from `0xFE000000` to `0xFFFFFFFF`. Memory in between `PHYSTOP` and `0xFE000000` does not exist, and cannot be accessed.

# Code comments

>`main` calls `kvmalloc` to create and switch to a page table with the mappings above `KERNBASE` required for the kernel to run.

Do remember the `%esp` pointer is well prepared right a moment ago. In `entry.S` we allocated a per-process kernel stack (4096 bytes) in the kernel. So the next call will store address in the kernel stack, just like an process that just has been trapped in the kernel.

Jump into `kvmalloc`: `vm.c.138:145`

```c
// Allocate one page table for the machine for the kernel address
// space for scheduler processes.
void
kvmalloc(void)
{
  kpgdir = setupkvm();
  switchkvm();
}
```

`kpgdir` is the directory of kernel program pages. It is defined above in `vm.c`.

```c
typedef uint pde_t;
pde_t *kpgdir;  // for use in scheduler()
```

As far we don't know what is a `scheduler`, but we just know `kpgdir` is an entry pointer and  it is global in kernel.

Jump into `setupkvm`, where the most work is done.`vm.c.117:136`

```c
// Set up kernel part of a page table.
pde_t*
setupkvm(void)
{
  pde_t *pgdir;
  struct kmap *k;

  if((pgdir = (pde_t*)kalloc()) == 0)
    return 0;
  memset(pgdir, 0, PGSIZE);
  if (P2V(PHYSTOP) > (void*)DEVSPACE)
    panic("PHYSTOP too high");
  for(k = kmap; k < &kmap[NELEM(kmap)]; k++)
    if(mappages(pgdir, k->virt, k->phys_end - k->phys_start,
                (uint)k->phys_start, k->perm) < 0) {
      freevm(pgdir);
      return 0;
    }
  return pgdir;
}
```

first we allocate a 4096 bytes page in physical memory, and clear it totally.

Then xv6 uses a static `struct kmap` array to represent the layout of the kernel virtual pages: `vm.c.103:115`

```c
// This table defines the kernel's mappings, which are present in
// every process's page table.
static struct kmap {
  void *virt;
  uint phys_start;
  uint phys_end;
  int perm;
} kmap[] = {
 { (void*)KERNBASE, 0,             EXTMEM,    PTE_W}, // I/O space
 { (void*)KERNLINK, V2P(KERNLINK), V2P(data), 0},     // kern text+rodata
 { (void*)data,     V2P(data),     PHYSTOP,   PTE_W}, // kern data+memory
 { (void*)DEVSPACE, DEVSPACE,      0,         PTE_W}, // more devices
};
```

The four entries in `kmap[]` correspond to the 4 areas  above `KERNBASE` in Figure 2.2.

`mappages` installs the translations that are described in `kmap` array in page directory. `vm.c.57:80`

```c
// Create PTEs for virtual addresses starting at va that refer to
// physical addresses starting at pa. va and size might not
// be page-aligned.
static int
mappages(pde_t *pgdir, void *va, uint size, uint pa, int perm)
{
  char *a, *last;
  pte_t *pte;

  a = (char*)PGROUNDDOWN((uint)va);
  last = (char*)PGROUNDDOWN(((uint)va) + size - 1);
  for(;;){
    if((pte = walkpgdir(pgdir, a, 1)) == 0)
      return -1;
    if(*pte & PTE_P)
      panic("remap");
    *pte = pa | perm | PTE_P;
    if(a == last)
      break;
    a += PGSIZE;
    pa += PGSIZE;
  }
  return 0;
}
```

In `mappages`, we use `walkpgdir` function to find the right page of given virtual address (needs to be rounded down to page borders). `vm.c.32:55`

```c
// Return the address of the PTE in page table pgdir
// that corresponds to virtual address va.  If alloc!=0,
// create any required page table pages.
static pte_t *
walkpgdir(pde_t *pgdir, const void *va, int alloc)
{
  pde_t *pde;
  pte_t *pgtab;

  pde = &pgdir[PDX(va)];  // Get first 10 bits of va, and index page directory entry
  if(*pde & PTE_P){ // If the page directory entry present
    pgtab = (pte_t*)P2V(PTE_ADDR(*pde)); // Get the first 20 bits of *pde, and index page table
  } else {
    if(!alloc || (pgtab = (pte_t*)kalloc()) == 0)
      return 0;
    // Make sure all those PTE_P bits are zero.
    memset(pgtab, 0, PGSIZE);
    // The permissions here are overly generous, but they can
    // be further restricted by the permissions in the page table
    // entries, if necessary.
    *pde = V2P(pgtab) | PTE_P | PTE_W | PTE_U;
  }
  return &pgtab[PTX(va)];
}
```

The interesting part is in `walkpgdir`, it mimics the actions of the x86 paging hardware as it looks up the PTE for a virtual address. walkpgdir uses the upper 10 bits of the virtual address to find the page directory entry. If the page directory entry isn’t present, then the required page table page hasn’t yet been allocated; if the `alloc` argument is set, walkpgdir allocates it and puts its physical address in the page directory. Finally it uses the next 10 bits of the virtual address to find the address of the PTE in the page table page.

# Physical memory allocation

We said `kalloc` again and again. When the xv6 works, we need to allocate and free physical memory dynamically.

>The kernel must allocate and free physical memory at run-time for page tables, process user memory, kernel stacks, and pipe buffers.

The way `kalloc` manages memory does not involve with a lot of optimization here. There is a list of all the physical memory pages and `kalloc` keeps track of them.

>There is a bootstrap problem: all of physical memory must be mapped in order for the allocator to initialize the free list, but creating a page table with those mappings involves allocating page-table pages. xv6 solves this problem by using a separate page allocator during entry, which allocates memory just after the end of the kernel’s data segment. This allocator does not support freeing and is limited by the 4 MB mapping in the `entrypgdir`, but that is sufficient to allocate the first kernel page table.

## Code: Physical memory allocator

This part is a little difficult to understand, xv6 use a *free list* to implement allocation, which is the simplest way. `kalloc.c.79:95`

```c
// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.
char*
kalloc(void)
{
  struct run *r;

  if(kmem.use_lock)
    acquire(&kmem.lock);
  r = kmem.freelist;
  if(r)
    kmem.freelist = r->next;
  if(kmem.use_lock)
    release(&kmem.lock);
  return (char*)r;
}
```

Free list strategy does not have the best allocation performance, however it is the simplest way.

>The function main calls `kinit1` and `kinit2` to initialize the allocator. The reason for having two calls is that for much of main one cannot use locks or memory above 4 megabytes. The call to `kinit1` sets up for lock-less allocation in the first 4 megabytes, and the call to kinit2 enables locking and arranges for more memory to be allocatable. main ought to determine how much physical memory is available, but this turns out to be difficult on the x86. Instead it assumes that the machine has 224 megabytes (PHYSTOP) of physical memory, and uses all the memory between the end of the kernel and `PHYSTOP` as the initial pool of free memory. `kinit1` and `kinit2` call `freerange` to add memory to the free list via per-page calls to `kfree`. A PTE can only refer to a physical address that is aligned on a 4096-byte boundary (is a multiple of 4096), so `freerange` uses `PGROUNDUP` to ensure that it frees only aligned physical addresses. The allocator starts with no memory; these calls to `kfree` give it some to manage.

`kalloc.c.26:44`

```c
// Initialization happens in two phases.
// 1. main() calls kinit1() while still using entrypgdir to place just
// the pages mapped by entrypgdir on free list.
// 2. main() calls kinit2() with the rest of the physical pages
// after installing a full page table that maps them on all cores.
void
kinit1(void *vstart, void *vend)
{
  initlock(&kmem.lock, "kmem");
  kmem.use_lock = 0;
  freerange(vstart, vend);
}

void
kinit2(void *vstart, void *vend)
{
  freerange(vstart, vend);
  kmem.use_lock = 1;
}
```

The function `kfree` begins by setting every byte in the memory being freed
to the value 1. This will cause code that uses memory after freeing it (uses ‘‘dangling
references’’) to read garbage instead of the old valid contents; hopefully that will cause
such code to break faster. `kalloc.c.54:77`

```c
//PAGEBREAK: 21
// Free the page of physical memory pointed at by v,
// which normally should have been returned by a
// call to kalloc().  (The exception is when
// initializing the allocator; see kinit above.)
void
kfree(char *v)
{
  struct run *r;

  if((uint)v % PGSIZE || v < end || V2P(v) >= PHYSTOP)
    panic("kfree");

  // Fill with junk to catch dangling refs.
  memset(v, 1, PGSIZE);

  if(kmem.use_lock)
    acquire(&kmem.lock);
  r = (struct run*)v;
  r->next = kmem.freelist;
  kmem.freelist = r;
  if(kmem.use_lock)
    release(&kmem.lock);
}
```

# User address space

![Figure 2-3. Memory layout of a user process with its initial stack.](user.png)

The program text and data segment start at virtual address 0 and end with a guard page. The guard page is not mapped so when the stack runs out, the hardware will throw an exception.

The initial content of stack is created by `exec`.

# Code: sbrk

`sbrk` is the system call for a process to shrink or grow its memory. The system call is implemented by the function `growproc`. If `n` is postive, `growproc` allocates one or more physical pages and maps them at the top of the process’s address space. If `n` is negative, `growproc` unmaps one or more pages from the process’s address space and frees the corresponding physical pages. To make these changes, xv6 modifies the process’s page table. The process’s page table is stored in memory, and so the kernel can update the table with ordinary assignment statements, which is what `allocuvm` and `deallocuvm` do. 

The x86 hardware caches page table entries in a Translation Lookaside Buffer (TLB), and when xv6 changes the page tables, it must invalidate the cached entries. If it didn’t invalidate the cached entries, then at some point later the TLB might use an old mapping, pointing to a physical page that in the mean time has been allocated to another process, and as a result, a process might be able to scribble on some other process’s memory. Xv6 invalidates stale cached entries, by reloading `cr3`, the register that holds the address of the current page table.

# Code: exec

`exec` allocates a new page table with no user mappings with `setupkvm`, allocates memory for each ELF segment with `allocuvm`, and loads each segment into
memory with `loaduvm`. `allocuvm` checks that the virtual addresses requested is
below `KERNBASE`. `loaduvm` uses `walkpgdir` to find the physical address of the allocated memory at which to write each page of the ELF segment, and `readi` to read
from the file.

>During the preparation of the new memory image, if exec detects an error like an invalid program segment, it jumps to the label bad, frees the new image, and returns –1. Exec must wait to free the old image until it is sure that the system call will succeed: if the old image is gone, the system call cannot return –1 to it. The only error cases in exec happen during the creation of the image. Once the image is complete, exec can install the new image (6701) and free the old one (6702). Finally, exec returns 0.

# Real World

The following content come directly from xv6 textbook, which is worth reading.

Like most operating systems, xv6 uses the paging hardware for memory protection and mapping. Most operating systems use x86’s 64-bit paging hardware (which has 3 levels of translation). 64-bit address spaces allow for a less restrictive memory layout than xv6’s; for example, it would be easy to remove xv6’s limit of 2 gigabytes for physical memory. Most operating systems make far more sophisticated use of paging than xv6; for example, xv6 lacks demand paging from disk, copy-on-write fork, shared memory, lazily-allocated pages, and automatically extending stacks. The x86 supports address translation using segmentation (see Appendix B), but xv6 uses segments only for the common trick of implementing per-cpu variables such as proc that are at a fixed address but have different values on different CPUs (see seginit). Implementations of per-CPU (or per-thread) storage on non-segment architectures would dedicate a register to holding a pointer to the per-CPU data area, but the x86 has so few general registers that the extra effort required to use segmentation is worthwhile. 

Xv6 maps the kernel in the address space of each user process but sets it up so that the kernel part of the address space is inaccessible when the processor is in user mode. This setup is convenient because after a process switches from user space to kernel space, the kernel can easily access user memory by reading memory locations directly. It is probably better for security, however, to have a separate page table for the kernel and switch to that page table when entering the kernel from user mode, so that the kernel and user processes are more separated from each other. This design, for example, would help mitigating side-channels that are exposed by the Meltdown vulnerability and that allow a user process to read arbitrary kernel memory.

On machines with lots of memory it might make sense to use the x86’s 4-megabytes ‘‘super pages.’’ Small pages make sense when physical memory is small, to allow allocation and page-out to disk with fine granularity. For example, if a program uses only 8 kilobytes of memory, giving it a 4 megabytes physical page is wasteful. Larger pages make sense on machines with lots of RAM, and may reduce overhead for page-table manipulation. Xv6 uses super pages in one place: the initial page table. The array initialization sets two of the 1024 PDEs, at indices zero and 512 (`KERNBASE>>PDXSHIFT`), leaving the other PDEs zero. Xv6 sets the PTE_PS bit in these two PDEs to mark them as super pages. The kernel also tells the paging hardware to allow super pages by setting the `CR_PSE` bit (Page Size Extension) in `%cr4`.

Xv6 should determine the actual RAM configuration, instead of assuming 224 MB. On the x86, there are at least three common algorithms: the first is to probe the physical address space looking for regions that behave like memory, preserving the values written to them; the second is to read the number of kilobytes of memory out of a known 16-bit location in the PC’s non-volatile RAM; and the third is to look in BIOS memory for a memory layout table left as part of the multiprocessor tables. Reading the memory layout table is complicated.

Memory allocation was a hot topic a long time ago, the basic problems being efficient use of limited memory and preparing for unknown future requests; see Knuth. Today people care more about speed than space-efficiency. In addition, a more elaborate kernel would likely allocate many different sizes of small blocks, rather than (as in xv6) just 4096-byte blocks; a real kernel allocator would need to handle small allocations as well as large ones.

---
title: "2019 2 3 Xv6 First Proces"
date: 2019-02-03 
draft: false
---

There goes the main part of xv6. Xv6 souce code provided a really clear view of each component of the kernel. All parts are shown below: `main.c:14-38`

In this chapter, we will go through the sub-function `userinit`, in which we initialize the memory for a user process, and create the first process.

<!--more-->

```c
// Bootstrap processor starts running C code here.
// Allocate a real stack and switch to it, first
// doing some setup required for memory allocator to work.
int
main(void)
{
  kinit1(end, P2V(4*1024*1024)); // phys page allocator
  kvmalloc();      // kernel page table
  mpinit();        // detect other processors
  lapicinit();     // interrupt controller
  seginit();       // segment descriptors
  picinit();       // disable pic
  ioapicinit();    // another interrupt controller
  consoleinit();   // console hardware
  uartinit();      // serial port
  pinit();         // process table
  tvinit();        // trap vectors
  binit();         // buffer cache
  fileinit();      // file table
  ideinit();       // disk 
  startothers();   // start other processors
  kinit2(P2V(4*1024*1024), P2V(PHYSTOP)); // must come after startothers()
  userinit();      // first user process
  mpmain();        // finish this processor's setup
}
```

`userinit`'s first action is to call `allocproc`. Go to the definition of it, we can see `allocproc` do several things:

* allocating a slot (`struct proc`) in process table. `allocproc` scans the proc table for a slot with state `UNUSED`. `proc.c:69-83`

```c
// Look in the process table for an UNUSED proc.
// If found, change state to EMBRYO and initialize
// state required to run in the kernel.
// Otherwise return 0.
static struct proc*
allocproc(void)
{
  struct proc *p;
  char *sp;

  acquire(&ptable.lock);

  for(p = ptable.proc; p < &ptable.proc[NPROC]; p++)
    if(p->state == UNUSED)
      goto found;
```

* initializing the parts of process's state required for its kernel thread to execute. It sets the state to `EMBRYO` to mark it as used and give the process a unique pid. `proc.c:88-92`

```c
found:
  p->state = EMBRYO;
  p->pid = nextpid++;

  release(&ptable.lock);
```

* Next, it allocate a kernel stack for the process's kernel thread. If failed, labeled the `struct proc` with `UNUSED`, return 0. `proc.c:94-99` (kernel thread is the thread when a process is trapped in kernel).

```c
 // Allocate kernel stack.
  if((p->kstack = kalloc()) == 0){
    p->state = UNUSED;
    return 0;
  }
  sp = p->kstack + KSTACKSIZE;
```

Here we meet a new function called `kalloc`. Go to its definition, `kalloc.c:79-95`

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

`kalloc` is used to allocate one 4096-byte page physical memory. We can infer from the above usage that `kalloc` return a pointer that can point to a stack space.

In `allocproc`, we assign `sp` at the top of the kernel stack. 

![Figure 1-4. A new kernel stack](kernel_stack.png)

Next `allocproc` use `sp` to form the stack like the picture above, and assign the various pointer like `p->tf` `p->context`.

What is trapframe and context? Well, trapframe is used for the code to return from the kernel back to the process's user code. All the registers and the porgram counter value (in `%eip`) are freezed before entering kernel and stored in trapframe through trap entry code. After exiting, all the registers are restored and the processor executes from the freezed program counter value. Then exit process is in `trapret` function. `trapasm.S:23-32`

The trapframe structure is defined in `x86.h:148-183`. 

```c
// Layout of the trap frame built on the stack by the
// hardware and by trapasm.S, and passed to trap().
struct trapframe {
  // registers as pushed by pusha
  uint edi;
  uint esi;
  uint ebp;
  uint oesp;      // useless & ignored
  uint ebx;
  uint edx;
  uint ecx;
  uint eax;

  // rest of trap frame
  ushort gs;
  ushort padding1;
  ushort fs;
  ushort padding2;
  ushort es;
  ushort padding3;
  ushort ds;
  ushort padding4;
  uint trapno;

  // below here defined by x86 hardware
  uint err;
  uint eip;
  ushort cs;
  ushort padding5;
  uint eflags;

  // below here only when crossing rings, such as from user to kernel
  uint esp;
  ushort ss;
  ushort padding6;
};
```

The trapframe is built by hardware and by `trapasm.S`: `trapasm.S:5-21`

```x86asm
alltraps:
  # Build trap frame.
  pushl %ds
  pushl %es
  pushl %fs
  pushl %gs
  pushal
  
  # Set up data segments.
  movw $(SEG_KDATA<<3), %ax
  movw %ax, %ds
  movw %ax, %es

  # Call trap(tf), where tf=%esp
  pushl %esp
  call trap
  addl $4, %esp
```

`trapret` is also defined in `trapasm.S:23-32`:

```x86asm
  # Return falls through to trapret...
.globl trapret
trapret:
  popal
  popl %gs
  popl %fs
  popl %es
  popl %ds
  addl $0x8, %esp  # trapno and errcode
  iret
```

The context works similar with trapframe, but it is used by context control system. The kernel switches processor from a process to another, so we also need to store the context frame into kernel stack for switching.

The `allocproc` function is called not only in `userinit`, but also every time when a new process is forked. So it is delicately designed for both two conditions. It sets the context frame's `%eip` value right at `forkret`, so when the processor switches to this process, `forkret` will be executed. Unlike `trapret`, `forkret` only does a simple thing, returning to whatever on the bottom of the kernel stack. At that time, the context frame has already poped, what at the stack bottom is the `trapret` entrance. So the processor assumes the process just return from kernel, `trapret` then restores the trapframe. 

However when the process was first allocated, we will deliberately set trapframe's `%eip` to 0x0, so when `trapret` returns, the user code will be executed at 0x0. `proc.c:101-106`

```c
  // Leave room for trap frame.
  sp -= sizeof *p->tf;
  p->tf = (struct trapframe*)sp;

  // Set up new context to start executing at forkret,
  // which returns to trapret.
  sp -= 4;
  *(uint*)sp = (uint)trapret;

  sp -= sizeof *p->context;
  p->context = (struct context*)sp;
  memset(p->context, 0, sizeof *p->context);
  p->context->eip = (uint)forkret;

  return p;
}
```

When the `allocproc` finish, we called `setupkvm` to prepare the memory page for our first process code, and manipulate the trapframe to let it return right to the beginning of the user code. (`allocproc` only modified the value of context and `forkret` part) We will discuss `setupkvm` in the near future, but at high level it creates a address space as the Figure 1-2.

![Figure 1-2. Layout of a virtual address space](address_space.png)

The initial contents of the first process’s user-space memory are the compiled form of `initcode.S`; as part of the kernel build process, the linker embeds that binary in the kernel and defines two special symbols, `_binary_initcode_start` and `_binary_initcode_size`, indicating the location and size of the binary. Userinit copies that binary into the new process’s memory by calling inituvm, which allocates one page of physical memory, maps virtual address zero to that memory, and copies the binary to that page.

Then userinit sets up the trap frame with the initial user mode state: the %cs register contains a segment selector for the `SEG_UCODE` segment running at privilege level `DPL_USER` (i.e., user mode rather than kernel mode), and similarly `%ds`, `%es`, and `%ss` use `SEG_UDATA` with privilege `DPL_USER`. The `%eflags` `FL_IF` bit is set to allow hardware interrupts; we will re-examine this in Chapter 3.

The stack pointer `%esp` is set to the process’s largest valid virtual address, `p->sz`. The instruction pointer is set to the entry point for the initcode, address 0. The function userinit sets `p->name` to initcode mainly for debugging. Setting `p->cwd` sets the process’s current working directory; we will examine namei in detail in Chapter 6. 

Once the process is initialized, userinit marks it available for scheduling by setting `p->state` to `RUNNABLE`.


# Lab3：Syscall及启动分析

## 代码分析
在ucore/kern/process/pstruct.h文件中定义了进程的数据结构，可以看到其中包含了指向父进程的结构体指针parent。
```
struct proc_struct {
  enum proc_state state;                        // Process state
  int pid;                                      // Process ID
  int runs;                                     // the running times of Proces
  uintptr_t kstack;                             // Process kernel stack
  bool need_resched;                            // bool value: need to be rescheduled to release CPU?
  struct proc_struct *parent;                   // the parent process
  struct mm_struct *mm;                         // Process's memory management field
  struct context context;                                  // Switch here to run process, save sp
  struct trapframe *tf;                         // Trap frame for current interrupt
  uintptr_t cr3;                                // CR3 register: the base addr of Page Directroy Table(PDT)
  uint32_t flags;                               // Process flag
  char name[PROC_NAME_LEN + 1];                 // Process name
  list_entry_t list_link;                       // Process link list
  list_entry_t hash_link;                       // Process hash list
  int exit_code;                                // exit code (be sent to parent proc)
  uint32_t wait_state;                          // waiting state
  struct proc_struct *cptr, *yptr, *optr;       // relations between processes
};
```

syscall调用源码文件为ucore/kern/syscall/syscall.h，在其中添加一个getppid函数，
```
int sys_getppid(uint32_t arg[]) {
  return current->parent->pid;
}
```

并设置

```
int syscall() {
  struct trapframe *tf = current->tf;
  uint32_t arg[2];
  int num = tf->tf_regs.a;
  arg[0] = tf->tf_regs.b;
  arg[1] = tf->tf_regs.c;
  switch (num) {

  case SYS_getppid:
    return sys_getppid(arg);

  }
```

同时，在ucore/lib/u.h文件中增加SYS_getppid的宏定义
```
/* syscall number */
#define SYS_exit            1
#define SYS_fork            2
#define SYS_wait            3
#define SYS_exec            4
#define SYS_clone           5
#define SYS_yield           10
#define SYS_sleep           11
#define SYS_kill            12
#define SYS_gettime         17
#define SYS_getpid          18
#define SYS_brk             19
#define SYS_mmap            20
#define SYS_munmap          21
#define SYS_shmem           22
#define SYS_getpid          23
#define SYS_putc            30
#define SYS_pgdir           31

```



## 建议
在浏览代码是顺便看到内核初始化函数ucore/kern/main.c
```
void kern_init() {

  // cons_init();                // init the console

  printf("(THU.CST) os is loading ...\n");

  pmm_init();                   // init physical memory management

  idt_init();                   // init interrupt descriptor table

  asm (STI);                    // enable irq interrupt

  spage(1);

  vmm_init();                   // init virtual memory management

  swap_init();                  // init swap

  proc_init();

  stmr(128*1024);          // init clock interrupt

  cpu_idle();

  // while (1) {
  //                             // do nothing
  // }
}
```
可以看到，依次初始化内存、idt表、irq等。这里可以改为函数指针的结构体，如UBoot的代码中因支持的硬件类型多，初始化过程繁杂，使用函数指针结构体能使代码的模块化更好。
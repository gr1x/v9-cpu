# Lab3：Syscall及启动分析

## 代码分析
在ucore/kern/process/pstruct.h文件中定义了进程的数据结构，可以看到其中包含了指向父进程的结构体指针parent。
```
struct proc_struct {
  enum proc_state state;                        
  int pid;                                      // 进程ID
  int runs;                                     
  uintptr_t kstack;                             
  bool need_resched;                            
  struct proc_struct *parent;                   // 父进程结构体指针
  struct mm_struct *mm;                         
  struct context context;
  struct trapframe *tf;
  uintptr_t cr3;                                
  uint32_t flags;                              
  char name[PROC_NAME_LEN + 1];                 
  list_entry_t list_link;                       
  list_entry_t hash_link;                       
  int exit_code;                                
  uint32_t wait_state;                          
  struct proc_struct *cptr, *yptr, *optr;       
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

  case SYS_getppid:    //增加调用功能号
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
#define SYS_getppid         23   //增加功能号宏定义
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
可以看到，依次初始化内存、idt表、irq等。这里可以改为函数指针的结构体，如UBoot的代码中因支持的硬件类型多，初始化过程繁杂，使用函数指针结构体能使代码的模块化更好，第三方开发人员也能在初始化流程中插入特定代码逻辑而无需重新编译内核。

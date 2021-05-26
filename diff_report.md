# Task 1: UID and GID and PPID

## Makefile
```diff
diff --git a/xv6-pdx/Makefile b/xv6-pdx/Makefile
index acba3d9..1ad40a8 100644
--- a/xv6-pdx/Makefile
+++ b/xv6-pdx/Makefile
@@ -1,6 +1,6 @@
 # Set flag to correct CS333 project number: 1, 2, ...
 # 0 == original xv6-pdx distribution functionality
-CS333_PROJECT ?= 1
+CS333_PROJECT ?= 2
 PRINT_SYSCALLS ?= 0
 CS333_CFLAGS ?= -DPDX_XV6
 ifeq ($(CS333_CFLAGS), -DPDX_XV6)
@@ -18,7 +18,7 @@ endif

 ifeq ($(CS333_PROJECT), 2)
 CS333_CFLAGS += -DCS333_P1 -DUSE_BUILTINS -DCS333_P2
-CS333_UPROGS += _date _time _ps
+CS333_UPROGS += #_date _time _ps
 CS333_TPROGS += _testsetuid  _testuidgid _p2-test
 endif
```

## proc.c
```diff
diff --git a/xv6-pdx/proc.c b/xv6-pdx/proc.c
index 25e0211..452e9da 100644
--- a/xv6-pdx/proc.c
+++ b/xv6-pdx/proc.c
@@ -6,6 +6,9 @@
 #include "x86.h"
 #include "proc.h"
 #include "spinlock.h"
+#ifdef CS333_P3
+#include "pdx.h"
+#endif

@@ -189,6 +192,10 @@ userinit(void)
   acquire(&ptable.lock);
   p->state = RUNNABLE;
   release(&ptable.lock);
+  #ifdef CS333_P2
+  p->uid = DEFAULT_UID; 
+  p->gid = DEFAULT_GID;
+  #endif
 }

@@ -250,6 +257,8 @@ fork(void)
   safestrcpy(np->name, curproc->name, sizeof(curproc->name));
 
   pid = np->pid;
+ np->uid = curproc->uid;
+ np->gid = curproc->gid;
```

## syscall.c
```diff
diff --git a/xv6-pdx/syscall.c b/xv6-pdx/syscall.c
index 5d3d225..7d5853e 100644
--- a/xv6-pdx/syscall.c
+++ b/xv6-pdx/syscall.c
@@ -109,6 +109,13 @@ extern int sys_halt(void);
 #ifdef CS333_P1
 extern int sys_date(void);
 #endif // PDX_XV6
+#ifdef CS333_P2
+extern int sys_getuid(void);
+extern int sys_getgid(void);
+extern int sys_getppid(void);
+extern int sys_setuid(void);
+extern int sys_setgid(void);
+#endif

@@ -138,6 +145,13 @@ static int (*syscalls[])(void) = {
 #ifdef CS333_P1
 [SYS_date]    sys_date,
 #endif // PDX_XV6
+#ifdef CS333_P2
+[SYS_getuid]  sys_getuid,
+[SYS_getgid]  sys_getgid,
+[SYS_getppid]  sys_getppid,
+[SYS_setuid]  sys_setuid,
+[SYS_setgid]  sys_setgid,
+#endif

@@ -169,6 +183,13 @@ static char *syscallnames[] = {
 #ifdef CS333_P1
   [SYS_date]    "date",
 #endif // PDX_XV6
+#ifdef CS333_P2
+[SYS_getuid]  "getuid",
+[SYS_getgid]  "getgid",
+[SYS_getppid]  "getppid",
+[SYS_setuid]  "setuid",
+[SYS_setgid]  "setgid",
+#endif
```

## test.c
```diff
diff --git a/xv6-pdx/p2-test.c b/xv6-pdx/p2-test.c
index 6f1ce25..4db4c0c 100755
--- a/xv6-pdx/p2-test.c
+++ b/xv6-pdx/p2-test.c
@@ -9,9 +9,9 @@
 // comment out tests for features the student doesn't have implemented
 // Note the CPUTIME_TEST requires GETPROCS_TEST
 #define UIDGIDPPID_TEST
-#define CPUTIME_TEST
-#define GETPROCS_TEST
-#define TIME_TEST
+//#define CPUTIME_TEST
+//#define GETPROCS_TEST
+//#define TIME_TEST
```

## proc.h
```diff
diff --git a/xv6-pdx/proc.h b/xv6-pdx/proc.h
index c7ee129..a9c6f2f 100644
--- a/xv6-pdx/proc.h
+++ b/xv6-pdx/proc.h
@@ -41,6 +41,8 @@ struct proc {
   char *kstack;                // Bottom of kernel stack for this process
   enum procstate state;        // Process state
   uint pid;                    // Process ID
+  uint uid;
+  uint gid;
   struct proc *parent;         // Parent process. NULL indicates no parent
   struct trapframe *tf;        // Trap frame for current syscall
   struct context *context;     // swtch() here to run process
```

## syscall.h
```diff
diff --git a/xv6-pdx/syscall.h b/xv6-pdx/syscall.h
index 14f3e17..6bfacca 100644
--- a/xv6-pdx/syscall.h
+++ b/xv6-pdx/syscall.h
@@ -23,3 +23,8 @@
 #define SYS_halt    SYS_close+1
 // student system calls begin here. Follow the existing pattern.
 #define SYS_date    SYS_halt+1
+#define SYS_getuid  SYS_date+1
+#define SYS_getgid  SYS_getuid+1
+#define SYS_getppid  SYS_getgid+1
+#define SYS_setuid  SYS_getppid+1
+#define SYS_setgid  SYS_setuid+1
```
## sysproc.c
```diff
diff --git a/xv6-pdx/sysproc.c b/xv6-pdx/sysproc.c
index 40b676e..dabf47c 100644
--- a/xv6-pdx/sysproc.c
+++ b/xv6-pdx/sysproc.c
@@ -6,6 +6,9 @@
 #include "memlayout.h"
 #include "mmu.h"
 #include "proc.h"
+#ifdef CS333_P2
+#include "uproc.h"
+#endif

@@ -112,3 +112,43 @@ sys_date(void)
 }
 #endif
 
+#ifdef CS333_P2
+
+int
+sys_getuid(void){
+  return myproc()->uid;
+}
+
+int
+sys_getgid(void){
+  return myproc()->gid;
+}
+
+int
+sys_getppid(void){
+  if(myproc()->pid != 1)
+    {return myproc()->parent->pid;}
+  return myproc()->pid;
+}
+
+int
+sys_setuid(void){
+  int var;
+
+  if (argint(0,&var) < 0 || var > 32767 || var < 0)
+    return -1;
+  myproc()->uid = (uint)var;
+  return 0;
+}
+
+int
+sys_setgid(void){
+  int var;
+
+  if (argint(0,&var) < 0 || var > 32767 || var < 0)
+    return -1;
+  myproc()->gid = (uint)var;
+  return 0;
+}
+
+#endif
```

## user.h 
```diff
diff --git a/xv6-pdx/user.h b/xv6-pdx/user.h
index e0231eb..1a3c5d2 100644
--- a/xv6-pdx/user.h
+++ b/xv6-pdx/user.h
@@ -29,6 +29,15 @@ int halt(void);
 int date(struct rtcdate*);
 #endif
 
+#ifdef CS333_P2
+uint getuid(void); // UID of the current process
+uint getgid(void); // GID of the current process
+uint getppid(void); // PID of the parent process
+
+int setuid(uint); // Set uid
+int setgid(uint); // set gid
+#endif
+
```

## usys.S
```diff
diff --git a/xv6-pdx/usys.S b/xv6-pdx/usys.S
index 84bd80b..090de9a 100644
--- a/xv6-pdx/usys.S
+++ b/xv6-pdx/usys.S
@@ -31,3 +31,8 @@ SYSCALL(sleep)
 SYSCALL(uptime)
 SYSCALL(halt)
 SYSCALL(date)
+SYSCALL(getuid)
+SYSCALL(getgid)
+SYSCALL(getppid)
+SYSCALL(setuid)
+SYSCALL(setgid)
```

# Task 2: Process Execution Time

## proc.c
```diff
diff --git a/xv6-pdx/proc.c b/xv6-pdx/proc.c
index 452e9da..04f98ee 100644
--- a/xv6-pdx/proc.c
+++ b/xv6-pdx/proc.c
@@ -5,6 +5,7 @@
 #include "mmu.h"
 #include "x86.h"
 #include "proc.h"
+#include "uproc.h"
 #include "spinlock.h"
 #ifdef CS333_P3
 #include "pdx.h"

@@ -155,6 +156,10 @@ allocproc(void)
   // initialized start_ticks as ticks global variabel
   p->start_ticks = ticks;
 
+  // Initialized cpu_ticks as 0 so not NULL
+  p->cpu_ticks_in = 0;
+  p->cpu_ticks_total = 0;
+

@@ -401,6 +406,7 @@ scheduler(void)
       c->proc = p;
       switchuvm(p);
       p->state = RUNNING;
+      p->cpu_ticks_in = ticks;
       swtch(&(c->scheduler), p->context);
       switchkvm();
 
@@ -441,6 +447,7 @@ sched(void)
   if(readeflags()&FL_IF)
     panic("sched interruptible");
   intena = mycpu()->intena;
+  p->cpu_ticks_total += ticks - (p->cpu_ticks_in);
   swtch(&p->context, mycpu()->scheduler);
   mycpu()->intena = intena;
 }
```

## proc.h
```diff
diff --git a/xv6-pdx/proc.h b/xv6-pdx/proc.h
index a9c6f2f..08c51d5 100644
--- a/xv6-pdx/proc.h
+++ b/xv6-pdx/proc.h
@@ -52,6 +52,8 @@ struct proc {
   struct inode *cwd;           // Current directory
   char name[16];               // Process name (debugging)
   uint start_ticks;
+  uint cpu_ticks_total;        // total elapsed ticks in cpu
+  uint cpu_ticks_in;          // ticks when scheduled
# time commad
```

# Task 3: The "ps" command

## defs.h
```diff
diff --git a/xv6-pdx/defs.h b/xv6-pdx/defs.h
index f85557d..ec0e390 100644
--- a/xv6-pdx/defs.h
+++ b/xv6-pdx/defs.h
@@ -4,6 +4,7 @@ struct file;
 struct inode;
 struct pipe;
 struct proc;
+struct uproc;
 struct rtcdate;
 struct spinlock;
 struct sleeplock;

@@ -124,6 +125,9 @@ void            userinit(void);
 int             wait(void);
 void            wakeup(void*);
 void            yield(void);
+#ifdef CS333_P2
+int             copiedProc(int, struct uproc*);
+#endif // CS333_p2
 #ifdef CS333_P3
 void            printFreeList(void);
 void            printList(int);
```

## ps.c
```diff
diff --git a/xv6-pdx/ps.c b/xv6-pdx/ps.c
new file mode 100644
index 0000000..592296f
--- /dev/null
+++ b/xv6-pdx/ps.c
@@ -0,0 +1,68 @@
+#ifdef CS333_P2
+
+#define MAX_ENTRIES 16
+
+#include "types.h"
+#include "user.h"
+#include "uproc.h"
+
+int
+main(int argc, char *argv[])
+{
+  struct uproc* temp;
+
+  temp = malloc(sizeof(struct uproc) * MAX_ENTRIES);
+
+  if (getprocs(MAX_ENTRIES, temp) < 0) {
+    printf(2,"Error: getprocs call failed. %s at line %d\n",
+        __FILE__, __LINE__);
+    exit();
+  }
+
+  printf(1, "\nPID\tName\t\tUID\tGID\tPPID\tElapsed\tCPU\tState\tSize\n");
+
+  for (int i = 0; i < MAX_ENTRIES; i++){
+    if (temp[i].pid == NULL)
+      break;
+    printf(1, "%d\t%s\t\t%d\t%d\t%d\t", 
+      temp[i].pid,
+      temp[i].name,
+      temp[i].uid,
+      temp[i].gid,
+      temp[i].ppid
+    );
+
+    // Special case for floating point
+    // Elapsed Ticks
+    if (temp[i].elapsed_ticks < 10){
+      printf(1, "0.00%d\t", temp[i].elapsed_ticks);
+    } else if (temp[i].elapsed_ticks < 100){
+      printf(1, "0.0%d\t", temp[i].elapsed_ticks);
+    } else if (temp[i].elapsed_ticks < 1000){
+      printf(1, "0.%d\t", temp[i].elapsed_ticks);
+    }else{
+      printf(1, "%d.%d\t", temp[i].elapsed_ticks/1000, temp[i].elapsed_ticks%1000);
+    }
+
+    // CPU Total Ticks
+    if (temp[i].CPU_total_ticks < 10){
+      printf(1, "0.00%d", temp[i].CPU_total_ticks);
+    } else if (temp[i].CPU_total_ticks < 100){
+      printf(1, "0.0%d", temp[i].CPU_total_ticks);
+    } else if (temp[i].CPU_total_ticks < 1000){
+      printf(1, "0.%d", temp[i].CPU_total_ticks);
+    }else{
+      printf(1, "%d.%d", temp[i].CPU_total_ticks/1000, temp[i].CPU_total_ticks%1000);
+    }
+
+    printf(1, "\t%s\t%d\n",
+      temp[i].state,
+      temp[i].size
+    );
+  }
+
+  printf(1, "\n");
+  exit();
+}
+
+#endif // CS333_P2
```

## syscall.c
```diff
diff --git a/xv6-pdx/syscall.c b/xv6-pdx/syscall.c
index 7d5853e..460d719 100644
--- a/xv6-pdx/syscall.c
+++ b/xv6-pdx/syscall.c
@@ -115,6 +115,7 @@ extern int sys_getgid(void);
 extern int sys_getppid(void);
 extern int sys_setuid(void);
 extern int sys_setgid(void);
+extern int sys_getprocs(void);
 #endif
 
 static int (*syscalls[])(void) = {
@@ -151,6 +152,7 @@ static int (*syscalls[])(void) = {
 [SYS_getppid]  sys_getppid,
 [SYS_setuid]  sys_setuid,
 [SYS_setgid]  sys_setgid,
+[SYS_getprocs]  sys_getprocs,
 #endif
 };
 
@@ -189,6 +191,7 @@ static char *syscallnames[] = {
 [SYS_getppid]  "getppid",
 [SYS_setuid]  "setuid",
 [SYS_setgid]  "setgid",
+[SYS_getprocs]  "getprocs",
```

## syscall.h
```diff
diff --git a/xv6-pdx/syscall.h b/xv6-pdx/syscall.h
index 6bfacca..9602739 100644
--- a/xv6-pdx/syscall.h
+++ b/xv6-pdx/syscall.h
@@ -28,3 +28,4 @@
 #define SYS_getppid  SYS_getgid+1
 #define SYS_setuid  SYS_getppid+1
 #define SYS_setgid  SYS_setuid+1
+#define SYS_getprocs SYS_getgid+1
```

## user.h
```diff
diff --git a/xv6-pdx/user.h b/xv6-pdx/user.h
index 1a3c5d2..a826be0 100644
--- a/xv6-pdx/user.h
+++ b/xv6-pdx/user.h
@@ -36,6 +36,7 @@ uint getppid(void); // PID of the parent process
 
 int setuid(uint); // Set uid
 int setgid(uint); // set gid
+int  getprocs(uint, struct uproc*);
 #endif
 ```

 ## usys.S
 ```diff
diff --git a/xv6-pdx/usys.S b/xv6-pdx/usys.S
index 090de9a..6ee47b8 100644
--- a/xv6-pdx/usys.S
+++ b/xv6-pdx/usys.S
@@ -36,3 +36,4 @@ SYSCALL(getgid)
 SYSCALL(getppid)
 SYSCALL(setuid)
 SYSCALL(setgid)
+SYSCALL(getprocs)
```

## proc.c
```diff
diff --git a/xv6-pdx/proc.c b/xv6-pdx/proc.c
index 452e9da..04f98ee 100644
--- a/xv6-pdx/proc.c
+++ b/xv6-pdx/proc.c
@@ -5,6 +5,7 @@
 #include "mmu.h"
 #include "x86.h"
 #include "proc.h"
+#include "uproc.h"
 #include "spinlock.h"
 #ifdef CS333_P3
 #include "pdx.h"
 
@@ -637,6 +678,48 @@ procdump(void)
 #endif // CS333_P1
 }
 
+
+#ifdef CS333_P2
+// Helper function to access ptable for sys_getprocs
+int
+copiedProc(int max, struct uproc* up)
+{
+  int count = 0;
+  struct proc* procTest;
+  acquire(&ptable.lock);
+
+  for(procTest = ptable.proc; procTest < &ptable.proc[NPROC]; procTest++){
+    if (count == max) {
+       break;
+    }
+      
+    if (procTest->state == UNUSED || procTest->state == EMBRYO){
+      continue;
+    } else if (procTest->state == SLEEPING || procTest->state == RUNNABLE || procTest->state == RUNNING || procTest->state == ZOMBIE) {
+      up[count].pid = procTest->pid;
+      up[count].uid = procTest->uid;
+      up[count].gid = procTest->gid;
+
+      // Handle init PPID
+      if (procTest->pid == 1){
+        up[count].ppid = procTest->pid;
+      }else{
+        up[count].ppid = procTest->parent->pid;
+      }
+
+      up[count].elapsed_ticks = ticks - procTest->start_ticks;
+      up[count].CPU_total_ticks = procTest->cpu_ticks_total;
+      safestrcpy(up[count].state, states[procTest->state], sizeof(up[count].state));
+      up[count].size = procTest->sz;
+      safestrcpy(up[count].name, (char*)procTest->name, sizeof(procTest->name));
+  
+      count++;
+    }
+  }
+
+  release(&ptable.lock);
+  return count;
+}
+#endif // CS333_P2
+
```

## Task 4: The time command

## runoff.list
```diff
diff --git a/xv6-pdx/runoff.list b/xv6-pdx/runoff.list
index 81930d9..0b89d10 100644
--- a/xv6-pdx/runoff.list
+++ b/xv6-pdx/runoff.list
@@ -85,6 +85,8 @@ halt.c
 uptime.c
 date.h
 date.c
+ps.c
+time.c
 uproc.h
 testuidgid.c
 testuidgid.txt
 ```

## time.c
```diff
diff --git a/xv6-pdx/time.c b/xv6-pdx/time.c
new file mode 100644
index 0000000..e053569
--- /dev/null
+++ b/xv6-pdx/time.c
@@ -0,0 +1,43 @@
+#include "types.h"
+#include "user.h"
+
+int
+main(int argc, char *argv[])
+{
+  int curr_time = uptime();
+
+  if (argc == 1){
+      int temp_left = 0;
+      int now = uptime() - curr_time;
+      if(now > 1000){
+        temp_left = now % 1000;
+        now /= 1000;
+    }
+      // saya capek pak
+      if(temp_left != 0){
+        printf(1, "(null) ran in %d.%d seconds\n", now, temp_left);
+    } else{
+        printf(1, "(null) ran in 0.%d seconds\n", now);
+     }
+    } else {
+        if (fork() == 0){
+            exec(argv[1], &argv[1]);
+        }else{
+            wait();
+             int temp_left = 0;
+             int now = uptime() - curr_time;
+            
+            if(now > 1000){
+            temp_left = now % 1000;
+            now /= 1000;
+            }
+            if(temp_left != 0){
+            printf(1, "%s ran in %d.%d seconds\n", argv[1], now, temp_left);
+            } else{
+            printf(1, "%s ran in 0.%d seconds\n", argv[1], now);
+            }
+        }
+    }
+
+  exit();
+}
```


## Task 5: modifying console 

## proc.c
```diff
diff --git a/xv6-pdx/proc.c b/xv6-pdx/proc.c
index 452e9da..04f98ee 100644
--- a/xv6-pdx/proc.c
+++ b/xv6-pdx/proc.c
@@ -5,6 +5,7 @@
 #include "mmu.h"
 #include "x86.h"
 #include "proc.h"
+#include "uproc.h"
 #include "spinlock.h"
 #ifdef CS333_P3
 #include "pdx.h"

@@ -568,8 +575,42 @@ kill(int pid)
 void
 procdumpP2P3P4(struct proc *p, char *state_string)
 {
-  cprintf("TODO for Project 2, delete this line and implement procdumpP2P3P4() in proc.c to print a row\n");
-  return;
+  cprintf("%d\t%s\t\t%d\t%d\t%d\t",
+    p->pid,
+    p->name,
+    p->uid,
+    p->gid,
+    p->pid == 1 ? p->pid : p->parent->pid
+  );
+
+  // Elapsed ticks
+  if ((ticks - (p->start_ticks)) < 10){
+    cprintf("0.00%d\t",(ticks - (p->start_ticks)) );
+  } else if((ticks - (p->start_ticks)) < 100) {
+    cprintf("0.0%d\t",(ticks - (p->start_ticks)) );
+  } else if((ticks - (p->start_ticks)) < 1000) {
+    cprintf("0.%d\t",(ticks - (p->start_ticks)) );
+  } else{
+    cprintf("%d.%d\t",(ticks - (p->start_ticks))/1000,(ticks - (p->start_ticks))%1000);
+  }
+
+  // CPU ticks
+  if (p->cpu_ticks_total < 10){
+    cprintf("0.00%d", p->cpu_ticks_total );
+  } else if(p->cpu_ticks_total < 100) {
+    cprintf("0.0%d",p->cpu_ticks_total );
+  } else if(p->cpu_ticks_total < 1000) {
+    cprintf("0.%d",p->cpu_ticks_total );
+  } else{
+    cprintf("%d.%d",p->cpu_ticks_total/1000,p->cpu_ticks_total%1000);
+  }
+
+  cprintf("\t%s\t%d\t",
+    states[p->state],
+    p->sz
+  );
+return;
+
```

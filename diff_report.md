# TASK 1: System Call Tracing
## Makefile
```diff
diff --git a/Makefile b/Makefile
old mode 100644
new mode 100755
index 6483959..6cd4fb9
--- a/Makefile
+++ b/Makefile
@@ -1,7 +1,7 @@
 # Set flag to correct CS333 project number: 1, 2, ...
 # 0 == original xv6-pdx distribution functionality   
-CS333_PROJECT ?= 0
-PRINT_SYSCALLS ?= 0
+CS333_PROJECT ?= 1
+PRINT_SYSCALLS ?= 1
 CS333_CFLAGS ?= -DPDX_XV6
 ifeq ($(CS333_CFLAGS), -DPDX_XV6)
 CS333_UPROGS +=        _halt _uptime
```
## syscall.c
```diff
diff --git a/syscall.c b/syscall.c
old mode 100644
new mode 100755
index 9105b52..0fe3633
--- a/syscall.c
+++ b/syscall.c
@@ -106,75 +102,91 @@ extern int sys_uptime(void);
 {
   int num;
   struct proc *curproc = myproc();

   num = curproc->tf->eax;
-  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
+  if (num > 0 && num < NELEM(syscalls) && syscalls[num])
+  {
     curproc->tf->eax = syscalls[num]();
-  } else {
+#ifdef PRINT_SYSCALLS
+cprintf("%s->%d\n", syscallnames[num], curproc->tf->eax);
+#endif
+  }
+  else
+  {
     cprintf("%d %s: unknown sys call %d\n",
             curproc->pid, curproc->name, num);
     curproc->tf->eax = -1;
   }
 }
+cprintf("%s->%d\n", syscallnames[num], curproc->tf->eax);
+#endif
+  }
+  else
+  {
     cprintf("%d %s: unknown sys call %d\n",
             curproc->pid, curproc->name, num);
     curproc->tf->eax = -1;
   }
 }
 ```

# TASK 2: Date System Call
## Makefile
```diff
diff --git a/Makefile b/Makefile
old mode 100644
new mode 100755
index 6483959..6cd4fb9
--- a/Makefile
+++ b/Makefile
@@ -13,7 +13,7 @@ endif

 ifeq ($(CS333_PROJECT), 1)
 CS333_CFLAGS += -DCS333_P1
-CS333_UPROGS += #_date
+CS333_UPROGS += _date
 endif

 ifeq ($(CS333_PROJECT), 2)
```
## user.h
```diff
diff --git a/user.h b/user.h
old mode 100644
new mode 100755
index 31d9134..cc496dd
--- a/user.h
+++ b/user.h
@@ -43,3 +43,7 @@ int atoi(const char*);
 int atoo(const char*);
 int strncmp(const char*, const char*, uint);
 #endif // PDX_XV6
+
+#ifdef CS333_P1
+int date(struct rtcdate*);
+#endif //CS333_P1
```
## usys.S
```diff
diff --git a/usys.S b/usys.S   
old mode 100644
new mode 100755
index 0d4eaed..84bd80b
--- a/usys.S
+++ b/usys.S
@@ -30,3 +30,4 @@ SYSCALL(sbrk)
 SYSCALL(sleep)
 SYSCALL(uptime)
 SYSCALL(halt)
+SYSCALL(date)
```
## syscall.c
```diff
@@ -106,75 +102,91 @@ extern int sys_uptime(void);
 #ifdef PDX_XV6
 extern int sys_halt(void);
 #endif // PDX_XV6
+#ifdef CS333_P1
+// internally, the function prototype must be 'int' not 'uint' for sys_date()
+extern int sys_date(void);
+#endif // CS333_P1

 static int (*syscalls[])(void) = {
-[SYS_fork]    sys_fork,
-[SYS_exit]    sys_exit,
-[SYS_wait]    sys_wait,
-[SYS_pipe]    sys_pipe,
-[SYS_read]    sys_read,
-[SYS_kill]    sys_kill,
-[SYS_exec]    sys_exec,
-[SYS_fstat]   sys_fstat,
-[SYS_chdir]   sys_chdir,
-[SYS_dup]     sys_dup,
-[SYS_getpid]  sys_getpid,
-[SYS_sbrk]    sys_sbrk,
-[SYS_sleep]   sys_sleep,
-[SYS_uptime]  sys_uptime,
-[SYS_open]    sys_open,
-[SYS_write]   sys_write,
-[SYS_mknod]   sys_mknod,
-[SYS_unlink]  sys_unlink,
-[SYS_link]    sys_link,
-[SYS_mkdir]   sys_mkdir,
-[SYS_close]   sys_close,
+    [SYS_fork] sys_fork,
+    [SYS_exit] sys_exit,
+    [SYS_wait] sys_wait,
+    [SYS_pipe] sys_pipe,
+    [SYS_read] sys_read,
+    [SYS_kill] sys_kill,
+    [SYS_exec] sys_exec,
+    [SYS_fstat] sys_fstat,
+    [SYS_chdir] sys_chdir,
+    [SYS_dup] sys_dup,
+    [SYS_getpid] sys_getpid,
+    [SYS_sbrk] sys_sbrk,
+    [SYS_sleep] sys_sleep,
+    [SYS_uptime] sys_uptime,
+    [SYS_open] sys_open,
+    [SYS_write] sys_write,
+    [SYS_mknod] sys_mknod,
+    [SYS_unlink] sys_unlink,
+    [SYS_link] sys_link,
+    [SYS_mkdir] sys_mkdir,
+    [SYS_close] sys_close,
 #ifdef PDX_XV6
-[SYS_halt]    sys_halt,
+    [SYS_halt] sys_halt,
 #endif // PDX_XV6
+#ifdef CS333_P1
+    [SYS_date] sys_date,
+#endif
 };

 #ifdef PRINT_SYSCALLS
 static char *syscallnames[] = {
-  [SYS_fork]    "fork",
-  [SYS_exit]    "exit",
-  [SYS_wait]    "wait",
-  [SYS_pipe]    "pipe",
-  [SYS_read]    "read",
-  [SYS_kill]    "kill",
-  [SYS_exec]    "exec",
-  [SYS_fstat]   "fstat",
-  [SYS_chdir]   "chdir",
-  [SYS_dup]     "dup",
-  [SYS_getpid]  "getpid",
-  [SYS_sbrk]    "sbrk",
-  [SYS_sleep]   "sleep",
-  [SYS_uptime]  "uptime",
-  [SYS_open]    "open",
-  [SYS_write]   "write",
-  [SYS_mknod]   "mknod",
-  [SYS_unlink]  "unlink",
-  [SYS_link]    "link",
-  [SYS_mkdir]   "mkdir",
-  [SYS_close]   "close",
+    [SYS_fork] "fork",
+    [SYS_exit] "exit",
+    [SYS_wait] "wait",
+    [SYS_pipe] "pipe",
+    [SYS_read] "read",
+    [SYS_kill] "kill",
+    [SYS_exec] "exec",
+    [SYS_fstat] "fstat",
+    [SYS_chdir] "chdir",
+    [SYS_dup] "dup",
+    [SYS_getpid] "getpid",
+    [SYS_sbrk] "sbrk",
+    [SYS_sleep] "sleep",
+    [SYS_uptime] "uptime",
+    [SYS_open] "open",
+    [SYS_write] "write",
+    [SYS_mknod] "mknod",
+    [SYS_unlink] "unlink",
+    [SYS_link] "link",
+    [SYS_mkdir] "mkdir",
+    [SYS_close] "close",
 #ifdef PDX_XV6
-  [SYS_halt]    "halt",
+    [SYS_halt] "halt",
 #endif // PDX_XV6
+#ifdef CS333_P1
+[SYS_date] "date",
+#endif
 };
 #endif // PRINT_SYSCALLS
 ```
 ## syscall.h
 ```diff
 diff --git a/syscall.h b/syscall.h
old mode 100644
new mode 100755
index 7fc8ce1..88f26e9
--- a/syscall.h
+++ b/syscall.h
@@ -22,3 +22,4 @@
 #define SYS_close   SYS_mkdir+1
:...skipping...
diff --git a/syscall.h b/syscall.h
old mode 100644
new mode 100755
index 7fc8ce1..88f26e9
--- a/syscall.h
+++ b/syscall.h
@@ -22,3 +22,4 @@
 #define SYS_close   SYS_mkdir+1
 #define SYS_halt    SYS_close+1
 // student system calls begin here. Follow the existing pattern.
+#define SYS_date    SYS_halt+1
```
## sysproc.c
```diff
diff --git a/sysproc.c b/sysproc.c
old mode 100644
new mode 100755
index 98563ea..620c547
--- a/sysproc.c
+++ b/sysproc.c
@@ -97,3 +97,16 @@ sys_halt(void)
   return 0;
 }
:...skipping...
diff --git a/sysproc.c b/sysproc.c
old mode 100644
new mode 100755
index 98563ea..620c547
--- a/sysproc.c
+++ b/sysproc.c
@@ -97,3 +97,16 @@ sys_halt(void) 
   return 0;
 }
 #endif // PDX_XV6
+
+#ifdef CS333_P1
+int 
+sys_date(void) {
+  struct rtcdate *d;
+
+  if (argptr(0, (void*)&d, sizeof(struct rtcdate)) < 0)
+  return -1; 
+
+  cmostime(d);
+  return 0;
+}
+#endif
```

# TASK 3: Process Information
## proc.h
```diff
diff --git a/proc.h b/proc.h
old mode 100644
new mode 100755
index 0a0b4c5..c7ee129
--- a/proc.h
+++ b/proc.h
@@ -49,6 +49,7 @@ struct proc {
   struct file *ofile[NOFILE];  // Open files
:...skipping...
diff --git a/proc.h b/proc.h
old mode 100644
new mode 100755
index 0a0b4c5..c7ee129
--- a/proc.h
+++ b/proc.h
@@ -49,6 +49,7 @@ struct proc {
   struct file *ofile[NOFILE];  // Open files
   struct inode *cwd;           // Current directory
   char name[16];               // Process name (debugging)      
+  uint start_ticks;
 };

 // Process memory is laid out contiguously, low addresses first:
 ```
 ## proc.c
 ```diff
 diff --git a/proc.c b/proc.c
old mode 100644
new mode 100755
index d030537..65d88f6
--- a/proc.c
+++ b/proc.c
@@ -137,25 +143,24 @@ allocproc(void)
 
   // Leave room for trap frame.
   sp -= sizeof *p->tf;
-  p->tf = (struct trapframe*)sp;
+  p->tf = (struct trapframe *)sp;
 
   // Set up new context to start executing at forkret,
   // which returns to trapret.
   sp -= 4;
-  *(uint*)sp = (uint)trapret;
+  *(uint *)sp = (uint)trapret;
 
   sp -= sizeof *p->context;
-  p->context = (struct context*)sp;
+  p->context = (struct context *)sp;
   memset(p->context, 0, sizeof *p->context);
   p->context->eip = (uint)forkret;
-
+  p->start_ticks = ticks;
   return p;
 }
@@ -553,23 +570,22 @@ kill(int pid)
 // No lock to avoid wedging a stuck machine further.

 #if defined(CS333_P2)
-void
-procdumpP2P3P4(struct proc *p, char *state_string)
+void procdumpP2P3P4(struct proc *p, char *state_string)
 {
   cprintf("TODO for Project 2, delete this line and implement procdumpP2P3P4() in proc.c to print a row\n");
   return;
 }
 #elif defined(CS333_P1)
-void
-procdumpP1(struct proc *p, char *state_string)
+void procdumpP1(struct proc *p, char *state_string)
 {
-  cprintf("TODO for Project 1, delete this line and implement procdumpP1() in proc.c to print a row\n");
+  int current = ticks - p->start_ticks;
+
+  cprintf("%d\t%s\t     %d.%d\t%s\t%d\t", p->pid, p->name, current / 1000, current % 1000, states[p->state], p->sz);
   return;
 }
 #endif
```

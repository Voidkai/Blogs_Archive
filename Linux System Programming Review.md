# Linux System Programming Review

# Chapter one 

## 1.What's Operating system

**Special layer of software that provides appilcation softwares access to hardware resources**

- Convenient abstraction of complex hardware devices
- Protected access to shared resources
- Security and authentication
- Communication amongst logical entities

**In conclusion**

+ OS provide a virtual machine abstraction to handle diverse hardware.
+ OS coordinate resources and protect users from each other.
+ OS simplify application development by providing standard services.
+ OS provide a array of fault containments, fault tolerance and fault recovery.

## standard input/output and error(the fd)



|    standard     | file descriptor |
| :-------------: | :-------------: |
| standard input  |        0        |
| standard output |        1        |
| standard error  |        2        |



## 2.System call and C Library

### system call

**All implementations provide a number of entry points directly to system kernel called system calls.**

**increases a middle layer between user space process and hardware device**, there are two function of this layer:

- providing an abstract layer of hardware for user space.
- system call ensure the reliable and security of system.

### type of system call

- process control type
+ - process creation
  - setting or accessing the process's attribute and so on
- file operate type
+ - create modify and delete file
- device management type
+ - open, close and manipulate device
- communication type
- - creating the connection between processes
  - send/receive message, or other communication modes
- Information maintenance type
  - tansferrring the message between application program and OS.



### the relationship between System call and library functions

- Each system call **has a function of the same name** in the standard c library. And user process calls the function by using the standard C calling sequence instead of system calls. 
- There is **not an one to one matching relationship** between system call and library function.
- system calls usually **provide a minimal interface** , whereas library functions often provide more elaborate functionality.
- An application can either make a system call or call a library routie.
- **Not all library function will invoke the system call**.


## 3.The stdin, stout, stderr redirection; the redirection without overwriting
In the Linux, the stdin ,stout, stderr of command can be redirected to other files by adopting the file redirection fuction.
### stdin redirection
expressed by a symbol "<"

- syntax 1; command &lt; input-file

- syntax 2; command O &lt; input-file  
  (the input of command comes from the input-file instead of keyboard)  
  Example: grep John &lt; tempfile  

  

### stout redirection
expressed by a symbol "&gt;"  

- syntax 1: command &gt; output-file
- syntax 2: command 1 &gt; output-file  
(let the output of command send to 'output-file' file instead of monitor  
Example: cat lab1 &gt; newfile
### stderr redirection
Stderr redirection is expressed by a symbol
"2&gt;"(adopting the symbol "&gt;" to connect the standard error and file descriptor);  

- syntax 1: command 2&gt; error-file  
  Example: $ ls -l foo 2&gt; error.log

### without overwriting

withour overwriting is expressed by a symbol ">>"

## 4.Interpret the meaning if each field in the ls -l result

|      Field       |                           Meaning                            |
| :--------------: | :----------------------------------------------------------: |
|    -rw-r--r--    | type of file and Access right to the file(owner, group,others) |
|        1         |                 File's number of hard links                  |
|       root       |               The username of the file's owner               |
|       root       |           The name of the group that own this file           |
|      32059       |                  size of the file in bytes                   |
| 2017-xx-xx xx:xx |        Date and time of the file's last modification         |
|    oo-cd.pdf     |                    the name of the files                     |



# Chapter two 

## 1.Virtual File System(VFS)

VFS is the key point for fulfilling these characteristics:

- differrent file systems can **exist together** in linux
- the file operation can be **executed between multi-file system**

**VFS is a software layer , that provides an abstartction function for the kernel** , to realize the two characteristics.

In order to support different file systems, **VFS defines basic, abstract interface and data structure** expected by the VFS.

### I-node

**data structure used to represent a filesystem object(file or directory), storing the metadata , including the manipulation metadate, owner or permission metadata, and other file relative attributes.**

## 2.open() function

### the return value

if the function is executed successfully,it will **return the file descriptor**.otherwise, it **return -1 and modified errno**.

### equal function

**open(path, O_WRONLY | O_CREAT | O_TRUNC, mode);**

## 3.create() function & read() function & write() function

### Create function

Head file: fcntl.h

int creat(const char *pathname, mode_t mode);

create is that the file is opened only for writing.

before new version of open was provided, if we were creatig a temporary file that we want to write and then read back , we had to call create, close and then open.

### Read function

Head file: unistd.h

ssize_t read(int filedes, void *buf, size_t nbytes);

the read operation starts at the file's current offset. before a successful return, the offset is incremented by **the number of bytes actually read**.

### Write function

Head file: unistd.h

ssize_t write(int fd, const void *buf, size_t nbytes);

The return value is usually equal to the nbytes argument;otherwise, an error has occurred.

## 4.stat structures

|        argument        |             meaning              |
| :--------------------: | :------------------------------: |
|     mode_t st_mode     |   file type & mode(permission)   |
|      ino_t st_ino      |   i-node number(serial number)   |
|      dev_t st_dev      |    device number(file system)    |
|    nlink_t st_nlink    |         number of links          |
|      uid_t st_uid      |         user ID of owner         |
|      gid_t st_gid      |        group ID of owner         |
|     off_t st_size      | size in bytes, for regular files |
| struct timespec st_atm |       time of last access        |
| struct timespec st_mtm |    time of last modification     |
| struct timespec st_ctm | time of last file status change  |
|  blksize_t st_blksize  |       best I/O block size        |
|   blkcnt_t st_blocks   | number of disk blocks allocated  |



## 5.File sharing

### one or different processes shares one or more files

The Linux system supports the sharing of open files among different processes.The kernel uses **three data structures** to represent an open file, and the relationship among them determine the effect one process has an another with regard to file sharing.

- Every process has an entry in the process table. Within each process table entry is **a table of open file descriptors**, each file descriptors entry points to a pointer to the file table entry.
- The kernel maintains **a file table** for all open files. Each file table entry contains
  - The file status flags for the files, such as read, write , append, sync, and non-blocking
  - The current file offset
  - A pointer to the i-node table entry for the file.
- Each open file (or device ) has an **i-node** that contains information about the type of file and data.

### when to share the same file offset

when **different processes or different file descriptors of one process** share the same file table entry. thefile offsets are the same which are equal to the file offset in the file table entry.

## 6.fopen() function

it is used to open the file associated with the steam.

| Restriction                       |  R   |  W   |  A   |  R+  |  W+  |  A+  |
| --------------------------------- | :--: | :--: | :--: | :--: | :--: | :--: |
| file must alread exist            |  1   |      |      |  1   |      |      |
| previous content of file discard  |      |  1   |      |      |  1   |      |
| stream can be read                |  1   |      |      |  1   |  1   |  1   |
| stream can be write               |      |  1   |  1   |  1   |  1   |  1   |
| stream can be written only at end |      |      |  1   |      |      |  1   |



## 7.File access permission management

| st_mode mask |    Meaning    |
| :----------: | :-----------: |
|   S_IRUSR    |   user_read   |
|   S_IWUSR    |  user_write   |
|   S_IXUSR    | user_execute  |
|   S_IRGRP    |  group_read   |
|   S_IWGRP    |  group_write  |
|   S_IXGRP    | group_execute |
|   S_IROTH    |  other_read   |
|   S_IWOTH    |  other_write  |
|   S_IXOTH    | other_execute |

### three types of permission for file 

|                            Action                            | Permission  |
| :----------------------------------------------------------: | :---------: |
|    open any type of file by name<br>search the directory     |  D(all).X   |
|                      read the directory                      |     D.R     |
|                   open a file for reading                    |     F.R     |
| open a existing file for writing<br>specify the O_TRUNC flag in open() |     F.W     |
| create a new file in a directory<br>delete an existing file  | D.W and D.X |
|   execute the file(regular) using the seven exec functions   |     F.X     |

# Chapter three

## 1.Running a new process

Executing a new program: One system call(exec family of calls) loads a binary program into memory, replacing the previous contents of the address space, and begins execution of the new program.

Create a new process: fork call is used to create a new process. Often the new process immediately executes a new program, which is called forking.

Execute a new program in a new process: first a fork to create a new process, and then an exec to load a new binary into that process.

### The Exec Family of calls

- int execl(const char *path, const char *arg, ...)
- int execlp(const char *file, const char *arg, ...)
- int execle(const char *path, const char *arg, ..., char * const envp[])
- int execv(const char *path, const char argv[])
- int execvp(const char *file, const char arg[])
- int execve(const char *filename, const char arg[], char * const envp[])

### Which one is the system call:

execve()

### fork()

parent: return child's pid

child: return 0;

error: return -1;

## 2.Terminating a process

1. void exit(int status): a standard function for terminationg the current process. a call to function exit() preform some basic shutdown steps to instructs the kernel to terminate the process. for example exit(EXIT_SUCCESS).
2. simply "falling off the end" of the program. when the main() function returns , the approach invokes a system call(exit) after its own shutdown code.
3. sent a signal whose default action can also terminate a process. such signal includes SIGTERM and SIGKILL.
4. by incurring the wrath of the kernel.

## 3.the definition of zombie

**When a child died before its parent, the kernel should put the child into a special process state which is knowed as a zombie.**

A process that has terminated but has not yet been waited upon by its parent is called a "zombie".

## 4.the steps of becoming a daemon

1. call fork(), the new process will become the daemon.
2. In the parent, call exit(). to realized that its child terminated, and the daemon's parents is no longer running, and that the daemon is not a process group leader.
3. call setsid(), giving the daemon a new process group and session, both of which have it as leader.
4. Change the working directory to the root directory via chdir(),
5. Close all file descriptor. because  it doesn't want to inherit open file descriptors.
6. open  file descriptor 0, 1 and 2 and redirect them to */dev/null*

```c
int main(void)
{
    pid_t pid;
    int i;
    /*create new process*/
    pid = fork();
    if(pid == -1){
        return -1;
    }else if(pid!=0)
        exit(EXIT_SUCCESS);
    /* create new session and process group*/
    if(setsid() == -1)
        return -1;
    /*set the working directory to the root directory*/
    if(chdir("/") == -1)
        return -1;
    /*close all open files--NR_OPEN is overkill, but works*/
    for (i=0;i<NR_OPEN; i++)
        close(i);
    /*redirect fd's 0 1 and 2 to /dev/null*/
    open("/dev/null", O_RDWR); /*stdin*/
    dup(0); /*stdout*/
    dup(0); /*stderror*/
    /* do its daemon thing... */
    return 0;
}
```



# Chapter four

## 1.Latency, Jitter and their difference

### Latency

**the period from the occurence of stimulus until the execution of the response.**And if the latency is less than or equal to the operational deadline, the system can work correctly.

### Jitter

**the variation in timing between responses. or successive events.**

### Difference

it is difficute to mearsure the accurate latency, so many attempts at instrumenting not latency but jitter.

## 2.linux scheduling policies and priority

### FIFO policy(SCHED_FIFO)

simple real-time policy without timeslices. A FIFO-calssed process will continue running so long as no higher-priority process becomes runnable.

### round-robin policy(SCHED_RR)

scheduler assign each process a timeslice. when a process exhausts its timeslice, the scheduler moves it to the end of the list of processes at its priority.

## 3.Hard real-time system and soft real-time system

- **hard real-time system: absolute adherence  to the operational deadline, and Exceeding the deadlines will constitute failure and is a major bug.**
- **soft real-time system: does not consider overrrunning the operational deadline to be a critical failure.**

the division between the soft and hard real-time system is indepentent of the size of the operational deadlines.

## 4.Multitasking: cooperative and preemptive

cooperative multitasking: **a process not stop running until it voluntarily decides to do so.**

preemptive multitasking(linux system): **the scheduler decides when one process is to stop running and a different process is to resume.** the act of suspending a runnning process instead of another one process is preemption.

## 5. the range of legal nice value

legal nicee values range from **-20 to 19** inclusive , with a default value of 0.

The lower a process's nice value. the higher the process's priority and the lager its timeslice.

# Chapter five

## 1.Threads

Virtual memory affords each process a unique view of memory taht seamlessly maps back to physical RAM or ondisk storage and **virtual memory is associated with the process**. all of the threads in a given process share that memory.

**A virtualized processor is associated with threads.**

## 2.Multithreading

### Primary Benefits to Multithread;

- programming abstraction
  - dividing up a work and assign each divisions to a thread, is a natual approach to many problems.
- Parallelism
  - threads provide an efficient way to acheive the true parallelism
- Inproving responsiveness
  - with multithreaading, a long-running operation may be delegated to worker threads, allowing at least one thread to remain responsive to user input and perform UI operations.
- Blocking I/O
  - In a multithreaded process, individual threads may block, waiting for I/O, while other threads are still running.
- Context switch
  - the cost of context switch between threads in a process is cheaper than process-to-processs context switch
- Memory savings 
  - threads provides an efficient way to share memory yet utilize mutiple units of execution.

## 3.Threading models

### 1:1 threading(kernel level threading)

**The kernel provides native support for threads, and each of those thread translate directly to the user-space concept of a thread.** one-to-one relationship between what the kernel provides and what the user consumes.

### N:1 threading(user level threading)

**In this model user space is the key to the system's threading support, as it implements the concept of a thread.A process with N threads will map to a single kernel process-hence N:1.**

### Hybrid threading

true parallelism of kernel level threading with the free context switch of user level threading,

**the kernel provides a native thread concept, while user space also implements user threads. User space, perheps in conjunction with the kernel , then decides how to map N user threads onto M kernel threads, whereN>=M.**

## 4.Concurrrency, Parallelism and Races

### Concurrency and Parallelism (difinition and difference)

#### Definition

Concurrency: the  ability of two or more threads to execute in **overlopping time periods**.

Parallelism: the ability of two or more threads to execute **at the same time**.

#### Difference

1. Concurrency can occur without parallelism, and parallelism is specific form of concurrency requiring multiple processors.
2. Concurrency is a **programming pattern**, a way of approching problems, but Parallelism is a **hardware feature**, acheivable through concurrency.

### race condition and how to prevent the race condition

**Race condition : a situation in which the unsynchronized access of a sharing resource by two or more threads leads to error program behavior.**

races are eliminated by synchronizing threads' access to the critical regions.

## 5.Synchronization

### Critical region(definition);

a window during which correct program behavior requires that threads do not interleave execution.

### mutexes(what's and how to use)

mutexes is the techniques for making critical regions atomic, from solutions for single instructions up through large blocks of code.

### lock(definition)

a mechanism for ensuring mutual exclusion within a critical region, rendering it atomic.

### deadlock(definition)

A deadlock is a situation in which two or more threading are waiting for the other to finish, and thus neither does.

In the case of mutexs, the two threads are waiting for a different mutex , that another thread is holding

## 6.Pthreads

### pthread mutexes(function)

Initilizing mutexes:Mutexes are represented by the pthread_mutex_t object. it is meant to be an opaque structure provided to the various mutex interfaces. Although you can dynamically create mutexes, most uses are static.

```c
/*define the initialize a mutex name 'mutex' */
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
```

that is all we have to do to start using it.

### locking mutexes(function)

int pthread_mutex_lock(pthread_mutex_t *mutex);

A successful call to pthread_mutex_lock() will block the calling thread until the mutex pointed at by mutex becomes available. Once available, the calling thread will wake up and this function will return zero. If the mutex is available on invocation, the function will return immediately.

### unlocking mutexes (function)

The counterpart to locking is unlocking, or releasing the mutex.

int pthread_mutex_unlock (pthread_mutex_t *mutex);

A successful call to pthread_mutex_unlock() releases the mutex pointed at by mutex and returns zero. The call does not block; the mutex is released immediately.

# Chapter six

## 1.Signal concepts

### signal

**Signals are software interrupts that provide a mechanism for handling asynchronous events. As a primitive form of interprocess communication (IPC), one process can also send a signal to another process.**

### the life cycle of signal (3 step)

1. First, a signal is raised.
2. The kernel then stores the signal until it is able to deliver it. 
3. Finally, once it is free to do so, the kernel handles the signal as appropriate.

### three actions performed by the signal

1. Ignore the signal:
   - No action is taken. There are two signals that cannot be ignored: SIGKILL and SIGSTOP. The system administrator needs to be able to kill or stop processes.
2. Catch and handle the signal:
   - The kernel will suspend execution of the process’s current code path and jump to a previously registered function. The process will then execute this function. Once the process returns from this function, it will jump back to wherever it was when it caught the signal. SIGINT and SIGTERM are two commonly caught signals. Processes catch SIGINT to handle the user generating the interrupt character. Processes catch SIGTERM to perform necessary cleanup.
3. Perform the default action:
   - This action depends on the signal being sent. The default action is often to terminate the process.

### signal identifiers;(the signal number)

**The signal numbers start at 1 (generally SIGHUP) and proceed linearly upward. There are about 31 signals in total.** There is no signal with the value 0, which is a special value known as the null signal. some system calls (such as kill()) use a value of 0 as a special case.

## 2.Signal basic management

### signal()

The simplest and oldest interface for signal management is the signal() function.

sighandler_t signal(int signo, sighandler_t handler);

except **SIGKILL** and **SIGSTOP**

aruguments:

remove the current action taken on receipt of the signal ***signo*** and instead handles the signal with the signal handler specified by ***handler***.

**Signo** is  one of the signal names.

**handler** function must return void, the function takes one argument, the signal identifier of the signal being handled. this allows a single function to handle mutiple signals.

### pause() to waiting for a signal

pause() puts a process to sleep until it receives a signal that either is handled or terminates the process

## 3.Advanced Signal management

### Sigaction()

int sigaction(int signo, const struct sigaction *act, struct sigaction *oldact);

#### arguments:

**signo**: the signal identified , which can be any value except those associated with SIGKILL and SIGSTOP.

**act**: if not null, changes the current behavior of the signal as specified by act.

**oldact**:if not null, the call stores the previous(or current, if null) behavior of the given signal there.

#### what will happen after calling it(success or error)

success: return 0, error: return -1.s 

# Experiment & Programming

## 1.how to implement the "ls -l " command

```c
int print_perm(mode_t st_mode){
    int i;
    unsigned int mask = 0x7;
    static char *perm[] = {"---","--x","-w-","-wx","r--","r-x","rw-","rwx"};
    for(i=3;i>0;i--){
        printf("%3s",perm[(st_mode >> (i-1)*3) & mask]);
    }
    printf(" ");
    return 0;
}

void print_ugname(const struct stat *currentstat)
{
    struct passwd *p_passwd;/*根据用户id获取用户名*/
    p_passwd = getpwuid(currentstat->st_uid);
    if(p_passwd != NULL)
        printf("%-10s ",p_passwd->pw_name);
    else
        printf("%-10d ",currentstat->st_uid);

    struct group * p_group;/*根据组id获取组名*/
    p_group = getgrgid(currentstat->st_gid);
    if(p_group != NULL)
        printf("%-10s ",p_group->gr_name);
    else
        printf("%-10d ",currentstat->st_gid);

}
void print_time(const struct stat *currentstat)
{
    struct tm * chtm = localtime(&(currentstat->st_mtime));
    if(chtm == NULL){
        printf("localtime is error");
        exit(EXIT_FAILURE);
    }
    else
        printf("%-2d月%2d  ",chtm->tm_mon+1,chtm->tm_mday);/*tm_mon属于[0,11]*/

    if(chtm->tm_hour < 10)/*0~9的数要写成0X格式*/
        printf("0%d:",chtm->tm_hour);
    else{printf("%d:",chtm->tm_hour);}
    if(chtm->tm_min < 10)
        printf("0%d  ",chtm->tm_min);
    else{printf("%2d  ",chtm->tm_min);};
}

void list_message(const char * filename,const struct stat *currentstat)/*处理读取到的文件*/
{
    print_type(currentstat->st_mode);/*判断并打印文件类型*/
    //print_perm(get_message);
    print_perm(currentstat->st_mode);/*判断并打印文件权限*/
    printf("%-3d ",currentstat->st_nlink);/*打印硬链接数*/
    print_ugname(currentstat);/*用户id与组id转换成用户名与组名并打印*/
    printf("%-3d\t",currentstat->st_size);/*打印所占空间文件大小*/
    print_time(currentstat);/*将GMT时间的秒数转换成标准时间格式输出*/
    printf("%s\n",filename);/*输出文件名*/
}
```



## 2.how to implement the "cp -r" command

```c
int Dirtodir(char* source, char* dest){
    DIR *currentdir = NULL;
    struct dirent currentdp = NULL;
    struct stat currentstat;
    char sourcebuf[500];
    char destbuf[500];
    int ret = 0;
    
    currentdir = opendir(source);
    while(currentdp = readdir(currentdir)){
        if(strcmp(currentdp->d_name, ".")&& strcmp(currentdp->d_name,"..")){
            sprintf(sourcebuf, "%s/%s",source, currentdp->d_name);
            memset(currentstat, 0, sizeof(currentstat));
            if(stat(sourcebuf, &currentstat) == -1){
                printf("error");
                return -1
            }
            if(S_IDIR(currentstat.st_mode)){
                sprintf(destbuf,"%s/%s", dest,currentdp->d_name);
                if(access(destbuf, F_OK)== -1){
                    mkdir(destbuf, currentstat.st_mode);
                }
                ret = DirToDir(sourcebuf, destbuf);
            }else{
                sprintf(destbuf, "%s/%s", dest, currentdp->d_name);
                ret = diletofile(sourcebuf,destbuf, &currentstat);
            }
        }
    }
    
    return ret;
}
```



```c
int Filetofile(char *source, char *dest, struct stat *pstat){
    int sourcefd=0;
    int destfd=0;
    char buf[4096];
    int readcount = 4096;
    int writecount = 0;
    int datacount = 0;
    
    sourcefd = open(source, O_RDONLY);
    datacount = (int)pstat->st_size;
    destfd = open(dest, O_WROLY|O_CREAT|);
    
    while(datacount != 0){
        if(datacount < readcount){
            readcount = datacount;
        }
        writecount = read(sourcefd, buf, readcount);
        if(writecount == readcount){
            write(destdef, buf, writecount);
            datacount -= writecount;
        }else{
            print("read error");
            return -1;
        }
    }
}
```



## 3.parent process runs the ls -l; child cp -r;

```c
pid_t pid;
pid = fork();

if(pid<0){
    printf("fork error\n");
    return -1;
}else if(pid == 0){
    cp
}else if(pid>0){
    while(pid ！= waitpid(pid, NULL, 0));
}
```
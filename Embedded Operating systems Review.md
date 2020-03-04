# Embedded Operating systems Review
## The First：Task management

### How to describe a task?
1.A task is a **excutable software entity** specialized to perform a special action (Enemy Detect, Danger Evaluation, Missile Launching...), which includes a sequential code, and relative data and computer resource.

2.A task is an **infinite loop function**. it looks just like any other C function containing a return type and an argument, but it must **never return**. The return type of a task must always be declared to be ***void***

#### What's the difference between task and process？

    int i=1;
    test(){
        sleep(10s); //等待main线程执行完i++
        printf("%d", i);
    }
    
    int main(){
        create_task(test,............);
        i++;
    }

《Embedded Operating System》P20:if create_task is the API of creating thread,the result of test() is 2;if it is the API of creating process, the result is 1.(多线程时，进程之间不能相互访问对方的变量)

### How to create a task in uc/OS II?



#### OsTaskCreate

    INT8U OStaskCreate(void(*task)(void *p_arg), void *p_arg, OS_STK *ptos,INT8U prio //prio由工程规划决定一个任务的优先级){
        OS_STK *psp;
        INT8U err;
        #if OS_CRITICAL_METHOD == 3    //第三种方式进入临界区
        OS_CPU_SR cpu_sr = 0;
        #endif
            OS_ENTER_CRITICAL();        //关中断
            if(OSIntNesting >0){        //判断是否发生中断嵌套
                OS_EXIT_CRITICAL();     //开中断
                return (OS_ERR_TASK_CREATE_ISR);    返回错误
            }
        if(OSTCBPrioTbl[prio] == (OS_TCB *)0){  //判断任务优先级是否被占用
            OSTCBPrioTbl[prio] = (OS_TCB *)1;
            OS_EXIT_CRITICAL();                 //关中断
        }
    
        //任务堆栈初始化
        psp = (OS_STK *)OSTaskStkInit(task, pdata, ptos,0);
        //TCB初始化        
        err = OS_TCBInit(prio, psp, (OS_STK *)0, 0, 0, (void *)0, 0);
        if(err==OS_NO_ERR){
            OS_ENTER_CRITICAL();
            OSTaskCtr++;
            OS_EXIT_CRITICAL();
            if(OSRuning == TRUE){
                OS_Sched();         //进入调度点
            }
            else{
                OS_ENTER_CRITICAL();
                OSTCBPrioTbl[prio] = (OS_TCB *)0;   //TCB初始化错误释放占用优先级
                OS_EXIT_CRITICAL();
            }
            return (err);
        }
        OS_EXIT_CRITICAL();
        return (OS_PRIO_EXIST);
    }

#### OSTaskStkInit

    OS_STK *OSTaskStkInit (void(*task)(void *pd),void *p_arg,OS_STK *ptos,INT16U opt){
        OS_STK *stk;                                                         
        opt    = opt; 
        stk    = ptos;                                  
        *(stk)  = (OS_STK)task;       
        /* Entry Point*/                               
        *(--stk) = (INT32U)0;   
        /*LR */                                     
        *(--stk) = (INT32U)0;
        /*R12*/                                     
        *(--stk) = (INT32U)0;           /*R11*/
        *(--stk) = (INT32U)0;           /*R10*/
        *(--stk) = (INT32U)0;           /*R9*/
        *(--stk) = (INT32U)0;           /*R8*/
        *(--stk) = (INT32U)0;           /*R7*/
        *(--stk) = (INT32U)0;           /*R6*/
        *(--stk) = (INT32U)0;           /*R5*/
        *(--stk) = (INT32U)0;           /*R4*/    
        *(--stk) = (INT32U)0;           /*R3*/
        *(--stk) = (INT32U)0;           /*R2*/
        *(--stk) = (INT32U)0;           /*R1*/
        *(--stk) = (INT32U)p_arg;       
        *(--stk) = (INT32U)0x 0x00000013L;                                
        return (stk);
    }

#### OS_TCBInit

    INT8U  OS_TCBInit (INT8U prio, OS_STK *ptos, OS_STK *pbos, INT16U id, INT32U stk_size, void *pext, INT16U opt){
        #if OS_CRITICAL_METHOD == 3                               
            OS_CPU_SR  cpu_sr;
        #endif    
      
        OS_TCB    *ptcb;
        OS_ENTER_CRITICAL();
        ptcb = OSTCBFreeList;   
                             
        if (ptcb != (OS_TCB *)0) {
            OSTCBFreeList = ptcb->OSTCBNext;          
            OS_EXIT_CRITICAL();
            ptcb->OSTCBStkPtr= ptos;                      
            ptcb->OSTCBPrio = (INT8U)prio;                
            ptcb->OSTCBStat = OS_STAT_RDY;               
            ptcb->OSTCBDly = 0;                        
            pext = pext;                      
            stk_size = stk_size;
            pbos = pbos;
            opt = opt;
            id = id;
           //Priority Bitmap
            ptcb->OSTCBY = prio >> 3;                             
            ptcb->OSTCBBitY= OSMapTbl[ptcb->OSTCBY];
            ptcb->OSTCBX =  prio & 0x07;
            ptcb->OSTCBBitX=  OSMapTbl[ptcb->OSTCBX];
            OS_ENTER_CRITICAL();
            OSTCBPrioTbl[prio] = ptcb;
            ptcb->OSTCBNext = OSTCBList;                  
            ptcb->OSTCBPrev = (OS_TCB *)0;
            if (OSTCBList != (OS_TCB *)0) {
                OSTCBList->OSTCBPrev = ptcb;
            }
            OSTCBList = ptcb;
            OSRdyGrp |= ptcb->OSTCBBitY;        
            OSRdyTbl[ptcb->OSTCBY] |= ptcb->OSTCBBitX;
            OS_EXIT_CRITICAL();
            return (OS_NO_ERR);
        }
        OS_EXIT_CRITICAL();
        return (OS_NO_MORE_TCB);
    }


### How to schedule tasks in the ready queue?
#### OS_Sched
    void OS_Sched(void){
        #if OS_CRITICAL_METHOD == 3
            OS_CPU_SR cpu_sr = 0;
        #endif
    
        OS_ENTER_CRITICAL();
        if(OSIntNesting == 0){
            if(OSLockNesting == 0){
                OS_SchedNew();     //函数内部构造见下一标题           
                if(OSPrioHighRdy != OSPrioCur){
                    OSTCBHighRdy = OSTCBPrioTbl[OSPrioHighRdy];
                    #if OS_TASK_PROFILE_EN >0
                        OSTCBHighRdy-> OSTCBCtxSwCtr++;
                    #endif
                        OSCtxSwCtr++;
                        OS_TASK_SW();
                }
            }
        }
        OS_EXIT_CRITICAL();
    }

####OS_SchedNew(以下涉及优先级位图法)

    #if OS_LOWEST_PRIO <= 63
        INT8U y; //采用优先级位图法获取最高优先级任务
        y= OSUnMapTbl[OSRdyGrp];
        OSPrioHighRdy = (INT8U)((y << 3)) + OSUnMapTbl[OSRdyTbl[y]]);
    #else
        INT8U y;
        INT16U *ptbl;
        if((OSRdyGrp & 0xFF)!=0){
            y = OSUnMapTbl[OSRdyGrp & 0xFF];
        } else {
            y = OSUnMapTbl[(OSRdyGrp >> 8) & 0xFF] + 8;
        }
    
        ptbl = &OSRdyTbl[y];
        if((*ptbl & 0xFF) !=0){
            OSPrioHighRdy = (INT8U)((y << 4) + OSUnMapTbl[(*ptbl & 0xFF)]);
        }else{
            OSPrioHighRdy = (INT8U)((y << 4) + OSUnMapTbl[(*ptbl >> 8) & 0xFF] + 8);
        }
    #endif
    }

### How to switch tasks on (uc/OS II + ARM 2440)?

When a kernel decides to run a different task, it simply saves the current task's **context (CPU registers)** in the current task’s context storage area **(Current task's stack)**. Once this operation is performed, the new task’s context is restored from its storage area **(New task’s stack)** and then resumes execution of the new task’s code. This process is called a ***context switch.***

#### OSCtxSw:(以下涉及[ARM汇编指令](https://kaixuan.art/archives/26/))

    OSCtxSw:
        STMFD   SP!,{LR}        ;PC
        STMFD   SP!,{R0-R12,LR} ;R0-R12 LR
        MRS     R1, CPSR        ;Push CPSR
        STMFD   SP!,{R0}       
    
        ;----------------------------------------------------------------------
        ;OSTCBCur->SOTCBStkPtr = Sp
        ;----------------------------------------------------------------------
        LDR     R0, =OSTCBCur
        LDR     R0,[R0]
        STR     SP!,[R0]
    
        BL      OSTaskSwHook
        ;----------------------------------------------------------------------
        ; OSTCBCur = OSTCBHighRdy;
        ;----------------------------------------------------------------------
        LDR     R0, =OSTCBHighRdy
        LDR     R1, =OSTCBCur
        LDR     R0, [R0]
        STR     R0, [R1]
    
        ;----------------------------------------------------------------------
        ; OSPrioCur = OSPrioHighRdy;
        ;----------------------------------------------------------------------
        LDR     R0, =OSPrioHighRdy
        LDR     R1, =OSPrioCur
        LDRB    R0, [R0]
        STRB    R0, [R1]
    
        ;----------------------------------------------------------------------
        ;  OSTCBHighRdy->OSTCBStkPtr;
        ;----------------------------------------------------------------------
        LDR     R0, =OSTCBHighRdy                                            
        LDR     R0, [R0]
        LDR     SP, [R0]
        ;----------------------------------------------------------------------
        ;Restore New task context
        ;----------------------------------------------------------------------
        LDMFD              SP!, {R0}    ;POP CPSR       
        MSR                SPSR_cxsf,R0
        LDMFD              SP!, {R0-R12, LR, PC}^

#### Critical section accessing during scheduling（进入临界区段的三种方法）

    #if  OS_CRITICAL_METHOD == 1 
        #define  OS_ENTER_CRITICAL()   (Cli()) 
        #define  OS_EXIT_CRITICAL()      (Sti())  
    
    #elseif   OS_CRITICAL_METHOD == 2  
        #define  OS_ENTER_CRITICAL()   (PushAndCli())  
        #define  OS_EXIT_CRITICAL()     (Pop()) 
    
    #elseif   OS_CRITICAL_METHOD == 3  
        #define  OS_ENTER_CRITICAL()  (cpu_sr = OSCPUSaveSR())
        #define  OS_EXIT_CRITICAL()   (OSCPURestoreSR(cpu_sr))
    #endif

####Priority Bitmap


#####How to get the highest priority from the "ready" tasks?

    #if OS_LOWEST_PRIO <= 63  
        INT8U  y;
        y= OSUnMapTbl[OSRdyGrp];           
        OSPrioHighRdy = (INT8U)((y << 3) + OSUnMapTbl[OSRdyTbl[y]]); 

##The Second: Interruption (Event triggered mechanism)
###Classfication of interruption
*   Interruption
    - 1.Outside interruption
    - 2.Hard interruption
*   Trap
    - 1.Inside interruption (INT 21H,SWI ?)
    - 2.Soft interruption
*   Exception

###Case1: Interruption on (ARM 2440 + uc/OS II)
![中断上下文切换.jpg](https://xuankai.wang/images/2019/05/14/07295E0D-9658-49F1-954A-0331EBF47218.jpg)

#### interruption vector

|         Type          | Address |
| :-------------------: | :-----: |
|          FIQ          |  0x1C   |
|          IRQ          |  0x18   |
|      (Reserved)       |  0x14   |
|      Data Abort       |  0x10   |
|    Prefetch Abort     |  0x0C   |
|  Software Interrupt   |  0x08   |
| Undefined Instruction |  0x04   |
|         Reset         |  0x00   |

####IRQ
    IRQ
        LDR     r0, =INTOFFSET      //基于中断向量表获取INTOFFSET
        LDR     r0, [r0]
        LDR     r1, =HandleEINT0    //IRQ中断基址
        LDR     pc, [r1, r0 lsl #2] //左移两位，因为一个地址占用四个单位空间

####How to save the information of the interrupted program?
    IRQ
        SUB     lr, lr,#4
        STMFD   sp!,{r0.r12, lr}
        MRS     r0,spsr
        STMFD   sp!,{r0}
        LDR     r0, =INTOFFSET          ;10: 0x28
        LDR     r0, [r0]
        LDR     r1, =HandleEINT0
        ADD     r1, r1, r0, lsl #2
        LDR     r1,[r1]
        MOV     lr, pc                  ;Why not (pc-4): pc -> LDMFD sp!,{r0}
        MOV     pc, R1                  ;跳转至中断子程序执行
        LDMFD   sp!,{r0}
        MSR     spsr_cxsf,r0
        LDMFD   sp!,{r0.r12,lr}
        MOVS    pc,lr

####How to process the IRQ if uc/OS II is running?
    IRQ
        STMDB   sp!,{r0-r2}                      ;sp_irq 
        MOV     r0,sp 
        ADD     sp,sp,#12 
        SUB     r1,lr,#4                         ;lr_irq 
        MRS       r2,spsr

        MSR     cpsr_cxsf,#SVCMODE|NOINT  
        STMDB   sp!,{r1}                         ;sp_svc 
        STMDB   sp!,{r3-r12,lr} 
        LDMIA   r0!,{r3-r5}   
        STMDB   sp!,{r2-r5}
    
        //中断嵌套变量的汇编代码（不要求掌握）
        LDR     r0,=OSIntNesting 
        LDR     r1,[r0]
        ADD     r1,r1,#1 
        STRB    r1,[r0]
        TEQ     r1,#1
        BNE     %F1  
    
        LDR     r0,=OSTCBCur 
        LDR     r0,[r0]
        STR     sp,[r0]
        MSR     psr_c,#IRQMODE|NOINT
        LDR     r0,= INTOFFSET                  ;10:    0x28
        LDR     r0,[r0]
        LDR     r1,=HandleEINT0                 ;0x33FFFF20
            
        MOV     lr,pc 
        LDR     pc,[r1,r0,lsl#2]
        MSR     cpsr_c,#SVCMODE|NOINT 
        BL      OSIntExit
    
        LDR     r0,[sp],#4
        MSR     spsr_cxsf,r0 
        LDMIA   sp!,{r0-r12,lr,pc}^

####How to process context switch triggered by the IRQ?
    void  OSIntExit (void)  {
        #if OS_CRITICAL_METHOD == 3
            OS_CPU_SR  cpu_sr;
        #endif
        
        if (OSRunning == TRUE) {
            OS_ENTER_CRITICAL();
            if (OSIntNesting > 0) {     
            OSIntNesting--;
            }
            If ((OSIntNesting == 0) && (OSLockNesting == 0)) {
                OSIntExitY= OSUnMapTbl[OSRdyGrp];
                OSPrioHighRdy=(INT8U)((OSIntExitY<<3)+ OSUnMapTbl[OSRdyTbl[OSIntExitY]]);
                if (OSPrioHighRdy != OSPrioCur) {
                    OSTCBHighRdy= OSTCBPrioTbl[OSPrioHighRdy];
                    OSCtxSwCtr++;
                    OSIntCtxSw(); 
                }    
            }    
        OS_EXIT_CRITICAL();
        }    
    }


###Case2: Interruption of ARM11MPCORE
*   Distributed Interrupt Controller
    -   1.Interrupt Distributor
    -   2.CPU Interfaces
*   Generic Interrupt Controller
*   Respond the interruption
*   Initialize the interruption

##The third: Time management(Time triggered mechanism)
*   Time management is important
*   Real-time Clock(RTC)
*   System Clock(Timer)
*   Size of system clock
    -   Tick(the period of output impulse generated by the timer)
*   Time management
    -   Task Delay(uc/OS II: *osTimeDly()*)
    -   Difference time chain
    -   Watchdog timer
    -   System clock ISR

###OSTimeDly
    void  OSTimeDly (INT16U ticks){                                        
    #if OS_CRITICAL_METHOD == 3
          OS_CPU_SR  cpu_sr;
    #endif    

```c++
if (ticks > 0){ 
    OS_ENTER_CRITICAL();
    if ((OSRdyTbl[OSTCBCur->OSTCBY] &= ~OSTCBCur->OSTCBBitX) == 0){
        OSRdyGrp &= ~OSTCBCur->OSTCBBitY;    }
        OSTCBCur->OSTCBDly = ticks; 
        OS_EXIT_CRITICAL();
        OS_Sched(); 
    }
}
```
<!-- TOC -->

- [1. 初衷](#1-初衷)
- [2. \_KTRAP\_FRAME](#2-_ktrap_frame)
- [3. \_CONTEXT](#3-_context)
- [4. \_KPCR](#4-_kpcr)
  - [4.1. \_NT\_TIB](#41-_nt_tib)
  - [4.2. \_KPRCB](#42-_kprcb)
- [5. \_ETHREAD](#5-_ethread)
  - [5.1. \_KTHREAD](#51-_kthread)
    - [5.1.1. \_DISPATCHER\_HEADER](#511-_dispatcher_header)
    - [5.1.2. \_KAPC\_STATE](#512-_kapc_state)
    - [5.1.3. \_KAPC](#513-_kapc)
    - [5.1.4. \_KWAIT\_BLOCK](#514-_kwait_block)
  - [5.2. \_CM\_POST\_BLOCK](#52-_cm_post_block)
  - [5.3. \_CLIENT\_ID](#53-_client_id)
- [6. \_EPROCESS](#6-_eprocess)
  - [6.1. \_KPROCESS](#61-_kprocess)
  - [6.2. \_HANDLE\_TABLE](#62-_handle_table)
  - [6.3. \_MMVAD](#63-_mmvad)
    - [6.3.1. \_CONTROL\_AREA](#631-_control_area)
    - [6.3.2. \_MMVAD\_FLAGS](#632-_mmvad_flags)
- [7. 可等待对象](#7-可等待对象)
  - [7.1. \_KSEMAPHORE](#71-_ksemaphore)
  - [7.2. \_KMUTANT](#72-_kmutant)
- [8. 异常相关](#8-异常相关)
  - [8.1. \_EXCEPTION\_RECORD](#81-_exception_record)
  - [8.2. \_EXCEPTION\_REGISTRATION\_RECORD](#82-_exception_registration_record)
  - [8.3. VC\_EXCEPTION\_REGISTRATION](#83-vc_exception_registration)
    - [8.3.1. scopetable\_entry](#831-scopetable_entry)
  - [8.4. \_VECTORED\_EXCEPTION\_NODE](#84-_vectored_exception_node)
- [9. \_MMPFN](#9-_mmpfn)
- [10. TEB](#10-teb)
  - [10.1. \_NT\_TIB](#101-_nt_tib)
  - [10.2. \_CLIENT\_ID](#102-_client_id)
  - [10.3. PEB](#103-peb)
    - [10.3.1. \_PEB\_LDR\_DATA](#1031-_peb_ldr_data)
      - [10.3.1.1. \_LDR\_DATA\_TABLE\_ENTRY](#10311-_ldr_data_table_entry)
      - [10.3.1.2. \_UNICODE\_STRING](#10312-_unicode_string)
    - [10.3.2. \_HEAP](#1032-_heap)

<!-- /TOC -->
# 1. 初衷
其他笔记中，涉及到结构体的时候，都只是记录了其中一部分（完整记录会显得篇幅过长），这就导致查阅起来比较麻烦，这里做一个完整的结构体解释，以便查阅
# 2. _KTRAP_FRAME
```x86asm
kd> dt _kTrap_Frame
nt!_KTRAP_FRAME
   +0x000 DbgEbp           : Uint4B          ; 调试等其他作用
   +0x004 DbgEip           : Uint4B          ;
   +0x008 DbgArgMark       : Uint4B          ;
   +0x00c DbgArgPointer    : Uint4B          ;
   +0x010 TempSegCs        : Uint4B          ;
   +0x014 TempEsp          : Uint4B          ;
   +0x018 Dr0              : Uint4B          ;
   +0x01c Dr1              : Uint4B          ;
   +0x020 Dr2              : Uint4B          ;
   +0x024 Dr3              : Uint4B          ;
   +0x028 Dr6              : Uint4B          ;
   +0x02c Dr7              : Uint4B          ;
   +0x030 SegGs            : Uint4B          ;
   +0x034 SegEs            : Uint4B          ;
   +0x038 SegDs            : Uint4B          ;
   +0x03c Edx              : Uint4B          ;
   +0x040 Ecx              : Uint4B          ;
   +0x044 Eax              : Uint4B          ;
   +0x048 PreviousPreviousMode : Uint4B                             ; Windows中的非易失性寄存器需要在中断例程中先保存
   +0x04c ExceptionList    : Ptr32 _EXCEPTION_REGISTRATION_RECORD   ;
   +0x050 SegFs            : Uint4B                                 ;
   +0x054 Edi              : Uint4B                                 ;
   +0x058 Esi              : Uint4B                                 ;
   +0x05c Ebx              : Uint4B                                 ;
   +0x060 Ebp              : Uint4B                                 ;
   +0x064 ErrCode          : Uint4B                                 ;
   +0x068 Eip              : Uint4B   ; 中断门时保存代码段和地址，iret返回时用
   +0x06c SegCs            : Uint4B   ;
   +0x070 EFlags           : Uint4B   ;
   +0x074 HardwareEsp      : Uint4B   ; 中断门时若发生权限切换，保存旧堆栈
   +0x078 HardwareSegSs    : Uint4B   ;
   +0x07c V86Es            : Uint4B   ; 虚拟8086模式下保存段寄存器，保护模式下不使用
   +0x080 V86Ds            : Uint4B   ;
   +0x084 V86Fs            : Uint4B   ;
   +0x088 V86Gs            : Uint4B   ;
```
# 3. _CONTEXT
```x86asm
kd> dt _CONTEXT
nt!_CONTEXT
   +0x000 ContextFlags     : Uint4B
   +0x004 Dr0              : Uint4B
   +0x008 Dr1              : Uint4B
   +0x00c Dr2              : Uint4B
   +0x010 Dr3              : Uint4B
   +0x014 Dr6              : Uint4B
   +0x018 Dr7              : Uint4B
   +0x01c FloatSave        : _FLOATING_SAVE_AREA
   +0x08c SegGs            : Uint4B
   +0x090 SegFs            : Uint4B
   +0x094 SegEs            : Uint4B
   +0x098 SegDs            : Uint4B
   +0x09c Edi              : Uint4B
   +0x0a0 Esi              : Uint4B
   +0x0a4 Ebx              : Uint4B
   +0x0a8 Edx              : Uint4B
   +0x0ac Ecx              : Uint4B
   +0x0b0 Eax              : Uint4B
   +0x0b4 Ebp              : Uint4B
   +0x0b8 Eip              : Uint4B
   +0x0bc SegCs            : Uint4B
   +0x0c0 EFlags           : Uint4B
   +0x0c4 Esp              : Uint4B
   +0x0c8 SegSs            : Uint4B
   +0x0cc ExtendedRegisters : [512] UChar
```
# 4. _KPCR
```x86asm
kd> dt _kpcr
nt!_KPCR
   +0x000 NtTib            : _NT_TIB
   +0x01c SelfPcr          : Ptr32 _KPCR         ;指向自身（_KPCR）
   +0x020 Prcb             : Ptr32 _KPRCB        ;指针，指向_KPCR的PrcbData成员，方便日后修改拓展_KPCR结构体
   +0x024 Irql             : UChar
   +0x028 IRR              : Uint4B
   +0x02c IrrActive        : Uint4B
   +0x030 IDR              : Uint4B
   +0x034 KdVersionBlock   : Ptr32 Void
   +0x038 IDT              : Ptr32 _KIDTENTRY    ;IDT表基址，每个CPU都有自己独立的IDT表
   +0x03c GDT              : Ptr32 _KGDTENTRY    ;GDT表基址，每个CPU都有自己独立的GDT表
   +0x040 TSS              : Ptr32 _KTSS         ;TSS任务段基址，每个CPU都有自己独立的TSS任务段
   +0x044 MajorVersion     : Uint2B
   +0x046 MinorVersion     : Uint2B
   +0x048 SetMember        : Uint4B
   +0x04c StallScaleFactor : Uint4B
   +0x050 DebugActive      : UChar
   +0x051 Number           : UChar               ;当前CPU编号
   +0x052 Spare0           : UChar
   +0x053 SecondLevelCacheAssociativity : UChar
   +0x054 VdmAlert         : Uint4B
   +0x058 KernelReserved   : [14] Uint4B
   +0x090 SecondLevelCacheSize : Uint4B
   +0x094 HalReserved      : [16] Uint4B
   +0x0d4 InterruptMode    : Uint4B
   +0x0d8 Spare1           : UChar
   +0x0dc KernelReserved2  : [17] Uint4B
   +0x120 PrcbData         : _KPRCB              ;_KPCR的拓展结构体

```
## 4.1. _NT_TIB
```x86asm
kd> dt _NT_TIB
nt!_NT_TIB
   +0x000 ExceptionList    : Ptr32 _EXCEPTION_REGISTRATION_RECORD    ;当前线程内核异常链表（SEH）
   +0x004 StackBase        : Ptr32 Void          ;当前线程内核栈基址
   +0x008 StackLimit       : Ptr32 Void          ;当前线程内核栈大小
   +0x00c SubSystemTib     : Ptr32 Void
   +0x010 FiberData        : Ptr32 Void
   +0x010 Version          : Uint4B
   +0x014 ArbitraryUserPointer : Ptr32 Void
   +0x018 Self             : Ptr32 _NT_TIB       ;指向自身（_NT_TIB和_KPCR）
```
## 4.2. _KPRCB
```x86asm
kd> dt _KPRCB
nt!_KPRCB
   +0x000 MinorVersion     : Uint2B
   +0x002 MajorVersion     : Uint2B
   +0x004 CurrentThread    : Ptr32 _KTHREAD      ;CPU当前正在运行的线程
   +0x008 NextThread       : Ptr32 _KTHREAD      ;CPU下一个调度的线程
   +0x00c IdleThread       : Ptr32 _KTHREAD      ;空闲线程，如果没有其它线程可以调度时，CPU需要调度的线程
   +0x010 Number           : Char                ;CPU编号
   +0x011 Reserved         : Char
   +0x012 BuildType        : Uint2B
   +0x014 SetMember        : Uint4B
   +0x018 CpuType          : Char
   +0x019 CpuID            : Char
   +0x01a CpuStep          : Uint2B              ;CPU子版本号
   +0x01c ProcessorState   : _KPROCESSOR_STATE   ;CPU状态
   +0x33c KernelReserved   : [16] Uint4B
   +0x37c HalReserved      : [16] Uint4B
   +0x3bc PrcbPad0         : [92] UChar
   +0x418 LockQueue        : [16] _KSPIN_LOCK_QUEUE
   +0x498 PrcbPad1         : [8] UChar
   +0x4a0 NpxThread        : Ptr32 _KTHREAD      ;NPX浮点寄存器，最后一次用过NPX浮点寄存器的线程
   +0x4a4 InterruptCount   : Uint4B              ;中断次数
   +0x4a8 KernelTime       : Uint4B              ;统计信息
   +0x4ac UserTime         : Uint4B              ;统计信息
   +0x4b0 DpcTime          : Uint4B
   +0x4b4 DebugDpcTime     : Uint4B
   +0x4b8 InterruptTime    : Uint4B
   +0x4bc AdjustDpcThreshold : Uint4B
   +0x4c0 PageColor        : Uint4B
   +0x4c4 SkipTick         : Uint4B
   +0x4c8 MultiThreadSetBusy : UChar
   +0x4c9 Spare2           : [3] UChar
   +0x4cc ParentNode       : Ptr32 _KNODE
   +0x4d0 MultiThreadProcessorSet : Uint4B
   +0x4d4 MultiThreadSetMaster : Ptr32 _KPRCB
   +0x4d8 ThreadStartCount : [2] Uint4B
   +0x4e0 CcFastReadNoWait : Uint4B
   +0x4e4 CcFastReadWait   : Uint4B
   +0x4e8 CcFastReadNotPossible : Uint4B
   +0x4ec CcCopyReadNoWait : Uint4B
   +0x4f0 CcCopyReadWait   : Uint4B
   +0x4f4 CcCopyReadNoWaitMiss : Uint4B
   +0x4f8 KeAlignmentFixupCount : Uint4B
   +0x4fc KeContextSwitches : Uint4B
   +0x500 KeDcacheFlushCount : Uint4B
   +0x504 KeExceptionDispatchCount : Uint4B
   +0x508 KeFirstLevelTbFills : Uint4B
   +0x50c KeFloatingEmulationCount : Uint4B
   +0x510 KeIcacheFlushCount : Uint4B
   +0x514 KeSecondLevelTbFills : Uint4B
   +0x518 KeSystemCalls    : Uint4B
   +0x51c SpareCounter0    : [1] Uint4B
   +0x520 PPLookasideList  : [16] _PP_LOOKASIDE_LIST
   +0x5a0 PPNPagedLookasideList : [32] _PP_LOOKASIDE_LIST
   +0x6a0 PPPagedLookasideList : [32] _PP_LOOKASIDE_LIST
   +0x7a0 PacketBarrier    : Uint4B
   +0x7a4 ReverseStall     : Uint4B
   +0x7a8 IpiFrame         : Ptr32 Void
   +0x7ac PrcbPad2         : [52] UChar
   +0x7e0 CurrentPacket    : [3] Ptr32 Void
   +0x7ec TargetSet        : Uint4B
   +0x7f0 WorkerRoutine    : Ptr32     void 
   +0x7f4 IpiFrozen        : Uint4B
   +0x7f8 PrcbPad3         : [40] UChar
   +0x820 RequestSummary   : Uint4B
   +0x824 SignalDone       : Ptr32 _KPRCB
   +0x828 PrcbPad4         : [56] UChar
   +0x860 DpcListHead      : _LIST_ENTRY
   +0x868 DpcStack         : Ptr32 Void
   +0x86c DpcCount         : Uint4B
   +0x870 DpcQueueDepth    : Uint4B
   +0x874 DpcRoutineActive : Uint4B
   +0x878 DpcInterruptRequested : Uint4B
   +0x87c DpcLastCount     : Uint4B
   +0x880 DpcRequestRate   : Uint4B
   +0x884 MaximumDpcQueueDepth : Uint4B
   +0x888 MinimumDpcRate   : Uint4B
   +0x88c QuantumEnd       : Uint4B              ;标志当前线程的CPU时间片是否用完，没用完的时候是0，用完的时候是非0值
   +0x890 PrcbPad5         : [16] UChar
   +0x8a0 DpcLock          : Uint4B
   +0x8a4 PrcbPad6         : [28] UChar
   +0x8c0 CallDpc          : _KDPC
   +0x8e0 ChainedInterruptList : Ptr32 Void
   +0x8e4 LookasideIrpFloat : Int4B
   +0x8e8 SpareFields0     : [6] Uint4B
   +0x900 VendorString     : [13] UChar
   +0x90d InitialApicId    : UChar
   +0x90e LogicalProcessorsPerPhysicalProcessor : UChar
   +0x910 MHz              : Uint4B              ;频率
   +0x914 FeatureBits      : Uint4B
   +0x918 UpdateSignature  : _LARGE_INTEGER
   +0x920 NpxSaveArea      : _FX_SAVE_AREA
   +0xb30 PowerState       : _PROCESSOR_POWER_STATE
```
# 5. _ETHREAD
```x86asm
kd> dt _ETHREAD
nt!_ETHREAD
   +0x000 Tcb              : _KTHREAD            ;指向内核层的KTHREAD对象
   +0x1c0 CreateTime       : _LARGE_INTEGER      ;线程的创建时间
   +0x1c0 NestedFaultCount : Pos 0, 2 Bits
   +0x1c0 ApcNeeded        : Pos 2, 1 Bit
   +0x1c8 ExitTime         : _LARGE_INTEGER      ;线程的退出时间，在ExitThread中赋值
   +0x1c8 LpcReplyChain    : _LIST_ENTRY         ;用于跨进程通信(LPC)
   +0x1c8 KeyedWaitChain   : _LIST_ENTRY         ;用于带键事件的等待链表
   +0x1d0 ExitStatus       : Int4B               ;线程的退出状态，当线程主动退出或被动退出时这个域会由框架代码填充
   +0x1d0 OfsChain         : Ptr32 Void
   +0x1d4 PostBlockList    : _LIST_ENTRY         ;双向链表，链表中的各个节点类型为PCM_POST_BLOCK，它被用于一个线程向"配置管理器"登记注册表键的变化通知
   +0x1dc TerminationPort  : Ptr32 _TERMINATION_PORT  ;链表头，当一个线程退出时，系统会通知所有已经登记过要接收其终止事件的那些"端口"
   +0x1dc ReaperLink       : Ptr32 _ETHREAD      ;单链表节点，仅在线程退出时使用。当线程被终止时，该节点将被挂到PsReaperListHead链表上(用以告知内核当前线程将要退出了，请收到相关的线程资源)，所以，在线程回收器(reaper)的工作项目(WorkItem)中该线程的内核栈得以收回
   +0x1dc KeyedWaitValue   : Ptr32 Void
   +0x1e0 ActiveTimerListLock : Uint4B           ;双向链表头，包含了当前线程的所有定时器
   +0x1e4 ActiveTimerListHead : _LIST_ENTRY      ;操作这个链表(包含当前线程的所有定时器的双链表)的自旋锁。使用自旋锁可以把原本可能发生的并行事件导致的问题通过强制串行化得到解决。比如对线程中的定时器这个互斥资源就典型的需要串行化，否则将导致定时器的错乱等很多问题
   +0x1ec Cid              : _CLIENT_ID          ;进程ID和线程ID
   +0x1f4 LpcReplySemaphore : _KSEMAPHORE        ;用于LPC应答通知
   +0x1f4 KeyedWaitSemaphore : _KSEMAPHORE       ;用于处理带键的事件
   +0x208 LpcReplyMessage  : Ptr32 Void          ;LPC应答的消息
   +0x208 LpcWaitingOnPort : Ptr32 Void          ;说明了线程在哪个"端口对象"上等待消息
   +0x20c ImpersonationInfo : Ptr32 _PS_IMPERSONATION_INFORMATION    ;线程的模仿信息，windows允许一个线程在执行过程中模仿其他的用户来执行一段功能，这样可以实现更为灵活的访问控制安全特性
   +0x210 IrpList          : _LIST_ENTRY         ;双向链表头，其中包含了当前线程所有正在处理但尚未完成的I/O请求(Irp对象)
   +0x218 TopLevelIrp      : Uint4B              ;指向线程的顶级IRP
   +0x21c DeviceToVerify   : Ptr32 _DEVICE_OBJECT;指向的是一个"待检验"的设备，当磁盘或CD-ROM设备的驱动程序"发现"自从上一次该线程访问该设备以来，该设备有了"变化"，就会设置线程的DeviceToVerify域，从而使最高层的驱动程序(比如文件系统)，可以检测到设备变化
   +0x220 ThreadsProcess   : Ptr32 _EPROCESS     ;当前进程结构体的指针，可以定位进程
   +0x224 StartAddress     : Ptr32 Void          ;线程的启动地址，通常是系统DLL中的线程启动地址，(例如kernel32.dll中的BaseProcessStart或BaseThreadStart函数)
   +0x228 Win32StartAddress : Ptr32 Void         ;线程的启动地址，windows子系统接收到的线程启动地址，即CreateThread API函数接收到的线程启动地址
   +0x228 LpcReceivedMessageId : Uint4B          ;包含了接收到的LPC消息的ID
   +0x22c ThreadListEntry  : _LIST_ENTRY         ;连接一个进程所有线程的双向链表，总共存在两个这样的链表
   +0x234 RundownProtect   : _EX_RUNDOWN_REF
   +0x238 ThreadLock       : _EX_PUSH_LOCK       ;推锁，用户保护线程的数据属性
   +0x23c LpcReplyMessageId : Uint4B             ;指明了当前线程正在等待对一个LPC消息的应答
   +0x240 ReadClusterSize  : Uint4B              ;指明了在一次I/O操作中读取多少个页面，用于页面交换文件和内存映射文件的读操作
   +0x244 GrantedAccess    : Uint4B              ;线程的访问权限
   +0x248 CrossThreadFlags : Uint4B              ;针对跨线程访问的标志位
   +0x248 Terminated       : Pos 0, 1 Bit        ;线程已终止操作
   +0x248 DeadThread       : Pos 1, 1 Bit        ;线程创建失败
   +0x248 HideFromDebugger : Pos 2, 1 Bit        ;该线程对于调试器不可见
   +0x248 ActiveImpersonationInfo : Pos 3, 1 Bit ;线程正在模仿
   +0x248 SystemThread     : Pos 4, 1 Bit        ;是一个系统线程
   +0x248 HardErrorsAreDisabled : Pos 5, 1 Bit   ;对于该线程，硬件错误无效
   +0x248 BreakOnTermination : Pos 6, 1 Bit      ;调试器在线程终止时停下该线程
   +0x248 SkipCreationMsg  : Pos 7, 1 Bit        ;不向调试器发送创建消息
   +0x248 SkipTerminationMsg : Pos 8, 1 Bit      ;不向调试器发送终止消息
   +0x24c SameThreadPassiveFlags : Uint4B        ;一些只有在最低中断级别(被动级别)上只能被该线程自身访问的标志位，访问时无需互锁操作
   +0x24c ActiveExWorker   : Pos 0, 1 Bit
   +0x24c ExWorkerCanWaitUser : Pos 1, 1 Bit
   +0x24c MemoryMaker      : Pos 2, 1 Bit
   +0x250 SameThreadApcFlags : Uint4B            ;一些在APC中断级别(也是很低的级别)上被该线程自身访问的标志位，访问时无需互锁操作
   +0x250 LpcReceivedMsgIdValid : Pos 0, 1 Bit
   +0x250 LpcExitThreadCalled : Pos 1, 1 Bit
   +0x250 AddressSpaceOwner : Pos 2, 1 Bit
   +0x254 ForwardClusterOnly : UChar             ;指示是否仅仅前向聚集，页面错误处理相关
   +0x255 DisablePageFaultClustering : UChar     ;用于控制页面交换的聚集与否。页面错误处理相关
```
## 5.1. _KTHREAD
```x86asm
kd> dt _kthread
nt!_KTHREAD
   +0x000 Header           : _DISPATCHER_HEADER  ;标识_KTHREAD即线程为可等待对象
   +0x010 MutantListHead   : _LIST_ENTRY         ;线程拥有的互斥体双向链表
   +0x018 InitialStack     : Ptr32 Void          ;线程切换相关
   +0x01c StackLimit       : Ptr32 Void          ;线程切换相关
   +0x020 Teb              : Ptr32 Void          ;Thread Environment Block，线程环境块，大小4KB，线程在3环的一个结构体（3环进程可以对其进行读写），里面包含了线程的重要信息，在三环时fs:[0] -> teb，在0环时fs:[0] -> _kpcr
   +0x024 TlsArray         : Ptr32 Void
   +0x028 KernelStack      : Ptr32 Void          ;线程切换时存储栈顶指针
   +0x02c DebugActive      : UChar               ;调试相关，该值为-1的话不能使用调试寄存器Dr0-Dr7，3环进入0环时不会填充_kTrap_Frame中调试相关成员的值
   +0x02d State            : UChar               ;线程状态：就绪、等待、运行，仅仅用作记录，和线程的调度无关，1-就绪，2-运行，5-等待
   +0x02e Alerted          : [2] UChar
   +0x030 Iopl             : UChar
   +0x031 NpxState         : UChar
   +0x032 Saturation       : Char
   +0x033 Priority         : Char
   +0x034 ApcState         : _KAPC_STATE         ;APC队列
   +0x04c ContextSwitches  : Uint4B
   +0x050 IdleSwapBlock    : UChar
   +0x051 Spare0           : [3] UChar
   +0x054 WaitStatus       : Int4B
   +0x058 WaitIrql         : UChar
   +0x059 WaitMode         : Char
   +0x05a WaitNext         : UChar
   +0x05b WaitReason       : UChar
   +0x05c WaitBlockList    : Ptr32 _KWAIT_BLOCK  ;等待块列表，同步机制中线程等待与唤醒相关
   +0x060 WaitListEntry    : _LIST_ENTRY         ;线程所属等待链表，线程调度相关
   +0x060 SwapListEntry    : _SINGLE_LIST_ENTRY  ;线程所属调度链表，线程调度相关，和线程所属等待链表占据同一个位置，也就是说线程只能同时属于其中一个链表
   +0x068 WaitTime         : Uint4B
   +0x06c BasePriority     : Char                ;基础优先级，从所属进程继承而来（KPROCESS->BasePriority），但是可以通过KeSetBasePriorityThread函数重新设置
   +0x06d DecrementCount   : UChar
   +0x06e PriorityDecrement : Char
   +0x06f Quantum          : Char                ;调度相关，CPU时间片
   +0x070 WaitBlock        : [4] _KWAIT_BLOCK    ;指示线程等待的是哪个对象
   +0x0d0 LegoData         : Ptr32 Void
   +0x0d4 KernelApcDisable : Uint4B              ;判断当前线程是否禁用内核APC，非0代表禁用，0代表不禁用
   +0x0d8 UserAffinity     : Uint4B
   +0x0dc SystemAffinityActive : UChar
   +0x0dd PowerState       : UChar
   +0x0de NpxIrql          : UChar
   +0x0df InitialNode      : UChar
   +0x0e0 ServiceTable     : Ptr32 Void          ;系统服务表基址
   +0x0e4 Queue            : Ptr32 _KQUEUE
   +0x0e8 ApcQueueLock     : Uint4B              ;APC相关
   +0x0f0 Timer            : _KTIMER
   +0x118 QueueListEntry   : _LIST_ENTRY
   +0x120 SoftAffinity     : Uint4B
   +0x124 Affinity         : Uint4B
   +0x128 Preempted        : UChar
   +0x129 ProcessReadyQueue : UChar
   +0x12a KernelStackResident : UChar
   +0x12b NextProcessor    : UChar
   +0x12c CallbackStack    : Ptr32 Void
   +0x130 Win32Thread      : Ptr32 Void          ;对于非GUI程序，该值为NULL，对于GUI程序，该值指向THREADINFO结构体（含有成员消息队列MessageQueue）
   +0x134 TrapFrame        : Ptr32 _KTRAP_FRAME  ;用于进0环时保存环境
   +0x138 ApcStatePointer  : [2] Ptr32 _KAPC_STATE    ;正常情况下，ApcStatePointer[0]指向ApcState，ApcStatePointer[1]指向SavedApcState，线程挂靠情况下，指针指向相反
   +0x140 PreviousMode     : Char                ;先前模式，用于某些内核函数判断程序是0环调用的还是3环调用的
   +0x141 EnableStackSwap  : UChar
   +0x142 LargeStack       : UChar
   +0x143 ResourceIndex    : UChar
   +0x144 KernelTime       : Uint4B
   +0x148 UserTime         : Uint4B
   +0x14c SavedApcState    : _KAPC_STATE         ;备用APC队列
   +0x164 Alertable        : UChar               ;标识当前线程在等待时是否可以被APC唤醒，0代表不可以，1代表可以；SleepEx和WaitForSingleObjectEx函数可以通过最后一个参数显示指定该值（该参数为True代表可以被唤醒，False代表不可以）
   +0x165 ApcStateIndex    : UChar               ;标识当前线程处于什么状态，0表示正常状态，1表示挂靠状态
   +0x166 ApcQueueable     : UChar               ;表示是否可以向线程的APC队列中插入APC，当线程正在执行退出的代码时，会将这个值设置为0，如果此时执行插入APC的代码（KeInsertQueueApc），在插入函数中会判断这个值是否为0，如果是则插入失败
   +0x167 AutoAlignment    : UChar
   +0x168 StackBase        : Ptr32 Void
   +0x16c SuspendApc       : _KAPC
   +0x19c SuspendSemaphore : _KSEMAPHORE
   +0x1b0 ThreadListEntry  : _LIST_ENTRY         ;连接一个进程所有线程的双向链表，总共存在两个这样的链表
   +0x1b8 FreezeCount      : Char
   +0x1b9 SuspendCount     : Char
   +0x1ba IdealProcessor   : UChar
   +0x1bb DisableBoost     : UChar
```
### 5.1.1. _DISPATCHER_HEADER
```
x86asm
kd> dt _DISPATCHER_HEADER
nt!_DISPATCHER_HEADER
   +0x000 Type             : UChar              ;可等待对象类型，0-事件，1-事件，2-互斥体，5-信号量
   +0x001 Absolute         : UChar              ;三个统计信息
   +0x002 Size             : UChar
   +0x003 Inserted         : UChar
   +0x004 SignalState      : Int4B              ;是否有信号
   +0x008 WaitListHead     : _LIST_ENTRY        ;链表头，指向等待块列表，这是一个双向循环链表，链接了所有等待该可等待对象的线程的等待块
```
### 5.1.2. _KAPC_STATE
```x86asm
kd> dt _KAPC_STATE
nt!_KAPC_STATE
    +0x000 ApcListHead      : [2] _LIST_ENTRY   ;APC队列，两个双向链表，里面存储了线程需要执行的APC函数。第一个是内核APC链表（函数为内核空间函数），第二个是用户APC链表（函数为用户空间函数）
    +0x010 Process          : Ptr32 _KPROCESS   ;指向为线程提供CR3的进程
    +0x014 KernelApcInProgress : UChar          ;指示内核APC函数是否正在执行，1代表正在执行
    +0x015 KernelApcPending : UChar             ;是否存在内核APC函数，存在则置1，不存在置0
    +0x016 UserApcPending   : UChar             ;是否存在用户APC函数，存在则置1，不存在置0
```
### 5.1.3. _KAPC
```x86asm
kd> dt _KAPC
nt!_KAPC
    +0x000 Type             : Int2B           ;Windows内核对象（进程、线程、事件等）类型，APC为0x12
    +0x002 Size             : Int2B           ;本结构体大小，0x30
    +0x004 Spare0           : Uint4B          ;无意义，用于内存对齐
    +0x008 Thread           : Ptr32 _KTHREAD  ;目标线程
    +0x00c ApcListEntry     : _LIST_ENTRY     ;APC队列所挂位置
    +0x014 KernelRoutine    : Ptr32     void  ;指向一个函数，APC函数执行完毕之后Windows会调用这个函数，该函数需调用ExFreePoolWithTag来销毁KAPC结构体 
    +0x018 RundownRoutine   : Ptr32     void  ;未使用
    +0x01c NormalRoutine    : Ptr32     void  ;用户APC：用户APC总入口；内核APC：函数地址
    +0x020 NormalContext    : Ptr32 Void      ;内核APC：NULL；用户APC：函数地址
    +0x024 SystemArgument1  : Ptr32 Void      ;APC参数1
    +0x028 SystemArgument2  : Ptr32 Void      ;APC参数2
    +0x02c ApcStateIndex    : Char            ;标识希望APC挂在哪个队列
    +0x02d ApcMode          : Char            ;0代表内核APC，1代表用户APC
    +0x02e Inserted         : UChar           ;标识当前APC是否已经插入队列，是为1，否为0
```
### 5.1.4. _KWAIT_BLOCK
```
kd> dt _KWAIT_BLOCK
nt!_KWAIT_BLOCK
   +0x000 WaitListEntry    : _LIST_ENTRY            ;连接一个被等待对象的所有等待块的双向链表
   +0x008 Thread           : Ptr32 _KTHREAD         ;等待块对应线程
   +0x00c Object           : Ptr32 Void             ;被等待对象的地址
   +0x010 NextWaitBlock    : Ptr32 _KWAIT_BLOCK     ;连接一个线程的所有等待块的单向循环链表中的下一跳指针
   +0x014 WaitKey          : Uint2B                 ;线程的等待块的索引
   +0x016 WaitType         : Uint2B                 ;等待类型，0代表需要所有被等待对象都符合条件才能激活，1代表只需要一个被等待对象符合条件就可以激活

```
## 5.2. _CM_POST_BLOCK
```c
typedef struct _CM_POST_BLOCK
{ 
#if DBG 
    BOOLEAN                     TraceIntoDebugger; 
#endif 
    LIST_ENTRY                  NotifyList; 
    LIST_ENTRY                  ThreadList; 
    LIST_ENTRY                  CancelPostList;    //slave notifications that are attached to this notification
    struct _CM_POST_KEY_BODY    *PostKeyBody; 
    ULONG                       NotifyType; 
    PCM_POST_BLOCK_UNION        u; 
} CM_POST_BLOCK, *PCM_POST_BLOCK; 
```
## 5.3. _CLIENT_ID
```c
typedef struct _CLIENT_ID
{
     PVOID UniqueProcess;           //等于所属进程的UniqueProcessId
     PVOID UniqueThread;            //等于此线程对象在进程句柄表中的句柄
} CLIENT_ID, *PCLIENT_ID;
```
# 6. _EPROCESS
```x86asm
kd> dt _EPROCESS
nt!_EPROCESS
   +0x000 Pcb              : _KPROCESS
   +0x06c ProcessLock      : _EX_PUSH_LOCK       ;控制结构体只允许一个人改
   +0x070 CreateTime       : _LARGE_INTEGER      ;进程的创建时间
   +0x078 ExitTime         : _LARGE_INTEGER      ;进程的退出时间
   +0x080 RundownProtect   : _EX_RUNDOWN_REF     ;线程的停止保护锁，对于跨线程引用TEB结构或者挂起线程的执行等操作，需要获得此锁才能运行，以避免在操作过程中线程被销毁
   +0x084 UniqueProcessId  : Ptr32 Void          ;进程ID
   +0x088 ActiveProcessLinks : _LIST_ENTRY       ;连接所有活动进程的双向链表，全局变量PsActiveProcessHead指向链表头。利用PsActiveProcessHead加上该成员可以遍历所有进程，通过断链可以使得可以从WindowsAPI中隐藏进程，但是进程仍然可以正常运行（Windows的调度单元是线程，进程仅仅提供了一个运行环境如资源和物理页基址）
   +0x090 QuotaUsage       : [3] Uint4B          ;物理页相关统计信息
   +0x09c QuotaPeak        : [3] Uint4B          ;物理页相关统计信息
   +0x0a8 CommitCharge     : Uint4B              ;物理页相关
   +0x0ac PeakVirtualSize  : Uint4B              ;虚拟内存峰值（虚拟内存就是用磁盘充当内存）
   +0x0b0 VirtualSize      : Uint4B              ;虚拟内存大小
   +0x0b4 SessionProcessLinks : _LIST_ENTRY      
   +0x0bc DebugPort        : Ptr32 Void          ;调试端口（INT 01和INT 03，消息发到调试端口，这两种异常调试器一般要单独处理），该值为0的情况下无法通过正常手段来调试进程（可以通过其它手段）
   +0x0c0 ExceptionPort    : Ptr32 Void          ;异常端口（其它异常，消息先发到调试器，然后发到异常端口），调试相关
   +0x0c4 ObjectTable      : Ptr32 _HANDLE_TABLE ;对象句柄表，存有进程使用的各种其它内核的对象（事件、互斥体、文件等）的句柄
   +0x0c8 Token            : _EX_FAST_REF
   +0x0cc WorkingSetLock   : _FAST_MUTEX
   +0x0ec WorkingSetPage   : Uint4B
   +0x0f0 AddressCreationLock : _FAST_MUTEX
   +0x110 HyperSpaceLock   : Uint4B
   +0x114 ForkInProgress   : Ptr32 _ETHREAD
   +0x118 HardwareTrigger  : Uint4B
   +0x11c VadRoot          : Ptr32 Void          ;标识用户地址空间0-2G使用情况（虚拟地址描述符）
   +0x120 VadHint          : Ptr32 Void
   +0x124 CloneRoot        : Ptr32 Void
   +0x128 NumberOfPrivatePages : Uint4B
   +0x12c NumberOfLockedPages : Uint4B
   +0x130 Win32Process     : Ptr32 Void
   +0x134 Job              : Ptr32 _EJOB
   +0x138 SectionObject    : Ptr32 Void
   +0x13c SectionBaseAddress : Ptr32 Void
   +0x140 QuotaBlock       : Ptr32 _EPROCESS_QUOTA_BLOCK
   +0x144 WorkingSetWatch  : Ptr32 _PAGEFAULT_HISTORY
   +0x148 Win32WindowStation : Ptr32 Void
   +0x14c InheritedFromUniqueProcessId : Ptr32 Void
   +0x150 LdtInformation   : Ptr32 Void
   +0x154 VadFreeHint      : Ptr32 Void
   +0x158 VdmObjects       : Ptr32 Void
   +0x15c DeviceMap        : Ptr32 Void
   +0x160 PhysicalVadList  : _LIST_ENTRY
   +0x168 PageDirectoryPte : _HARDWARE_PTE
   +0x168 Filler           : Uint8B
   +0x170 Session          : Ptr32 Void
   +0x174 ImageFileName    : [16] UChar          ;进程镜像文件名，最多显示16个字节，多余字节将被截断
   +0x184 JobLinks         : _LIST_ENTRY
   +0x18c LockedPagesList  : Ptr32 Void
   +0x190 ThreadListHead   : _LIST_ENTRY         ;连接该进程所有线程的双向链表（总共存在两个这样的链表）的头，示意图见2.1节
   +0x198 SecurityPort     : Ptr32 Void
   +0x19c PaeTop           : Ptr32 Void
   +0x1a0 ActiveThreads    : Uint4B              ;活动线程的数量，0代表无活动线程，进程即将结束，将会从活动进程链表中被移除
   +0x1a4 GrantedAccess    : Uint4B
   +0x1a8 DefaultHardErrorProcessing : Uint4B
   +0x1ac LastThreadExitStatus : Int4B           ;进程中的每个线程退出时，除了对ETHREAD的ExitStatus赋值以外，还会给当前线程所属的进程的该域进行赋值
   +0x1b0 Peb              : Ptr32 _PEB          ;Process Environment Block，进程环境块，进程在3环的一个结构体（3环进程可以对其进行读写），里面包含了进程的模块列表、是否处于调试状态等重要信息
   +0x1b4 PrefetchTrace    : _EX_FAST_REF
   +0x1b8 ReadOperationCount : _LARGE_INTEGER
   +0x1c0 WriteOperationCount : _LARGE_INTEGER
   +0x1c8 OtherOperationCount : _LARGE_INTEGER
   +0x1d0 ReadTransferCount : _LARGE_INTEGER
   +0x1d8 WriteTransferCount : _LARGE_INTEGER
   +0x1e0 OtherTransferCount : _LARGE_INTEGER
   +0x1e8 CommitChargeLimit : Uint4B
   +0x1ec CommitChargePeak : Uint4B
   +0x1f0 AweInfo          : Ptr32 Void
   +0x1f4 SeAuditProcessCreationInfo : _SE_AUDIT_PROCESS_CREATION_INFO
   +0x1f8 Vm               : _MMSUPPORT          ;进程已经使用的物理页相关内容
   +0x238 LastFaultCount   : Uint4B
   +0x23c ModifiedPageCount : Uint4B
   +0x240 NumberOfVads     : Uint4B
   +0x244 JobStatus        : Uint4B
   +0x248 Flags            : Uint4B
   +0x248 CreateReported   : Pos 0, 1 Bit
   +0x248 NoDebugInherit   : Pos 1, 1 Bit
   +0x248 ProcessExiting   : Pos 2, 1 Bit
   +0x248 ProcessDelete    : Pos 3, 1 Bit
   +0x248 Wow64SplitPages  : Pos 4, 1 Bit
   +0x248 VmDeleted        : Pos 5, 1 Bit
   +0x248 OutswapEnabled   : Pos 6, 1 Bit
   +0x248 Outswapped       : Pos 7, 1 Bit
   +0x248 ForkFailed       : Pos 8, 1 Bit
   +0x248 HasPhysicalVad   : Pos 9, 1 Bit
   +0x248 AddressSpaceInitialized : Pos 10, 2 Bits
   +0x248 SetTimerResolution : Pos 12, 1 Bit
   +0x248 BreakOnTermination : Pos 13, 1 Bit
   +0x248 SessionCreationUnderway : Pos 14, 1 Bit
   +0x248 WriteWatch       : Pos 15, 1 Bit
   +0x248 ProcessInSession : Pos 16, 1 Bit
   +0x248 OverrideAddressSpace : Pos 17, 1 Bit
   +0x248 HasAddressSpace  : Pos 18, 1 Bit
   +0x248 LaunchPrefetched : Pos 19, 1 Bit
   +0x248 InjectInpageErrors : Pos 20, 1 Bit
   +0x248 VmTopDown        : Pos 21, 1 Bit
   +0x248 Unused3          : Pos 22, 1 Bit
   +0x248 Unused4          : Pos 23, 1 Bit
   +0x248 VdmAllowed       : Pos 24, 1 Bit
   +0x248 Unused           : Pos 25, 5 Bits
   +0x248 Unused1          : Pos 30, 1 Bit
   +0x248 Unused2          : Pos 31, 1 Bit
   +0x24c ExitStatus       : Int4B
   +0x250 NextPageColor    : Uint2B
   +0x252 SubSystemMinorVersion : UChar
   +0x253 SubSystemMajorVersion : UChar
   +0x252 SubSystemVersion : Uint2B
   +0x254 PriorityClass    : UChar
   +0x255 WorkingSetAcquiredUnsafe : UChar
   +0x258 Cookie           : Uint4B
```
## 6.1. _KPROCESS
```x86asm
kd> dt _KPROCESS
nt!_KPROCESS
   +0x000 Header           : _DISPATCHER_HEADER    ;标识_KPROCESS即进程为可等待对象（即可被WaitForSingleObject等函数等待的对象，如Mutex互斥体、Event事件等），所有可等待对象的第一个成员均为_DISPATCHER_HEADER结构体
   +0x010 ProfileListHead  : _LIST_ENTRY
   +0x018 DirectoryTableBase : [2] Uint4B          ;页目录表的基址
   +0x020 LdtDescriptor    : _KGDTENTRY            ;历史遗留问题，16位Windows段选择子不够，所以每个进程都有一个LDT表；32位Windows下该成员无意义
   +0x028 Int21Descriptor  : _KIDTENTRY            ;DOS下使用
   +0x030 IopmOffset       : Uint2B
   +0x032 Iopl             : UChar
   +0x033 Unused           : UChar
   +0x034 ActiveProcessors : Uint4B
   +0x038 KernelTime       : Uint4B                ;统计信息，统计进程在内核模式下所花的时间
   +0x03c UserTime         : Uint4B                ;统计信息，统计进程在用户模式下所花的时间
   +0x040 ReadyListHead    : _LIST_ENTRY
   +0x048 SwapListEntry    : _SINGLE_LIST_ENTRY
   +0x04c VdmTrapcHandler  : Ptr32 Void
   +0x050 ThreadListHead   : _LIST_ENTRY           ;连接该进程所有线程的双向链表（总共存在两个这样的链表）的头
   +0x058 ProcessLock      : Uint4B
   +0x05c Affinity         : Uint4B                ;规定进程里面的所有线程能够在哪个CPU上运行，比特位为1的下标号对应的CPU可以运行，比如4（100）代表可以在2号CPU上运行，如果此时只有一个或者两个CPU，该进程将会结束。所以32位最多32核，64位最多64核
   +0x060 StackCount       : Uint2B
   +0x062 BasePriority     : Char                  ;基础优先级，该进程中所有线程最起码的优先级（即所有线程的优先级均会大于等于此优先级）
   +0x063 ThreadQuantum    : Char                  ;调度相关，线程CPU时间片基础值
   +0x064 AutoAlignment    : UChar
   +0x065 State            : UChar
   +0x066 ThreadSeed       : UChar
   +0x067 DisableBoost     : UChar
   +0x068 PowerState       : UChar
   +0x069 DisableQuantum   : UChar
   +0x06a IdealNode        : UChar
   +0x06b Flags            : _KEXECUTE_OPTIONS
   +0x06b ExecuteOptions   : UChar
```
## 6.2. _HANDLE_TABLE
```x86asm
nt!_HANDLE_TABLE
   +0x000 TableCode        : Uint4B              ;句柄表地址
   +0x004 QuotaProcess     : Ptr32 _EPROCESS
   +0x008 UniqueProcessId  : Ptr32 Void
   +0x00c HandleTableLock  : [4] _EX_PUSH_LOCK
   +0x01c HandleTableList  : _LIST_ENTRY
   +0x024 HandleContentionEvent : _EX_PUSH_LOCK
   +0x028 DebugInfo        : Ptr32 _HANDLE_TRACE_DEBUG_INFO
   +0x02c ExtraInfoPages   : Int4B
   +0x030 FirstFree        : Uint4B
   +0x034 LastFree         : Uint4B
   +0x038 NextHandleNeedingPool : Uint4B
   +0x03c HandleCount      : Int4B
   +0x040 Flags            : Uint4B
   +0x040 StrictFIFO       : Pos 0, 1 Bit
```
## 6.3. _MMVAD
```
kd> dt _MMVAD
nt!_MMVAD
   +0x000 StartingVpn      : Uint4B                    ;起始线性地址
   +0x004 EndingVpn        : Uint4B                    ;结束线性地址
   +0x008 Parent           : Ptr32 _MMVAD              ;父节点
   +0x00c LeftChild        : Ptr32 _MMVAD              ;左子节点
   +0x010 RightChild       : Ptr32 _MMVAD              ;右子节点
   +0x014 u                : __unnamed                 ;类型为_MMVAD_FLAGS，指示了内存块的一些属性
   +0x018 ControlArea      : Ptr32 _CONTROL_AREA       ;内存块类别
   +0x01c FirstPrototypePte : Ptr32 _MMPTE
   +0x020 LastContiguousPte : Ptr32 _MMPTE
   +0x024 u2               : __unnamed
```
### 6.3.1. _CONTROL_AREA
```
kd> dt _CONTROL_AREA
nt!_CONTROL_AREA
   +0x000 Segment          : Ptr32 _SEGMENT
   +0x004 DereferenceList  : _LIST_ENTRY
   +0x00c NumberOfSectionReferences : Uint4B
   +0x010 NumberOfPfnReferences : Uint4B
   +0x014 NumberOfMappedViews : Uint4B
   +0x018 NumberOfSubsections : Uint2B
   +0x01a FlushInProgressCount : Uint2B
   +0x01c NumberOfUserReferences : Uint4B
   +0x020 u                : __unnamed
   +0x024 FilePointer      : Ptr32 _FILE_OBJECT         ;私有内存-NULL；映射内存的文件映射-文件路径；映射内存的共享内存-NULL
   +0x028 WaitingForDeletion : Ptr32 _EVENT_COUNTER
   +0x02c ModifiedWriteCount : Uint2B
   +0x02e NumberOfSystemCacheViews : Uint2B

```
### 6.3.2. _MMVAD_FLAGS
```
kd> dt _MMVAD_FLAGS
nt!_MMVAD_FLAGS
   +0x000 CommitCharge     : Pos 0, 19 Bits
   +0x000 PhysicalMapping  : Pos 19, 1 Bit
   +0x000 ImageMap         : Pos 20, 1 Bit               ;如果内存是映射内存，指示映射文件是否为可执行文件，1代表是，0代表不是
   +0x000 UserPhysicalPages : Pos 21, 1 Bit
   +0x000 NoChange         : Pos 22, 1 Bit
   +0x000 WriteWatch       : Pos 23, 1 Bit
   +0x000 Protection       : Pos 24, 5 Bits              ;内存的权限：1-READONLY，2-EXECUTE，3-EXECUTE_READ，4-READWRITE
                                                         ;5-WRITECOPY，6-EXECUTE_READWRITE，7-EXECUTE_WRITECOPY
   +0x000 LargePages       : Pos 29, 1 Bit
   +0x000 MemCommit        : Pos 30, 1 Bit
   +0x000 PrivateMemory    : Pos 31, 1 Bit               ;指示了内存块的类型，1代表私有内存，0代表映射内存
```
# 7. 可等待对象
## 7.1. _KSEMAPHORE
```x86asm
nt!_KSEMAPHORE
   +0x000 Header           : _DISPATCHER_HEADER
   +0x010 Limit            : Int4B
```
## 7.2. _KMUTANT
```x86asm
kd> dt _KMUTANT
nt!_KMUTANT
   +0x000 Header           : _DISPATCHER_HEADER
   +0x010 MutantListEntry  : _LIST_ENTRY           ;是个链表头，圈着所有互斥体
   +0x018 OwnerThread      : Ptr32 _KTHREAD        ;正在拥有互斥体的线程
   +0x01c Abandoned        : UChar                 ;是否已经被放弃不用
   +0x01d ApcDisable       : UChar                 ;是否禁用内核APC
```
# 8. 异常相关
## 8.1. _EXCEPTION_RECORD
```c
typedef struct _EXCEPTION_RECORD {
  DWORD                    ExceptionCode;          //异常代码
  DWORD                    ExceptionFlags;         //异常状态，CPU异常值为0，软件模拟异常值为1，嵌套异常值为0x10
  struct _EXCEPTION_RECORD *ExceptionRecord;       //下一个异常，通常为空，出现嵌套异常时，会用到这个成员
  PVOID                    ExceptionAddress;       //异常发生地址
  DWORD                    NumberParameters;       //附加参数个数，用以进一步描述异常的信息，很少用到
  ULONG_PTR                ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];     //附加参数指针，用以进一步描述异常的信息，很少用到
} EXCEPTION_RECORD;
```
## 8.2. _EXCEPTION_REGISTRATION_RECORD
```c
typedef struct _EXCEPTION_REGISTRATION_RECORD
{
     PEXCEPTION_REGISTRATION_RECORD Next;          //异常链表下一个异常结构体指针
     PEXCEPTION_DISPOSITION Handler;               //异常处理函数句柄
} EXCEPTION_REGISTRATION_RECORD, *PEXCEPTION_REGISTRATION_RECORD;
```
## 8.3. VC_EXCEPTION_REGISTRATION
```c
struct VC_EXCEPTION_REGISTRATION
{
     VC_EXCEPTION_REGISTRATION* prev;        //前一个结构体的指针
     FARPROC                    handler;     //永远指向_exception_handler3回调函数
     scopetable_entry*          scopetable;  //作用域表，scopetable指向了一个scopetable_entry结构体数组，函数中每个try{}都会在其中占据一个位置
     int                        _index;      //scopetable_entry结构体数组的索引，trylevel
                                             //该值会在函数执行期间被改变（每当代码进入新的try{}结构时该值会变化），
                                             //该值指示当前执行代码所位于的try{}结构在scopetable中的索引，
                                             //如果当前执行代码不位于任何try{}结构中，_index会被设置为-1。
     DWORD                      _ebp;        //函数起始时压入的ebp，上一个函数体的栈基址
                                             //对于使用FPO优化的函数，编译器通常不生成标准的堆栈帧
                                             //如果函数使用了SEH，那么无论是否使用FPO优化，编译器一定生成标准的堆栈帧（第一条指令PUSH EBP）
                                             //函数刚进入，压入EBP，移动esp之后，便压入了该异常结构，所以ebp的负偏移可以用于寻址
}
```
### 8.3.1. scopetable_entry
```c
struct scopetable_entry
{
     DWORD     prev_entryindex;   //该try{}结构的上层try{}结构在scopetable中的索引，如果没有上层try{}结构，则该值为-1
     FARPROC   lpfnFilter;        //过滤函数地址，except的小括号代码块地址，finally时为NULL
     FARPROC   lpfnHandler;       //异常处理程序地址，except或finally代码块地址
}
```
## 8.4. _VECTORED_EXCEPTION_NODE
VEH结点。
```c
struct _VECTORED_EXCEPTION_NODE
{
   DWORD m_pNextNode; 
   DWORD m_pPreviousNode;
   PVOID m_pfnVectoredHandler;
}
```
# 9. _MMPFN
物理页相关。该结构体包含大量union，意义复杂。不同的操作系统版本该结构体的大小不同，有的是0x18，有的是0x1C。
```
kd> dt _MMPFN
nt!_MMPFN
   +0x000 u1               : __unnamed
   +0x004 PteAddress       : Ptr32 _MMPTE
   +0x008 u2               : __unnamed
   +0x00c u3               : __unnamed
   +0x010 OriginalPte      : _MMPTE
   +0x018 u4               : __unnamed
```
# 10. TEB
TEB(Thread Environment Block，线程环境块)，其中存放着进程中所有线程的各种信息。ntdll.NtCurrentTeb函数，fs:[0x18]，fs:[0x0]均可定位TEB。
```
nt!_TEB
   +0x000 NtTib            : _NT_TIB                      ;TIB（Thread Information Block，线程信息块）
   +0x01c EnvironmentPointer : Ptr32 Void
   +0x020 ClientId         : _CLIENT_ID                   ;存储了PID和TID
   +0x028 ActiveRpcHandle  : Ptr32 Void
   +0x02c ThreadLocalStoragePointer : Ptr32 Void
   +0x030 ProcessEnvironmentBlock : Ptr32 _PEB            ;指向PEB，EP中EBX寄存器默认为PEB的结构体地址
   +0x034 LastErrorValue   : Uint4B
   +0x038 CountOfOwnedCriticalSections : Uint4B
   +0x03c CsrClientThread  : Ptr32 Void
   +0x040 Win32ThreadInfo  : Ptr32 Void
   +0x044 User32Reserved   : [26] Uint4B
   +0x0ac UserReserved     : [5] Uint4B
   +0x0c0 WOW32Reserved    : Ptr32 Void
   +0x0c4 CurrentLocale    : Uint4B
   +0x0c8 FpSoftwareStatusRegister : Uint4B
   +0x0cc SystemReserved1  : [54] Ptr32 Void
   +0x1a4 ExceptionCode    : Int4B
   +0x1a8 ActivationContextStack : _ACTIVATION_CONTEXT_STACK
   +0x1bc SpareBytes1      : [24] UChar
   +0x1d4 GdiTebBatch      : _GDI_TEB_BATCH
   +0x6b4 RealClientId     : _CLIENT_ID
   +0x6bc GdiCachedProcessHandle : Ptr32 Void
   +0x6c0 GdiClientPID     : Uint4B
   +0x6c4 GdiClientTID     : Uint4B
   +0x6c8 GdiThreadLocalInfo : Ptr32 Void
   +0x6cc Win32ClientInfo  : [62] Uint4B
   +0x7c4 glDispatchTable  : [233] Ptr32 Void
   +0xb68 glReserved1      : [29] Uint4B
   +0xbdc glReserved2      : Ptr32 Void
   +0xbe0 glSectionInfo    : Ptr32 Void
   +0xbe4 glSection        : Ptr32 Void
   +0xbe8 glTable          : Ptr32 Void
   +0xbec glCurrentRC      : Ptr32 Void
   +0xbf0 glContext        : Ptr32 Void
   +0xbf4 LastStatusValue  : Uint4B
   +0xbf8 StaticUnicodeString : _UNICODE_STRING
   +0xc00 StaticUnicodeBuffer : [261] Uint2B
   +0xe0c DeallocationStack : Ptr32 Void
   +0xe10 TlsSlots         : [64] Ptr32 Void
   +0xf10 TlsLinks         : _LIST_ENTRY
   +0xf18 Vdm              : Ptr32 Void
   +0xf1c ReservedForNtRpc : Ptr32 Void
   +0xf20 DbgSsReserved    : [2] Ptr32 Void
   +0xf28 HardErrorsAreDisabled : Uint4B
   +0xf2c Instrumentation  : [16] Ptr32 Void
   +0xf6c WinSockData      : Ptr32 Void
   +0xf70 GdiBatchCount    : Uint4B
   +0xf74 InDbgPrint       : UChar
   +0xf75 FreeStackOnTermination : UChar
   +0xf76 HasFiberData     : UChar
   +0xf77 IdealProcessor   : UChar
   +0xf78 Spare3           : Uint4B
   +0xf7c ReservedForPerf  : Ptr32 Void
   +0xf80 ReservedForOle   : Ptr32 Void
   +0xf84 WaitingOnLoaderLock : Uint4B
   +0xf88 Wx86Thread       : _Wx86ThreadState
   +0xf94 TlsExpansionSlots : Ptr32 Ptr32 Void
   +0xf98 ImpersonationLocale : Uint4B
   +0xf9c IsImpersonating  : Uint4B
   +0xfa0 NlsCache         : Ptr32 Void
   +0xfa4 pShimData        : Ptr32 Void
   +0xfa8 HeapVirtualAffinity : Uint4B
   +0xfac CurrentTransactionHandle : Ptr32 Void
   +0xfb0 ActiveFrame      : Ptr32 _TEB_ACTIVE_FRAME
   +0xfb4 SafeThunkCall    : UChar
   +0xfb5 BooleanSpare     : [3] UChar
```
## 10.1. _NT_TIB
```x86asm
kd> dt _NT_TIB
nt!_NT_TIB
   +0x000 ExceptionList    : Ptr32 _EXCEPTION_REGISTRATION_RECORD    ;当前线程异常链表（SEH）
   +0x004 StackBase        : Ptr32 Void          ;当前线程栈基址
   +0x008 StackLimit       : Ptr32 Void          ;当前线程栈大小
   +0x00c SubSystemTib     : Ptr32 Void
   +0x010 FiberData        : Ptr32 Void
   +0x010 Version          : Uint4B
   +0x014 ArbitraryUserPointer : Ptr32 Void
   +0x018 Self             : Ptr32 _NT_TIB       ;指向自身（_NT_TIB和_TEB）
```
## 10.2. _CLIENT_ID
```x86asm
kd> dt _CLIENT_ID
nt!_CLIENT_ID
   +0x000 UniqueProcess    : Ptr32 Void          ;当前进程的的PID，函数GetCurrentProcessId访问的结构体成员
   +0x004 UniqueThread     : Ptr32 Void          ;当前线程的的TID，函数GetCurrentThreadId访问的结构体成员
```
## 10.3. PEB
```x86asm
nt!_PEB
   +0x000 InheritedAddressSpace : UChar
   +0x001 ReadImageFileExecOptions : UChar
   +0x002 BeingDebugged    : UChar                                ;表示当前进程是否处于调试状态，函数IsDebuggerPresent访问的结构体成员
   +0x003 SpareBool        : UChar
   +0x004 Mutant           : Ptr32 Void
   +0x008 ImageBaseAddress : Ptr32 Void                           ;自身映像基址，函数GetModuleHandle(0)获取自身模块句柄所访问的结构体成员
   +0x00c Ldr              : Ptr32 _PEB_LDR_DATA                  ;结构体指针，可以获取进程加载的所有模块的基址和其他信息
   +0x010 ProcessParameters : Ptr32 _RTL_USER_PROCESS_PARAMETERS
   +0x014 SubSystemData    : Ptr32 Void
   +0x018 ProcessHeap      : Ptr32 Void                           ;指向进程堆的相关结构体_HEAP的指针，函数GetProcessHeap访问的结构体成员
   +0x01c FastPebLock      : Ptr32 _RTL_CRITICAL_SECTION
   +0x020 FastPebLockRoutine : Ptr32 Void
   +0x024 FastPebUnlockRoutine : Ptr32 Void
   +0x028 EnvironmentUpdateCount : Uint4B
   +0x02c KernelCallbackTable : Ptr32 Void
   +0x030 SystemReserved   : [1] Uint4B
   +0x034 AtlThunkSListPtr32 : Uint4B
   +0x038 FreeList         : Ptr32 _PEB_FREE_BLOCK
   +0x03c TlsExpansionCounter : Uint4B
   +0x040 TlsBitmap        : Ptr32 Void
   +0x044 TlsBitmapBits    : [2] Uint4B
   +0x04c ReadOnlySharedMemoryBase : Ptr32 Void
   +0x050 ReadOnlySharedMemoryHeap : Ptr32 Void
   +0x054 ReadOnlyStaticServerData : Ptr32 Ptr32 Void
   +0x058 AnsiCodePageData : Ptr32 Void
   +0x05c OemCodePageData  : Ptr32 Void
   +0x060 UnicodeCaseTableData : Ptr32 Void
   +0x064 NumberOfProcessors : Uint4B
   +0x068 NtGlobalFlag     : Uint4B                      ;在调试状态时，值为0x70
   +0x070 CriticalSectionTimeout : _LARGE_INTEGER
   +0x078 HeapSegmentReserve : Uint4B
   +0x07c HeapSegmentCommit : Uint4B
   +0x080 HeapDeCommitTotalFreeThreshold : Uint4B
   +0x084 HeapDeCommitFreeBlockThreshold : Uint4B
   +0x088 NumberOfHeaps    : Uint4B
   +0x08c MaximumNumberOfHeaps : Uint4B
   +0x090 ProcessHeaps     : Ptr32 Ptr32 Void
   +0x094 GdiSharedHandleTable : Ptr32 Void
   +0x098 ProcessStarterHelper : Ptr32 Void
   +0x09c GdiDCAttributeList : Uint4B
   +0x0a0 LoaderLock       : Ptr32 Void
   +0x0a4 OSMajorVersion   : Uint4B
   +0x0a8 OSMinorVersion   : Uint4B
   +0x0ac OSBuildNumber    : Uint2B
   +0x0ae OSCSDVersion     : Uint2B
   +0x0b0 OSPlatformId     : Uint4B
   +0x0b4 ImageSubsystem   : Uint4B
   +0x0b8 ImageSubsystemMajorVersion : Uint4B
   +0x0bc ImageSubsystemMinorVersion : Uint4B
   +0x0c0 ImageProcessAffinityMask : Uint4B
   +0x0c4 GdiHandleBuffer  : [34] Uint4B
   +0x14c PostProcessInitRoutine : Ptr32     void 
   +0x150 TlsExpansionBitmap : Ptr32 Void
   +0x154 TlsExpansionBitmapBits : [32] Uint4B
   +0x1d4 SessionId        : Uint4B
   +0x1d8 AppCompatFlags   : _ULARGE_INTEGER
   +0x1e0 AppCompatFlagsUser : _ULARGE_INTEGER
   +0x1e8 pShimData        : Ptr32 Void
   +0x1ec AppCompatInfo    : Ptr32 Void
   +0x1f0 CSDVersion       : _UNICODE_STRING
   +0x1f8 ActivationContextData : Ptr32 Void
   +0x1fc ProcessAssemblyStorageMap : Ptr32 Void
   +0x200 SystemDefaultActivationContextData : Ptr32 Void
   +0x204 SystemAssemblyStorageMap : Ptr32 Void
   +0x208 MinimumStackCommit : Uint4B
```
### 10.3.1. _PEB_LDR_DATA
```x86asm
ntdll!_PEB_LDR_DATA
   +0x000 Length            ;结构体大小
   +0x004 Initialized       ;进程是否初始化完成
   +0x008 SsHandle 
   +0x00c InLoadOrderModuleList : _LIST_ENTRY             ;双向链表，链接了进程中加载的所有DLL对应的_LDR_DATA_TABLE_ENTRY结构体
   +0x014 InMemoryOrderModuleList : _LIST_ENTRY           ;双向链表，链接了进程中加载的所有DLL对应的_LDR_DATA_TABLE_ENTRY结构体
   +0x01c InInitializationOrderModuleList : _LIST_ENTRY   ;双向链表，链接了进程中加载的所有DLL对应的_LDR_DATA_TABLE_ENTRY结构体
   +0x024 EntryInProgress
   +0x028 ShutdownInProgress
   +0x02c ShutdownThreadId
```
#### 10.3.1.1. _LDR_DATA_TABLE_ENTRY
进程中每个加载的DLL都有一个对应的_LDR_DATA_TABLE_ENTRY结构体
```x86asm
kd> dt _LDR_DATA_TABLE_ENTRY
nt!_LDR_DATA_TABLE_ENTRY
   +0x000 InLoadOrderLinks : _LIST_ENTRY
   +0x008 InMemoryOrderLinks : _LIST_ENTRY
   +0x010 InInitializationOrderLinks : _LIST_ENTRY
   +0x018 DllBase          : Ptr32 Void
   +0x01c EntryPoint       : Ptr32 Void
   +0x020 SizeOfImage      : Uint4B
   +0x024 FullDllName      : _UNICODE_STRING
   +0x02c BaseDllName      : _UNICODE_STRING
   +0x034 Flags            : Uint4B
   +0x038 LoadCount        : Uint2B
   +0x03a TlsIndex         : Uint2B
   +0x03c HashLinks        : _LIST_ENTRY
   +0x03c SectionPointer   : Ptr32 Void
   +0x040 CheckSum         : Uint4B
   +0x044 TimeDateStamp    : Uint4B
   +0x044 LoadedImports    : Ptr32 Void
   +0x048 EntryPointActivationContext : Ptr32 Void
   +0x04c PatchInformation : Ptr32 Void
```
#### 10.3.1.2. _UNICODE_STRING
```
typedef struct _UNICODE_STRING {
  USHORT Length;
  USHORT MaximumLength;
  PWSTR  Buffer;
} UNICODE_STRING, *PUNICODE_STRING;
```
### 10.3.2. _HEAP
```x86asm
kd> dt _HEAP
nt!_HEAP
   +0x000 Entry            : _HEAP_ENTRY
   +0x008 Signature        : Uint4B
   +0x00c Flags            : Uint4B        ;程序正常运行（非调试）时，值为2 
   +0x010 ForceFlags       : Uint4B        ;程序正常运行（非调试）时，值为0
   +0x014 VirtualMemoryThreshold : Uint4B
   +0x018 SegmentReserve   : Uint4B
   +0x01c SegmentCommit    : Uint4B
   +0x020 DeCommitFreeBlockThreshold : Uint4B
   +0x024 DeCommitTotalFreeThreshold : Uint4B
   +0x028 TotalFreeSize    : Uint4B
   +0x02c MaximumAllocationSize : Uint4B
   +0x030 ProcessHeapsListIndex : Uint2B
   +0x032 HeaderValidateLength : Uint2B
   +0x034 HeaderValidateCopy : Ptr32 Void
   +0x038 NextAvailableTagIndex : Uint2B
   +0x03a MaximumTagIndex  : Uint2B
   +0x03c TagEntries       : Ptr32 _HEAP_TAG_ENTRY
   +0x040 UCRSegments      : Ptr32 _HEAP_UCR_SEGMENT
   +0x044 UnusedUnCommittedRanges : Ptr32 _HEAP_UNCOMMMTTED_RANGE
   +0x048 AlignRound       : Uint4B
   +0x04c AlignMask        : Uint4B
   +0x050 VirtualAllocdBlocks : _LIST_ENTRY
   +0x058 Segments         : [64] Ptr32 _HEAP_SEGMENT
   +0x158 u                : __unnamed
   +0x168 u2               : __unnamed
   +0x16a AllocatorBackTraceIndex : Uint2B
   +0x16c NonDedicatedListLength : Uint4B
   +0x170 LargeBlocksIndex : Ptr32 Void
   +0x174 PseudoTagEntries : Ptr32 _HEAP_PSEUDO_TAG_ENTRY
   +0x178 FreeLists        : [128] _LIST_ENTRY
   +0x578 LockVariable     : Ptr32 _HEAP_LOCK
   +0x57c CommitRoutine    : Ptr32     long 
   +0x580 FrontEndHeap     : Ptr32 Void
   +0x584 FrontHeapLockCount : Uint2B
   +0x586 FrontEndHeapType : UChar
   +0x587 LastSegmentIndex : UChar
```

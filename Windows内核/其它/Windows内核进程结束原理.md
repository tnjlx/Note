- [1. NtTerminateProcess的函数实现](#1-ntterminateprocess的函数实现)
- [2. PspTerminateThreadByPointer的函数实现](#2-pspterminatethreadbypointer的函数实现)
- [3. PspExitThread的函数实现](#3-pspexitthread的函数实现)
- [4. 总结结束进程原理](#4-总结结束进程原理)
- [5. 内核APC执行时机](#5-内核apc执行时机)
  - [5.1. 中断和异常返回](#51-中断和异常返回)
  - [5.2. 高irql转到第irql](#52-高irql转到第irql)
  - [5.3. 线程切换的时候](#53-线程切换的时候)
- [6. 完整代码](#6-完整代码)
  - [6.1. Entry.c](#61-entryc)
  - [6.2. KrTypeDef.h](#62-krtypedefh)
  - [6.3. 防止过程中进程创建了新的线程](#63-防止过程中进程创建了新的线程)

# 1. NtTerminateProcess的函数实现
遍历进程的线程，然后对每个线程执行PspTerminateThreadByPointer函数。
`NTSTATUS NTAPI NtTerminateProcess(IN HANDLE ProcessHandle OPTIONAL, IN NTSTATUS ExitStatus)`

# 2. PspTerminateThreadByPointer的函数实现
* 目标为本身线程时，直接执行PspExitThread函数
* 目标为别的线程时，在对方线程插入一个内核APC，APC会执行PspExitThread函数
* 
# 3. PspExitThread的函数实现
* 执行了一大堆清理代码,主要清理当前线程Ethread的资源
* 从调度链表和等待链表中去掉它
* 如果是进程的最后一个线程,直接清理进程空间
* 执行KiSwapThread切换到一个新线程去
* 
# 4. 总结结束进程原理
结束进程就是依次结束进程的所有线程，最后一个线程会负责清理进程空间。线程只能被自己结束，所以需要通过内核APC来使得别的线程自己执行结束代码。

# 5. 内核APC执行时机
## 5.1. 中断和异常返回
## 5.2. 高irql转到第irql
KfLowerIrql -> HalpCheckForSoftwareInterrrupt -> HalpDispatchSoftwareInterrupt -> KiDeliverApc

## 5.3. 线程切换的时候
# 6. 完整代码
## 6.1. Entry.c
```c
#include <KrTypeDef.h>
#include <ntddk.h>
//需要杀死的进程id
#define PCHUNTER_ID   3232
VOID DriverUnload(PDRIVER_OBJECT pDriver);
PEPROCESS LookupProcess(HANDLE hPid);
PETHREAD LookupThread(HANDLE hTid);
VOID KillProcess(PEPROCESS pEProcess);
ULONG GetPspTerminateThreadByPointer();
ULONG GetPspExitThread(ULONG PspTerminateThreadByPointer);
VOID SelfTerminateThread(
    KAPC *Apc,
    PKNORMAL_ROUTINE *NormalRoutine,
    PVOID *NormalContext,
    PVOID *SystemArgument1,
    PVOID *SystemArgument2);
fpTypePspExitThread g_fpPspExitThreadAddr = NULL;
NTSTATUS DriverEntry(PDRIVER_OBJECT pDriver, PUNICODE_STRING pPath)
{
    DbgBreakPoint();
    pDriver->DriverUnload = DriverUnload;
    //提前把函数查找出来
    ULONG uPspTerminateThreadByPointerAddr =  GetPspTerminateThreadByPointer();
    if (0 == uPspTerminateThreadByPointerAddr)
    {
        KdPrint(("查找PspTerminateThreadByPointerAddr地址出错\n"));
        return STATUS_SUCCESS;
    }
    g_fpPspExitThreadAddr = (fpTypePspExitThread)GetPspExitThread(uPspTerminateThreadByPointerAddr);
    if (NULL == g_fpPspExitThreadAddr)
    {
        KdPrint(("查找PspExitThread地址出错\n"));
        return STATUS_SUCCESS;
    }
    //
    PEPROCESS pProcess = LookupProcess((HANDLE)PCHUNTER_ID);
    if (NULL == pProcess)
    {
        KdPrint((("没有在PsCidTable中找到进程,尼玛不会隐藏了吧\n")));
    }
    else
    {
        KillProcess(pProcess);
    }
    return STATUS_SUCCESS;
}
VOID DriverUnload(PDRIVER_OBJECT pDriver)
{
    KdPrint(("驱动退出\n"));
}
PEPROCESS LookupProcess(HANDLE hPid)
{
    PEPROCESS pEProcess = NULL;
    if (NT_SUCCESS(PsLookupProcessByProcessId(hPid, &pEProcess)))
        return pEProcess;
    return NULL;
}
VOID KillProcess(PEPROCESS pEProcess)
{
    PEPROCESS pEProc = NULL;
    PETHREAD  pEThrd = NULL;
    ULONG i = 0;
    for (i = 4; i < 0x25600; i += 4)
    {
        pEThrd = LookupThread((HANDLE)i);
        if (!pEThrd)  continue;
        pEProc = IoThreadToProcess(pEThrd);
        if (pEProc == pEProcess)
        {
            PKAPC pApc = NULL;
            pApc = (PKAPC)ExAllocatePool(NonPagedPool, sizeof(KAPC));
            if (NULL == pApc) return;
            //插入内核apc
            KeInitializeApc(pApc, (PKTHREAD)pEThrd, OriginalApcEnvironment, (PKKERNEL_ROUTINE)&SelfTerminateThread, NULL, NULL, 0, NULL);
            KeInsertQueueApc(pApc, NULL, 0, 2);
        }
        ObDereferenceObject(pEThrd);
    }
}
PETHREAD LookupThread(HANDLE hTid)
{
    PETHREAD pEThread = NULL;
    if (NT_SUCCESS(PsLookupThreadByThreadId(hTid, &pEThread)))
        return pEThread;
    return NULL;
}
VOID SelfTerminateThread(
    KAPC *Apc,
    PKNORMAL_ROUTINE *NormalRoutine,
    PVOID *NormalContext,
    PVOID *SystemArgument1,
    PVOID *SystemArgument2)
{
    ExFreePool(Apc);
    g_fpPspExitThreadAddr(STATUS_SUCCESS);
}
ULONG GetPspTerminateThreadByPointer()
{
    UNICODE_STRING funcName;
    RtlInitUnicodeString(&funcName, L"PsTerminateSystemThread");
    ULONG step = 0;
    ULONG targetFunAddr = 0;
    ULONG baseFunAddr = (ULONG)MmGetSystemRoutineAddress(&funcName);
    for (step = baseFunAddr; step < (baseFunAddr + 1024); step++)
    {
        //searching for 0x50,0xe8
        if (((*(PUCHAR)(UCHAR*)(step - 1)) == 0x50) && ((*(PUCHAR)(UCHAR*)(step)) == 0xe8))
        {
            ULONG offset = *(PULONG)(step + 1);
            targetFunAddr = step + 5 + offset;
            break;
        }
    }
    return targetFunAddr;
} //PspExitThread stamp code:0x0c 0xe8
ULONG GetPspExitThread(ULONG PspTerminateThreadByPointer)
{
    ULONG step = 0;
    ULONG targetFunAddr = 0;
    ULONG baseFunc = PspTerminateThreadByPointer;
    for (step = baseFunc; step < (baseFunc + 1024); step++)
    {
        //searching for 0x0c,0xe8
        if (((*(PUCHAR)(UCHAR*)(step - 1)) == 0x0c) && ((*(PUCHAR)(UCHAR*)(step)) == 0xe8))
        {
            ULONG m_offset = *(PULONG)(step + 1);
            targetFunAddr = step + 5 + m_offset;
            break;
        }
    }
    return targetFunAddr;
}
```
## 6.2. KrTypeDef.h
```c
#pragma  once
#include <ntifs.h>
#include <ntddk.h>
#pragma warning(disable:4189 4100)
typedef enum _KAPC_ENVIRONMENT
{
    OriginalApcEnvironment,
    AttachedApcEnvironment,
    CurrentApcEnvironment,
    InsertApcEnvironment
} KAPC_ENVIRONMENT;
typedef VOID (*PKNORMAL_ROUTINE) (
    IN PVOID NormalContext,
    IN PVOID SystemArgument1,
    IN PVOID SystemArgument2
    );
typedef VOID(*PKKERNEL_ROUTINE) (
    IN struct _KAPC *Apc,
    IN OUT PKNORMAL_ROUTINE *NormalRoutine,
    IN OUT PVOID *NormalContext,
    IN OUT PVOID *SystemArgument1,
    IN OUT PVOID *SystemArgument2
    );
typedef VOID(*PKRUNDOWN_ROUTINE) (
    IN struct _KAPC *Apc
    );
VOID NTAPI KeInitializeApc(__in PKAPC   Apc,
    __in PKTHREAD     Thread,
    __in KAPC_ENVIRONMENT     TargetEnvironment,
    __in PKKERNEL_ROUTINE     KernelRoutine,
    __in_opt PKRUNDOWN_ROUTINE    RundownRoutine,
    __in PKNORMAL_ROUTINE     NormalRoutine,
    __in KPROCESSOR_MODE  Mode,
    __in PVOID    Context
    );
BOOLEAN  NTAPI KeInsertQueueApc(IN PKAPC Apc,
    IN PVOID SystemArgument1,
    IN PVOID SystemArgument2,
    IN KPRIORITY PriorityBoost);
typedef VOID(NTAPI *fpTypePspExitThread)(
    IN NTSTATUS ExitStatus
    );
#define OFFSET(type, f) ((SIZE_T) ((char *)&((type *)0)->f - (char *)(type *)0))
```
## 6.3. 防止过程中进程创建了新的线程
锁住进程的_EPROCESS结构的+0x0b0偏移处的RundownProtect（_EX_RUNDOWN_REF结构体）即可

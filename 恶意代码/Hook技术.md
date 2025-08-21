- [1. Ring3 Hook](#1-ring3-hook)
  - [1.1. 如何实现全局Hook](#11-如何实现全局hook)
- [2. Ring0 Hook](#2-ring0-hook)
- [3. Hook检测](#3-hook检测)
- [4. Hook库](#4-hook库)
- [5. \_declspec(naked)](#5-_declspecnaked)

# 1. Ring3 Hook
* lpk Hook：Windows未公开函数InitializeLpkHooks，可Hook库lpk.dll中的4个函数LpkTabbedTextOut/LpkPSMTextOut/LpkDrawTextEx/LpkEditControl
* SetWindowsHookEx Hook：微软官方API，消息Hook
* Inline Hook（API Hook）
* IAT Hook：替换导入表中的函数地址
* EAT Hook：替换导出表中的函数地址
* SEH Hook：安装顶层SEH函数，并在函数开头插入触发异常代码，以此获取控制权
* VEH Hook：插入优先VEH函数，并在函数开头插入触发异常代码，以此获取控制权
* VirtualFunctionHook：替换C++虚函数表中的函数指针

## 1.1. 如何实现全局Hook
Hook NtResumeThread函数以拦截进程创建行为（NtResumeThread并非仅仅在创建进程时调用，需要进行区分）

# 2. Ring0 Hook
* Inline Hook（API Hook）
* IRP Hook
* MSR Hook（SYSENTER-HOOK/KiFastCallEntry-Hook）：修改SYSENTER_EIP_MSR寄存器，使其指向我们自己的函数
* SSDT Hook：加载驱动，Hook SSDT表
* Object Hook

# 3. Hook检测
* 读取本地文件，与内存中的代码进行对比
* 对内存模块进行CRC校验
* 设置回调函数，检测某个IAT或者函数的前几个指令是否被修改
* 监测VirtualProtect和WriteProcess等敏感函数
* 利用PsSetCreateProcessNotifyRoutineEx监控进程创建行为
* 利用PsSetCreateThreadNotifyRoutine监控线程创建行为
* 利用PsSetLoadImageNotifyRoutine拦截模块载入行为，修改OEP处代码为RET

# 4. Hook库
* EasyHook：支持Ring0（不够稳定）和Ring3，该库对多线程未进行处理
* Mhook：只支持Ring3
* Detours：微软官方Hook库

# 5. _declspec(naked)
就是告诉编译器，在编译的时候，不要优化代码，不要添加额外代码来控制堆栈平衡，一切代码都需要自己来写，防止破坏被Hook函数的堆栈或者导致堆栈不平衡。
```c
#define NAKED __declspec(naked)
void NAKED code(void)
{
    __asm{
        ret
    }
}
```

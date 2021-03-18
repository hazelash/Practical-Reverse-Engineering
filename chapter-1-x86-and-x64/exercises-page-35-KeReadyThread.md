# ObFastDereferenceObject

_Decompile the following kernel routine in Windows:_

```asm
.text:00000001400FFEDC KeReadyThread   proc near               ; CODE XREF: PspInsertThread+4B0↓p
.text:00000001400FFEDC                                         ; DATA XREF: .rdata:00000001403ECED0↓o ...
.text:00000001400FFEDC
.text:00000001400FFEDC ; FUNCTION CHUNK AT .text:000000014025B32E SIZE 0000002D BYTES
.text:00000001400FFEDC
.text:00000001400FFEDC                 push    rbx
.text:00000001400FFEDE                 sub     rsp, 20h
.text:00000001400FFEE2                 mov     rdx, [rcx+0B8h]
.text:00000001400FFEE9                 mov     rbx, rcx
.text:00000001400FFEEC                 mov     eax, [rdx+240h]
.text:00000001400FFEF2                 test    al, 7
.text:00000001400FFEF4                 jnz     short loc_1400FFF04
.text:00000001400FFEF6
.text:00000001400FFEF6 loc_1400FFEF6:                          ; CODE XREF: KeReadyThread+4F↓j
.text:00000001400FFEF6                 mov     rcx, rbx
.text:00000001400FFEF9                 call    KiFastReadyThread
.text:00000001400FFEFE
.text:00000001400FFEFE loc_1400FFEFE:                          ; CODE XREF: KeReadyThread+4D↓j
.text:00000001400FFEFE                 add     rsp, 20h
.text:00000001400FFF02                 pop     rbx
.text:00000001400FFF03                 retn
.text:00000001400FFF04 ; ---------------------------------------------------------------------------
.text:00000001400FFF04
.text:00000001400FFF04 loc_1400FFF04:                          ; CODE XREF: KeReadyThread+18↑j
.text:00000001400FFF04                 mov     r8, cr8
.text:00000001400FFF08                 mov     ecx, 2
.text:00000001400FFF0D                 mov     cr8, rcx
.text:00000001400FFF11                 mov     eax, cs:KiIrqlFlags
.text:00000001400FFF17                 test    eax, eax
.text:00000001400FFF19                 jnz     loc_14025B32E
.text:00000001400FFF1F
.text:00000001400FFF1F loc_1400FFF1F:                          ; CODE XREF: KeReadyThread+15B454↓j
.text:00000001400FFF1F                                         ; KeReadyThread+15B45D↓j ...
.text:00000001400FFF1F                 mov     rcx, rbx
.text:00000001400FFF22                 call    KiInSwapSingleProcess
.text:00000001400FFF27                 test    al, al
.text:00000001400FFF29                 jnz     short loc_1400FFEFE
.text:00000001400FFF2B                 jmp     short loc_1400FFEF6
.text:00000001400FFF2B KeReadyThread   endp
```

```c
ntdll!_KTHREAD
   +0x000 Header           : _DISPATCHER_HEADER
   +0x018 SListFaultAddress : Ptr64 Void
   +0x020 QuantumTarget    : Uint8B
   +0x028 InitialStack     : Ptr64 Void
   +0x030 StackLimit       : Ptr64 Void
   +0x038 StackBase        : Ptr64 Void
   +0x040 ThreadLock       : Uint8B
   +0x048 CycleTime        : Uint8B
   +0x050 CurrentRunTime   : Uint4B
   +0x054 ExpectedRunTime  : Uint4B
   +0x058 KernelStack      : Ptr64 Void
   +0x060 StateSaveArea    : Ptr64 _XSAVE_FORMAT
   +0x070 WaitRegister     : _KWAIT_STATUS_REGISTER
   +0x071 Running          : UChar
   +0x072 Alerted          : [2] UChar
   +0x074 AutoBoostActive  : Pos 0, 1 Bit
   +0x074 ReadyTransition  : Pos 1, 1 Bit
   +0x074 WaitNext         : Pos 2, 1 Bit
   +0x074 SystemAffinityActive : Pos 3, 1 Bit
   +0x074 Alertable        : Pos 4, 1 Bit
   +0x074 UserStackWalkActive : Pos 5, 1 Bit
   +0x074 ApcInterruptRequest : Pos 6, 1 Bit
   +0x074 QuantumEndMigrate : Pos 7, 1 Bit
   +0x074 UmsDirectedSwitchEnable : Pos 8, 1 Bit
   +0x074 TimerActive      : Pos 9, 1 Bit
   +0x074 SystemThread     : Pos 10, 1 Bit
   +0x074 ProcessDetachActive : Pos 11, 1 Bit
   +0x074 CalloutActive    : Pos 12, 1 Bit
   +0x074 ScbReadyQueue    : Pos 13, 1 Bit
   +0x074 ApcQueueable     : Pos 14, 1 Bit
   +0x074 ReservedStackInUse : Pos 15, 1 Bit
   +0x074 UmsPerformingSyscall : Pos 16, 1 Bit
   +0x074 TimerSuspended   : Pos 17, 1 Bit
   +0x074 SuspendedWaitMode : Pos 18, 1 Bit
   +0x074 SuspendSchedulerApcWait : Pos 19, 1 Bit
   +0x074 CetUserShadowStack : Pos 20, 1 Bit
   +0x074 BypassProcessFreeze : Pos 21, 1 Bit
   +0x074 Reserved         : Pos 22, 10 Bits
   +0x074 MiscFlags        : Int4B
   +0x078 BamQosLevel      : Pos 0, 2 Bits
   +0x078 AutoAlignment    : Pos 2, 1 Bit
   +0x078 DisableBoost     : Pos 3, 1 Bit
   +0x078 AlertedByThreadId : Pos 4, 1 Bit
   +0x078 QuantumDonation  : Pos 5, 1 Bit
   +0x078 EnableStackSwap  : Pos 6, 1 Bit
   +0x078 GuiThread        : Pos 7, 1 Bit
   +0x078 DisableQuantum   : Pos 8, 1 Bit
   +0x078 ChargeOnlySchedulingGroup : Pos 9, 1 Bit
   +0x078 DeferPreemption  : Pos 10, 1 Bit
   +0x078 QueueDeferPreemption : Pos 11, 1 Bit
   +0x078 ForceDeferSchedule : Pos 12, 1 Bit
   +0x078 SharedReadyQueueAffinity : Pos 13, 1 Bit
   +0x078 FreezeCount      : Pos 14, 1 Bit
   +0x078 TerminationApcRequest : Pos 15, 1 Bit
   +0x078 AutoBoostEntriesExhausted : Pos 16, 1 Bit
   +0x078 KernelStackResident : Pos 17, 1 Bit
   +0x078 TerminateRequestReason : Pos 18, 2 Bits
   +0x078 ProcessStackCountDecremented : Pos 20, 1 Bit
   +0x078 RestrictedGuiThread : Pos 21, 1 Bit
   +0x078 VpBackingThread  : Pos 22, 1 Bit
   +0x078 ThreadFlagsSpare : Pos 23, 1 Bit
   +0x078 EtwStackTraceApcInserted : Pos 24, 8 Bits
   +0x078 ThreadFlags      : Int4B
   +0x07c Tag              : UChar
   +0x07d SystemHeteroCpuPolicy : UChar
   +0x07e UserHeteroCpuPolicy : Pos 0, 7 Bits
   +0x07e ExplicitSystemHeteroCpuPolicy : Pos 7, 1 Bit
   +0x07f Spare0           : UChar
   +0x080 SystemCallNumber : Uint4B
   +0x084 ReadyTime        : Uint4B
   +0x088 FirstArgument    : Ptr64 Void
   +0x090 TrapFrame        : Ptr64 _KTRAP_FRAME
   +0x098 ApcState         : _KAPC_STATE
      +0x000 ApcListHead      : [2] _LIST_ENTRY
         +0x000 Flink            : Ptr64 _LIST_ENTRY
         +0x008 Blink            : Ptr64 _LIST_ENTRY
      +0x020 Process          : Ptr64 _KPROCESS
    ... cut to save space
   +0x5e0 ThreadTimerDelay : Uint4B
   +0x5e4 ThreadFlags2     : Int4B
   +0x5e4 PpmPolicy        : Pos 0, 2 Bits
   +0x5e4 ThreadFlags2Reserved : Pos 2, 30 Bits
   +0x5e8 TracingPrivate   : [1] Uint8B
   +0x5f0 SchedulerAssist  : Ptr64 Void
   +0x5f8 AbWaitObject     : Ptr64 Void

ntdll!_KSTACK_COUNT
   +0x000 Value            : Int4B
   +0x000 State            : Pos 0, 3 Bits
   +0x000 StackCount       : Pos 3, 29 Bits

0:000> dt nt!_KPCR
ntdll!_KPCR
   +0x000 NtTib            : _NT_TIB
   +0x000 GdtBase          : Ptr64 _KGDTENTRY64
   +0x008 TssBase          : Ptr64 _KTSS64
   +0x010 UserRsp          : Uint8B
   +0x018 Self             : Ptr64 _KPCR
   +0x020 CurrentPrcb      : Ptr64 _KPRCB
   +0x028 LockArray        : Ptr64 _KSPIN_LOCK_QUEUE
   +0x030 Used_Self        : Ptr64 Void
   +0x038 IdtBase          : Ptr64 _KIDTENTRY64
   +0x040 Unused           : [2] Uint8B
   +0x050 Irql             : UChar
   +0x051 SecondLevelCacheAssociativity : UChar
   +0x052 ObsoleteNumber   : UChar
   +0x053 Fill0            : UChar
   +0x054 Unused0          : [3] Uint4B
   +0x060 MajorVersion     : Uint2B
   +0x062 MinorVersion     : Uint2B
   +0x064 StallScaleFactor : Uint4B
   +0x068 Unused1          : [3] Ptr64 Void
   +0x080 KernelReserved   : [15] Uint4B
   +0x0bc SecondLevelCacheSize : Uint4B
   +0x0c0 HalReserved      : [16] Uint4B
   +0x100 Unused2          : Uint4B
   +0x108 KdVersionBlock   : Ptr64 Void
   +0x110 Unused3          : Ptr64 Void
   +0x118 PcrAlign1        : [24] Uint4B
   +0x180 Prcb             : _KPRCB
```

```c
VOID KeReadyThread (IN PKTHREAD Thread) {
    
    KIRQL OldIrql;
    PKPROCESS Process;
    KSTACK_COUNT StackCount;

    Process = Thread->ApcState.Process;
    StackCount.Value = Process->StackCount.Value;

    if (StackCount.State != ProcessInMemory) {
        OldIrql = KeRaiseIrqlToDpcLevel();
        if (KiInSwapSingleProcess(Thread, Process, OldIrql)) {
            return;
        }
    }
    KiFastReadyThread(Thread);
}
```

- `KeRaiseIrqlToDpcLevel()` seems to be inline inside `KeReadyThread()` like many other cases.
- Generally, cr8 uses the four low-order bits for specifying a task priority and the remaining 60 bits are reserved and must be written with zeros.
- In Windows, cr8 hols the current IRQL, so `mov rax, cr8` is like calling `KeGetCurrentIrql()`
- `#define DISPATCH_LEVEL 2  // Dispatcher level`

```c
KIRQL KeRaiseIrqlToDpcLevel(KIRQL NewIrql) {

    KIRQL OldIrql;

    OldIrql = KeGetCurrentIrql();

    __writecr8(DISPATCH_LEVEL);

    if (KiIrqlFlags && KiIrqlFlags & 1 != 0 && OldIrql < DISPATCH_LEVEL) {
        _InterlockedOr(KeGetCurrentPrcb()->SchedulerAssist, 0x10000);
    }

    return OldIrql;
}
```
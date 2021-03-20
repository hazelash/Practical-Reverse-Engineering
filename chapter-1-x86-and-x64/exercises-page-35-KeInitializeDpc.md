# KeInitializeDpc

_Decompile the following kernel routine in Windows:_

```asm
text:000000014000AD80 ; void __stdcall KeInitializeDpc(PRKDPC Dpc, PKDEFERRED_ROUTINE DeferredRoutine, PVOID DeferredContext)
.text:000000014000AD80                 public KeInitializeDpc
.text:000000014000AD80 KeInitializeDpc proc near               ; CODE XREF: PopEndMirroring+14D↓p
.text:000000014000AD80                                         ; PfpStartLoggingHardFaultEvents+9B↓p ...
.text:000000014000AD80                 xor     eax, eax
.text:000000014000AD82                 mov     dword ptr [rcx], 113h
.text:000000014000AD88                 mov     [rcx+38h], rax
.text:000000014000AD8C                 mov     [rcx+10h], rax
.text:000000014000AD90                 mov     [rcx+18h], rdx
.text:000000014000AD94                 mov     [rcx+20h], r8
.text:000000014000AD98                 retn
.text:000000014000AD98 KeInitializeDpc endp
```

```c
dll!_KDPC
   +0x000 TargetInfoAsUlong : Uint4B
   +0x000 Type             : UChar
   +0x001 Importance       : UChar
   +0x002 Number           : Uint2B
   +0x008 DpcListEntry     : _SINGLE_LIST_ENTRY
   +0x010 ProcessorHistory : Uint8B
   +0x018 DeferredRoutine  : Ptr64     void 
   +0x020 DeferredContext  : Ptr64 Void
   +0x028 SystemArgument1  : Ptr64 Void
   +0x030 SystemArgument2  : Ptr64 Void
   +0x038 DpcData          : Ptr64 Void


VOID KeInitializeDpc(PRKDPC Dpc, PKDEFERRED_ROUTINE DeferredRoutine, PVOID DeferredContext) {
   Dpc->TargetInfoAsUlong = 0x113;
   Dpc->DpcData = 0;
   Dpc->ProcessorHistory = 0;
   Dpc->DeferredRoutine = DeferredRoutine;
   Dpc->DeferredContext = DeferredContext;
}
```

- why TargetInfoAsUlong is specifically set to a value of 0x113 ? Looking back at the struct definition in WinDBG, we can see that `TargetInfoAsUlong` is:
```c
dll!_KDPC
   +0x000 TargetInfoAsUlong : Uint4B
      +0x000 Type             : UChar
      +0x001 Importance       : UChar
      +0x002 Number           : Uint2B
```      

- This leaves us with Type to 0x13, Importance to 0x1 and Number to 0x0. 
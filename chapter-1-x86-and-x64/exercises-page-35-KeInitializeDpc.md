# KeInitializeDpc

_Decompile the following kernel routine in Windows:_

```
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
```
dll!_KDPC
   +0x000 TargetInfoAsUlong : Uint4B
      +0x000 Type             : UChar
      +0x001 Importance       : UChar
      +0x002 Number           : Uint2B
```      

- This leaves us with Type to 0x13, Importance to 0x1 and Number to 0x0. 
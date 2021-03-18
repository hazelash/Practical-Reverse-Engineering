# KeInitializeApc

_Decompile the following kernel routine in Windows:_

```asm
.text:0000000140138420 ; =============== S U B R O U T I N E =======================================
.text:0000000140138420
.text:0000000140138420
.text:0000000140138420                 public KeInitializeApc
.text:0000000140138420 KeInitializeApc proc near               ; CODE XREF: ExpSetTimerObject+806↑p
.text:0000000140138420                                         ; EtwpEventWriteFull+FCA↑p ...
.text:0000000140138420
.text:0000000140138420 arg_20          = qword ptr  28h
.text:0000000140138420 arg_28          = qword ptr  30h
.text:0000000140138420 arg_30          = byte ptr  38h
.text:0000000140138420 arg_38          = qword ptr  40h
.text:0000000140138420
.text:0000000140138420                 mov     byte ptr [rcx], 12h
.text:0000000140138423                 mov     r10, rcx
.text:0000000140138426                 mov     byte ptr [rcx+2], 58h ; 'X'
.text:000000014013842A                 cmp     r8d, 2
.text:000000014013842E                 jz      short loc_140138473
.text:0000000140138430
.text:0000000140138430 loc_140138430:                          ; CODE XREF: KeInitializeApc+5A↓j
.text:0000000140138430                 mov     rax, [rsp+arg_20]
.text:0000000140138435                 mov     [rcx+50h], r8b
.text:0000000140138439                 mov     [rcx+28h], rax
.text:000000014013843D                 mov     [rcx+8], rdx
.text:0000000140138441                 mov     rdx, [rsp+arg_28]
.text:0000000140138446                 mov     [rcx+30h], rdx
.text:000000014013844A                 mov     rax, rdx
.text:000000014013844D                 neg     rax
.text:0000000140138450                 mov     [rcx+20h], r9
.text:0000000140138454                 sbb     rcx, rcx
.text:0000000140138457                 and     rcx, [rsp+arg_38]
.text:000000014013845C                 neg     rdx
.text:000000014013845F                 sbb     al, al
.text:0000000140138461                 and     al, [rsp+arg_30]
.text:0000000140138465                 mov     [r10+51h], al   ; ApcMode
.text:0000000140138469                 mov     [r10+38h], rcx  ; NormalRoutine
.text:000000014013846D                 mov     byte ptr [r10+52h], 0
.text:0000000140138472                 retn
.text:0000000140138473 ; ---------------------------------------------------------------------------
.text:0000000140138473
.text:0000000140138473 loc_140138473:                          ; CODE XREF: KeInitializeApc+E↑j
.text:0000000140138473                 mov     r8b, [rdx+24Ah]
.text:000000014013847A                 jmp     short loc_140138430
.text:000000014013847A KeInitializeApc endp
.text:000000014013847A
.text:000000014013847A ; ---------------------------------------------------------------------------
```

- At the first attempt, when we decompiled this function, we did a manual asm to C translation for the following asm instructions:

```asm
  neg     rax
.text:0000000140138450                 mov     [rcx+20h], r9
.text:0000000140138454                 sbb     rcx, rcx
.text:0000000140138457                 and     rcx, [rsp+arg_38]
.text:000000014013845C                 neg     rdx
.text:000000014013845F                 sbb     al, al
.text:0000000140138461                 and     al, [rsp+arg_30]
.text:0000000140138465                 mov     [r10+51h], al   ; ApcMode
.text:0000000140138469                 mov     [r10+38h], rcx  ; NormalRoutine
```

- the output was not looking like what the authors would have originally written, though it is functional.
- looking a bit closer, we learned that if you have an if/else clause and setting the values inside to either 0 or 1, that might be  translated to assembly as a neg + sbb.
- sbb: subtracts the source from the destination, and subtracts 1 extra if the CF is set.
- neg: 0 - reg, CF is affected: if(Destination == 0) CF = 0; else CF = 1.


```c
// From ReactOS
typedef enum _MODE {
    KernelMode,
    UserMode,
    MaximumMode
} MODE;


ntdll!_KAPC
   +0x000 Type             : UChar
   +0x001 SpareByte0       : UChar
   +0x002 Size             : UChar
   +0x003 SpareByte1       : UChar
   +0x004 SpareLong0       : Uint4B
   +0x008 Thread           : Ptr64 _KTHREAD
   +0x010 ApcListEntry     : _LIST_ENTRY
   +0x020 KernelRoutine    : Ptr64     void 
   +0x028 RundownRoutine   : Ptr64     void 
   +0x030 NormalRoutine    : Ptr64     void 
   +0x020 Reserved         : [3] Ptr64 Void
   +0x038 NormalContext    : Ptr64 Void
   +0x040 SystemArgument1  : Ptr64 Void
   +0x048 SystemArgument2  : Ptr64 Void
   +0x050 ApcStateIndex    : Char
   +0x051 ApcMode          : Char
   +0x052 Inserted         : UChar

0:000> dt nt!_KTHREAD
ntdll!_KTHREAD
   +0x000 Header           : _DISPATCHER_HEADER
   +0x018 SListFaultAddress : Ptr64 Void
    ...
   +0x24a ApcStateIndex    : UChar
   +0x24b WaitBlockCount   : UChar
    ...
   +0x5f8 AbWaitObject     : Ptr64 Void

VOID KeInitializeApc(
  IN  PRKAPC Apc,
  IN  PRKTHREAD Thread,
  IN  KAPC_ENVIRONMENT Environment,
  IN  PKKERNEL_ROUTINE KernelRoutine,
  IN  PKRUNDOWN_ROUTINE RundownRoutine OPTIONAL,
  IN  PKNORMAL_ROUTINE NormalRoutine OPTIONAL,
  IN  KPROCESSOR_MODE ApcMode OPTIONAL,
  IN  PVOID NormalContext OPTIONAL)
{
    Apc->Type = 0x12;
    Apc->size = 0x58;

    if (Environment == CurrentApcEnvironment ) {
        Apc->ApcStateIndex = Thread->ApcStateIndex;
    } else {
        Apc->ApcStateIndex = Environment;
    }

    Apc->RundownRoutine = RundownRoutine;
    Apc->Thread = Thread;
    Apc->NormalRoutine = NormalRoutine;
    Apc->KernelRoutine = KernelRoutine;

    if (NormalRoutine  != NULL) {
        Apc->ApcMode = UserMode;
        Apc->NormalContext = NULL;
    } else {
        Apc->ApcMode = KernelMode;
        Apc->NormalContext = NormalContext;
    }

    Apc->Inserted = 0;
}
```
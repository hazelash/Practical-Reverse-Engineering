# KeInitializeQueue

_Decompile the following kernel routine in Windows:_

```
ntdll!_KQUEUE
   +0x000 Header           : _DISPATCHER_HEADER
   +0x018 EntryListHead    : _LIST_ENTRY
   +0x028 CurrentCount     : Uint4B
   +0x02c MaximumCount     : Uint4B
   +0x030 ThreadListHead   : _LIST_ENTRY

0:000> dt nt!_DISPATCHER_HEADER
ntdll!_DISPATCHER_HEADER
   +0x000 Lock             : Int4B
   +0x000 LockNV           : Int4B
   +0x000 Type             : UChar
   +0x001 Signalling       : UChar
   +0x002 Size             : UChar
   +0x003 Reserved1        : UChar
   +0x000 TimerType        : UChar
   +0x001 TimerControlFlags : UChar
   +0x001 Absolute         : Pos 0, 1 Bit
   +0x001 Wake             : Pos 1, 1 Bit
   +0x001 EncodedTolerableDelay : Pos 2, 6 Bits
   +0x002 Hand             : UChar
   +0x003 TimerMiscFlags   : UChar
   +0x003 Index            : Pos 0, 6 Bits
   +0x003 Inserted         : Pos 6, 1 Bit
   +0x003 Expired          : Pos 7, 1 Bit
   +0x000 Timer2Type       : UChar
   +0x001 Timer2Flags      : UChar
   +0x001 Timer2Inserted   : Pos 0, 1 Bit
   +0x001 Timer2Expiring   : Pos 1, 1 Bit
   +0x001 Timer2CancelPending : Pos 2, 1 Bit
   +0x001 Timer2SetPending : Pos 3, 1 Bit
   +0x001 Timer2Running    : Pos 4, 1 Bit
   +0x001 Timer2Disabled   : Pos 5, 1 Bit
   +0x001 Timer2ReservedFlags : Pos 6, 2 Bits
   +0x002 Timer2ComponentId : UChar
   +0x003 Timer2RelativeId : UChar
   +0x000 QueueType        : UChar
   +0x001 QueueControlFlags : UChar
   +0x001 Abandoned        : Pos 0, 1 Bit
   +0x001 DisableIncrement : Pos 1, 1 Bit
   +0x001 QueueReservedControlFlags : Pos 2, 6 Bits
   +0x002 QueueSize        : UChar
   +0x003 QueueReserved    : UChar
   +0x000 ThreadType       : UChar
   +0x001 ThreadReserved   : UChar
   +0x002 ThreadControlFlags : UChar
   +0x002 CycleProfiling   : Pos 0, 1 Bit
   +0x002 CounterProfiling : Pos 1, 1 Bit
   +0x002 GroupScheduling  : Pos 2, 1 Bit
   +0x002 AffinitySet      : Pos 3, 1 Bit
   +0x002 Tagged           : Pos 4, 1 Bit
   +0x002 EnergyProfiling  : Pos 5, 1 Bit
   +0x002 SchedulerAssist  : Pos 6, 1 Bit
   +0x002 ThreadReservedControlFlags : Pos 7, 1 Bit
   +0x003 DebugActive      : UChar
   +0x003 ActiveDR7        : Pos 0, 1 Bit
   +0x003 Instrumented     : Pos 1, 1 Bit
   +0x003 Minimal          : Pos 2, 1 Bit
   +0x003 Reserved4        : Pos 3, 3 Bits
   +0x003 UmsScheduled     : Pos 6, 1 Bit
   +0x003 UmsPrimary       : Pos 7, 1 Bit
   +0x000 MutantType       : UChar
   +0x001 MutantSize       : UChar
   +0x002 DpcActive        : UChar
   +0x003 MutantReserved   : UChar
   +0x004 SignalState      : Int4B
   +0x008 WaitListHead     : _LIST_ENTRY


VOID KeInitializeQueue(PRKQUEUE Queue, ULONG Count) {

    LIST_ENTRY Entry, Head; // sub     rsp, 28h

    Queue->Header.Type = QueueObject; // 4
    Queue->Header.Size = sizeof(KQUEUE) / sizeof(LONG); // 0x1000; 
    Queue->SignalState = 0; 

    /* Init WaitListHead
    * Entry.FLink = Queue->Header.WaitListHead;
    * Entry.Blink = Queue->Header.WaitListHead;
    * Queue->Header.WaitListHead = Entry;
    /*
    InitializeListHead(&Queue->Header.WaitListHead);

    /* Init EntryListHead
    * Entry.FLink = Queue->EntryListHead;
    * Entry.Blink = Queue->EntryListHead;
    * Queue.EntryListHead = Entry;
    /*
    InitializeListHead(&Queue->EntryListHead);

    /* Init ThreadListHead
    * Entry.FLink = Queue->ThreadListHead;
    * Entry.Blink = Queue->ThreadListHead;
    * Queue.ThreadListHead = Entry;
    /*
    InitializeListHead(&Queue->ThreadListHead);

    Queue->CurrentCount = 0;
    if(!Queue->CurrentCount) {
        Queue->MaximumCount = KeQueryActiveProcessorCountEx(ALL_PROCESSOR_GROUPS); // 0xffff
    } else {
        Queue->MaximumCount = Count;
    }
}
```
# KxWaitForLockChainValid

_Decompile the following kernel routine in Windows:_

```
.text:0000000140003200 ; =============== S U B R O U T I N E =======================================
.text:0000000140003200
.text:0000000140003200
.text:0000000140003200 KxWaitForLockChainValid proc near       ; CODE XREF: CcCanIWrite+4C4↑p
.text:0000000140003200                                         ; CcCanIWrite+4E1↑p ...
.text:0000000140003200
.text:0000000140003200 arg_0           = qword ptr  8
.text:0000000140003200
.text:0000000140003200 ; FUNCTION CHUNK AT .text:00000001401E00B0 SIZE 00000028 BYTES
.text:0000000140003200
.text:0000000140003200                 mov     [rsp+arg_0], rbx
.text:0000000140003205                 push    rdi
.text:0000000140003206                 sub     rsp, 20h
.text:000000014000320A                 mov     rdi, rcx
.text:000000014000320D                 xor     ebx, ebx
.text:000000014000320F
.text:000000014000320F loc_14000320F:                          ; CODE XREF: KxWaitForLockChainValid+25↓j
.text:000000014000320F                 inc     ebx
.text:0000000140003211                 test    cs:HvlLongSpinCountMask, ebx
.text:0000000140003217                 jz      loc_1401E00B0
.text:000000014000321D
.text:000000014000321D loc_14000321D:                          ; CODE XREF: KxWaitForLockChainValid+1DCEB8↓j
.text:000000014000321D                                         ; KxWaitForLockChainValid+1DCEC5↓j
.text:000000014000321D                 pause
.text:000000014000321F
.text:000000014000321F loc_14000321F:                          ; CODE XREF: KxWaitForLockChainValid+1DCED3↓j
.text:000000014000321F                 mov     rax, [rdi]
.text:0000000140003222                 test    rax, rax
.text:0000000140003225                 jz      short loc_14000320F
.text:0000000140003227                 mov     rbx, [rsp+28h+arg_0]
.text:000000014000322C                 add     rsp, 20h
.text:0000000140003230                 pop     rdi
.text:0000000140003231                 retn
.text:0000000140003231 KxWaitForLockChainValid endp
```

```
ntdll!_KSPIN_LOCK_QUEUE
   +0x000 Next             : Ptr64 _KSPIN_LOCK_QUEUE
   +0x008 Lock             : Ptr64 Uint8B
```

### few notes:

- `YieldProcessor` macro definition varies from platform to platform. The following are some definitions of this :
    - define YieldProcessor() __asm { rep nop }
    - #define YieldProcessor _mm_pause
    - #define YieldProcessor __yield
- [How do you use the pause assembly instruction in 64-bit C++ code?](https://stackoverflow.com/questions/5833527/)
- [How does x86 pause instruction work in spinlock *and* can it be used in other scenarios?](https://stackoverflow.com/questions/4725676/how-does-x86-pause-instruction-work-in-spinlock-and-can-it-be-used-in-other-sc#:~:text=4%20Answers&text=PAUSE%20notifies%20the%20CPU%20that,some%20time%20to%20save%20power.)


```
PKSPIN_LOCK_QUEUE KxWaitForLockChainValid(__INOUT PKSPIN_LOCK_QUEUE LockQueue) {

    PKSPIN_LOCK_QUEUE NextQueue;
    ULONG SpinCount = 0

    do {
        SpinCount++;
        if ((spinCount & HvlLongSpinCountMask) && (HvlEnlightenments & 0x40 != 0) 
            && (KiCheckVpBackingLongSpinWaitHypercall())) {
            // notify the hypervisor that the calling virtual processor is
            // attempting to acquire a resource that is potentially held by
            //  another virtual processor within the same partition.
            HvlNotifyLongSpinWait(SpinCount);
        } else {
            KeYieldProcessor();
        }

        NextQueue = LockQueue->Next;
    } while (NextQueue == NULL);
 
    return NextQueue;
}
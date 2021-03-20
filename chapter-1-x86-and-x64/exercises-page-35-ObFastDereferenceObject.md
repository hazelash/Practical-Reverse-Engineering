# ObFastDereferenceObject

_Decompile the following kernel routine in Windows:_

```asm
text:000000014010E700 ; =============== S U B R O U T I N E =======================================
.text:000000014010E700
.text:000000014010E700
.text:000000014010E700 ObFastReferenceObject proc near         ; CODE XREF: CcReferenceSharedCacheMapFileObject+D↑p
.text:000000014010E700                                         ; MiMakeSystemCacheRangeValid+555↑p ...
.text:000000014010E700
.text:000000014010E700 BugCheckParameter4= qword ptr -18h
.text:000000014010E700 arg_0           = qword ptr  8
.text:000000014010E700
.text:000000014010E700 ; FUNCTION CHUNK AT .text:0000000140260918 SIZE 00000058 BYTES
.text:000000014010E700
.text:000000014010E700                 mov     [rsp+arg_0], rbx
.text:000000014010E705                 push    rdi
.text:000000014010E706                 sub     rsp, 30h
.text:000000014010E70A                 mov     rbx, rcx
.text:000000014010E70D                 prefetchw byte ptr [rcx]
.text:000000014010E710                 mov     r8, [rcx]
.text:000000014010E713                 test    r8b, 0Fh
.text:000000014010E717                 jz      short loc_14010E72B
.text:000000014010E719
.text:000000014010E719 loc_14010E719:                          ; CODE XREF: ObFastReferenceObject+101↓j
.text:000000014010E719                 lea     rdx, [r8-1]
.text:000000014010E71D                 mov     rax, r8
.text:000000014010E720                 lock cmpxchg [rcx], rdx
.text:000000014010E725                 jnz     loc_14010E7FC
.text:000000014010E72B
.text:000000014010E72B loc_14010E72B:                          ; CODE XREF: ObFastReferenceObject+17↑j
.text:000000014010E72B                                         ; ObFastReferenceObject+107↓j
.text:000000014010E72B                 mov     rdi, r8
.text:000000014010E72E                 and     r8d, 0Fh
.text:000000014010E732                 and     rdi, 0FFFFFFFFFFFFFFF0h
.text:000000014010E736                 cmp     r8d, 1
.text:000000014010E73A                 jbe     short loc_14010E74A
.text:000000014010E73C
.text:000000014010E73C loc_14010E73C:                          ; CODE XREF: ObFastReferenceObject+8B↓j
.text:000000014010E73C                                         ; ObFastReferenceObject+C5↓j ...
.text:000000014010E73C                 mov     rax, rdi
.text:000000014010E73F                 mov     rbx, [rsp+38h+arg_0]
.text:000000014010E744                 add     rsp, 30h
.text:000000014010E748                 pop     rdi
.text:000000014010E749                 retn
.text:000000014010E74A ; ---------------------------------------------------------------------------
```

```c
0:000> dt nt!_EX_FAST_REF
ntdll!_EX_FAST_REF
   +0x000 Object           : Ptr64 Void
   +0x000 RefCnt           : Pos 0, 4 Bits
   +0x000 Value            : Uint8B

typedef struct _EX_FAST_REF
{
    union
    {
        void *Object;
        unsigned __int64 RefCnt : 4;
        unsigned __int64 Value;
    };
} EX_FAST_REF, *PEX_FAST_REF;

#define MAX_FAST_REFS 15

VOID ObFastDereferenceObject(IN PEX_FAST_REF FastRef, IN PVOID Object) {

    EX_FAST_REF NewFastRef, OldFastRef;

    while(TRUE) {

        OldFastRef = *FastRef;

        if (OldFastRef.Value ^ (ULONG_PTR)Object >= MAX_FAST_REFS) {
            return;
        }

        NewFastRef.Value = OldFastRef.Value + 1;
        InterlockedCompareExchangePointer(
            &FastRef->Object, NewFastRef.Value, OldFastRef.Value);

        if (NewFastRef.Value == OldFastRef.Value) {
            return;
        }
    }

    ObfDereferenceObject(Object);
}
```

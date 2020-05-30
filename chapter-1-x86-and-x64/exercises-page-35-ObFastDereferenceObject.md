# ObFastDereferenceObject

_Decompile the following kernel routine in Windows:_

```
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

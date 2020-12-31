# RtlValidateUnicodeString

_Decompile the following kernel routine in Windows:_

```assembly
.text:000000014013CC30 RtlValidateUnicodeString proc near      ; CODE XREF: RtlDuplicateUnicodeString+57↓p
.text:000000014013CC30                 xor     eax, eax
.text:000000014013CC32                 test    ecx, ecx
.text:000000014013CC34                 jnz     short loc_14013CC72
.text:000000014013CC36                 test    rdx, rdx
.text:000000014013CC39                 jz      short locret_14013CC66
.text:000000014013CC3B                 movzx   r8d, word ptr [rdx]
.text:000000014013CC3F                 test    r8b, 1
.text:000000014013CC43                 jnz     short loc_14013CC72
.text:000000014013CC45                 movzx   ecx, word ptr [rdx+2]
.text:000000014013CC49                 test    cl, 1
.text:000000014013CC4C                 jnz     short loc_14013CC72
.text:000000014013CC4E                 cmp     r8w, cx
.text:000000014013CC52                 ja      short loc_14013CC72
.text:000000014013CC54                 mov     r9d, 0FFFEh
.text:000000014013CC5A                 cmp     cx, r9w
.text:000000014013CC5E                 ja      short loc_14013CC72
.text:000000014013CC60                 cmp     [rdx+8], rax
.text:000000014013CC64                 jz      short loc_14013CC67
.text:000000014013CC66
.text:000000014013CC66 locret_14013CC66:                       ; CODE XREF: RtlValidateUnicodeString+9↑j
.text:000000014013CC66                                         ; RtlValidateUnicodeString+40↓j
.text:000000014013CC66                 retn
.text:000000014013CC67 ; ---------------------------------------------------------------------------
.text:000000014013CC67
.text:000000014013CC67 loc_14013CC67:                          ; CODE XREF: RtlValidateUnicodeString+34↑j
.text:000000014013CC67                 test    r8w, r8w
.text:000000014013CC6B                 jnz     short loc_14013CC72
.text:000000014013CC6D                 test    cx, cx
.text:000000014013CC70                 jz      short locret_14013CC66
.text:000000014013CC72
.text:000000014013CC72 loc_14013CC72:                          ; CODE XREF: RtlValidateUnicodeString+4↑j
.text:000000014013CC72                                         ; RtlValidateUnicodeString+13↑j ...
.text:000000014013CC72                 mov     eax, 0C000000Dh
.text:000000014013CC77                 retn
.text:000000014013CC77 RtlValidateUnicodeString endp
```

```c
NTSTATUS NTAPI
RtlValidateUnicodeString(IN ULONG Flags, IN PCUNICODE_STRING UnicodeString)
{
    NTSTATUS Status = STATUS_SUCCESS;
    if (Flags)
    {
        Status = STATUS_INVALID_PARAMETER;
    }
    else
    {
        if (UnicodeString)
        {
            if ( (UnicodeString->Length % 2 != 0)  ||
                (UnicodeString->MaximumLength % 2 != 0 ) ||
                (UnicodeString->Length > UnicodeString->MaximumLength) ||
                (UnicodeString->MaximumLength > 0xfffe)) {
                Status = STATUS_INVALID_PARAMETER;
            } 

            else if (UnicodeString->Buffer == NULL &&
                UnicodeString->Length != 0  &&
                UnicodeString->MaximumLength != 0) {
                Status = STATUS_INVALID_PARAMETER;
            }
        }
    }

    return Status;
}
```
# chapter 1 exercise (page 11)

_This function uses a combination SCAS and STOS to do its work. First, explain what is the type of the [EBP+8] and [EBP+C] in line 1 and 8, respectively. Next, explain what this snippet does._

```
01: 8B 7D 08 	mov edi, [ebp+8] // *(ebp+8) = edi
02: 8B D7 		mov edx, edi // edx = edi
03: 33 C0 		xor eax, eax // eax = 0
04: 83 C9 FF 	or ecx, 0FFFFFFFFh // ecx = 0xFFFFFFFF
05: F2 AE 		repne scasb // search for \x00
06: 83 C1 02 	add ecx, 2 // ecx += 2
07: F7 D9 		neg ecx  // 2’s complement, ecx = strlen(buffer)
08: 8A 45 0C 	mov al, [ebp+0Ch] // al = the byte we want to set.
09: 8B FA 		mov edi, edx // edi = edx = *(ebp+8)
10: F3 AA 		rep stosb // memset()
11: 8B C2 		mov eax, edx // eax = edx =  *(ebp+8)
```

_What is the type of [EBP+8] and [EBP+C]?_
- type of [EBP+8] = char*
- type of [EBP+C] = char
- [ebp+8] is copied to EDI to be prepared for the `rep stosb`, this indicates that [ebp+8] is the destination string that will be used in memset(), and the [ebp+0xC] is the value we want to initialize the array of bytes to.

_Next, explain what this snippet does._

- Read comments to understand each line.
- This snippet is similar to calling:
`memset( dest_buffer, init_value, strlen(dest_buffer) );` Where *dest_buffer* is ECX+0 and *init_value* is ECX+C.

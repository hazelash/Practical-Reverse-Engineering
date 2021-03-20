# chapter 1 exercise (page 35)

_Repeat the walk-through by yourself. Draw the stack layout, including parameters and local variables._

- (1) Will return after was DllMain() called.
- (2) Base frame pointer of the function that called DllMain().
- (3) szExeFile is an array of bytes with MAX_PATH (260 bytes) as size.

```
      LOW ADDRESS
          ^ ^
         | | |
+---------------------+ <-- ebp - 0x134 (esp)
|         edi         |
+---------------------+ <-- ebp - 0x130 (procentry)
|        dwSize       | 
+---------------------+ <-- ebp - 0x12c
|       cntUsage      |
+---------------------+ <-- ebp - 0x128
|    th32ProcessID    |
+---------------------+ <-- ebp - 0x124
|  th32DefaultHeapID  |
+---------------------+ <-- ebp - 0x120
|     th32ModuleID    |
+---------------------+ <-- ebp - 0x11c
|      cntThreads     |
+---------------------+ <-- ebp - 0x118
| th32ParentProcessID |
+---------------------+ <-- ebp - 0x114
|    pcPriClassBase   |
+---------------------+ <-- ebp - 0x110
|       dwFlags       |
+---------------------+ <-- ebp - 0x10c (3)
|      szExeFile      |
+---------------------+ <-- ebp - 8
|         @sidt       |
+---------------------+ <-- ebp - 4
|                     |
+---------------------+ <-- ebp
|        ebp(2)       |
+---------------------+
|       RETN(1)       |
+---------------------+
|       hinstDLL      |
+---------------------+ <--ebp+0xc
|      fdwReason      | 
+---------------------+
|     lpvReserved     |
+---------------------+
         | | |
      HIGH ADDRESS
```

_In the example walk-through, we did a nearly one-to-one translation of the assembly code to C. As an exercise, re-decompile this whole function so that it looks more natural._

```c
typedef struct _IDTR
{
   DWORD base;
   SHORT limit;
} IDTR, *PIDTR;

BOOL __stdcall DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) 
{
  // declare some vars.
	IDTR idtr;
	HANDLE hSnapshot;
	PROCESSENTRY32 procentry;

	// get interrupt descriptor table register.
	__sidt(&idtr);

	if (idtr.base > 0x8003F400 && idtr.base <= 0x80047400) {
		return FALSE;
	}

	hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
	if (hSnapshot == INVALID_HANDLE_VALUE) {
		return FALSE;
	}

	memset(&procentry, 0, sizeof(PROCESSENTRY32));
	procentry.dwSize = sizeof(procentry);

	if (Process32First(hSnapshot, &procentry)) {
		if (_stricmp(procentry.szExeFile, "explorer.exe") != 0) {
			while (Process32Next(hSnapshot, &procentry)) {
				if (_stricmp(procentry.szExeFile, "explorer.exe") == 0) {
					if (procentry.th32ParentProcessID == procentry.th32ProcessID) {
						return FALSE;
					}
					goto start_thread;
				}
			}
		}
		if (procentry.th32ParentProcessID == procentry.th32ProcessID) {
			return FALSE;
		}
	}

	if (fdwReason == DLL_PROCESS_DETACH) {
		return FALSE;
	}

start_thread:
	CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)0x100032D0, NULL, 0, NULL);
	return TRUE;
}

```

_What can you say about the developer’s skill level/experience? Explain your reasons. Can you do a better job?_

- developer skill/experience: not bad for trying to detect the VM, however, the VM check is not reliable due to hardcoding IDT bases + subject to change.
- can I do a better job ? 
   1. vague question knowing the purpose of this routine is pretty small.
   2. a better job in which sense ?
This routine tries to detect the VM, check if `explorer.exe` is running and if so, create a thread that infects the target machine. In our humble opinion, the author did two mistakes: the virtualization environment check is not realiable, and more importantly, the usage of the instruction `sidt` make it more fishy. We rarely see this instruction in legitimate user mode apps, that would raise a red flag for heuristics engines. This can be improved by using a less known one (in the sense that AVs would not have knowledge or code that can see it) without sacrificing reliability. We can also use NtQuerySystemInformation to iterate over running processes hoping the ntdll API is not being hooked by the sandbox.

_In some of the assembly listings, the function name has a @ prefix followed by a number. Explain when and why this decoration exists_

- The decoration is referred as: __mangling__, that is a scheme used in Microsoft's Visual C++ series of compilers. It provides a way of encoding the name and additional information about a function, structure, class or another datatype in order to pass more semantic information from the Microsoft Visual C++ compiler to its linker.
- In the stdcall and fastcall mangling schemes, the function is encoded as _name@X and @name@X respectively, where X is the number of bytes, in decimal, of the argument(s) in the parameter list (including those passed in registers, for fastcall). In the case of cdecl, the function name is merely prefixed by an underscore.

_Implement the following functions in x86 assembly: strlen, strchr, memcpy, memset, strcmp, strset._

## strlen

```asm
_Strlen Proc Src:Ptr Byte
	Xor Eax, Eax
	Mov Esi, [Src]
	cmp esi, 0
	je exit
@@:
	Mov Cl, Byte Ptr [Esi + Eax]
	Inc Eax
	Cmp Cl, 0
	Jnz @B
	Dec Eax
exit:
	Ret
_Strlen EndP
```

- Or:

```asm
_Strlen2 Proc Src :Ptr Byte
	Xor Eax, Eax
	Mov Edi, [Src]
	cmp Edi, 0		; check null pointer
	jz exit
	Mov Edx, Edi
	Mov Ecx, -1
	Repne Scasb
	Sub Edi, Edx 	; we can also do a add ecx, 2
	Dec Edi			; then neg ecx
	Mov Eax, Edi
exit:
	Ret
_Strlen2 EndP
```

- There is a faster implementation of this which you would find in ntdll!strlen or glibc implementation.
- It takes a string one dword at a time rather than one byte at a time and then performs some strange maths on it which will turn all 0s in a string into 0x80s and all other characters into 0x00s. For example (x86) version:
	- A word from our string: 00474d43
	- Gets turned into:       80000000
- Here is its disassembly:

```asm
.text:4B2F9000 _strlen         proc near               ; DATA XREF: .text:4B283505↑o
.text:4B2F9000                                         ; .text:off_4B3873D8↓o
.text:4B2F9000
.text:4B2F9000 Str             = dword ptr  4
.text:4B2F9000
.text:4B2F9000                 mov     ecx, [esp+Str]
.text:4B2F9004                 test    ecx, 3
.text:4B2F900A                 jz      short loc_4B2F9030
.text:4B2F900C
.text:4B2F900C loc_4B2F900C:                           ; CODE XREF: _strlen+1B↓j
.text:4B2F900C                 mov     al, [ecx]
.text:4B2F900E                 add     ecx, 1
.text:4B2F9011                 test    al, al
.text:4B2F9013                 jz      short loc_4B2F9063
.text:4B2F9015                 test    ecx, 3
.text:4B2F901B                 jnz     short loc_4B2F900C
.text:4B2F901D                 add     eax, 0
.text:4B2F9022                 lea     esp, [esp+0]
.text:4B2F9029                 lea     esp, [esp+0]
.text:4B2F9030
.text:4B2F9030 loc_4B2F9030:                           ; CODE XREF: _strlen+A↑j
.text:4B2F9030                                         ; _strlen+46↓j ...
.text:4B2F9030                 mov     eax, [ecx]
.text:4B2F9032                 mov     edx, 7EFEFEFFh
.text:4B2F9037                 add     edx, eax
.text:4B2F9039                 xor     eax, 0FFFFFFFFh
.text:4B2F903C                 xor     eax, edx
.text:4B2F903E                 add     ecx, 4
.text:4B2F9041                 test    eax, 81010100h
.text:4B2F9046                 jz      short loc_4B2F9030
.text:4B2F9048                 mov     eax, [ecx-4]
.text:4B2F904B                 test    al, al
.text:4B2F904D                 jz      short loc_4B2F9081
.text:4B2F904F                 test    ah, ah
.text:4B2F9051                 jz      short loc_4B2F9077
.text:4B2F9053                 test    eax, 0FF0000h
.text:4B2F9058                 jz      short loc_4B2F906D
.text:4B2F905A                 test    eax, 0FF000000h
.text:4B2F905F                 jz      short loc_4B2F9063
.text:4B2F9061                 jmp     short loc_4B2F9030
.text:4B2F9063 ; ---------------------------------------------------------------------------
.text:4B2F9063
.text:4B2F9063 loc_4B2F9063:                           ; CODE XREF: _strlen+13↑j
.text:4B2F9063                                         ; _strlen+5F↑j
.text:4B2F9063                 lea     eax, [ecx-1]
.text:4B2F9066                 mov     ecx, [esp+Str]
.text:4B2F906A                 sub     eax, ecx
.text:4B2F906C                 retn
.text:4B2F906D ; ---------------------------------------------------------------------------
.text:4B2F906D
.text:4B2F906D loc_4B2F906D:                           ; CODE XREF: _strlen+58↑j
.text:4B2F906D                 lea     eax, [ecx-2]
.text:4B2F9070                 mov     ecx, [esp+Str]
.text:4B2F9074                 sub     eax, ecx
.text:4B2F9076                 retn
```

## strchr

```asm
_StrChr Proc Src:Ptr Byte, Value:Byte
	Cld
	Xor Eax, Eax
	Mov Edi, [Src]
	Cmp Edi, 0 				; check for NULL ptr
	Je exit

	Mov Bl, Value
	Xor Ecx, Ecx
@@:
	Mov Dl, Byte Ptr [Edi]
	Cmp Dl, Bl 				; did we find our char
	Je exit
	Scasb 					; did we reach the end of the string
	Jne @B
	mov Edi, 0 				; char not found, return NULL
exit:
	Mov Eax, Edi 			; set eax to first occurance
	Ret
```

## memcpy

```asm
_MemCpy Proc Dest:Ptr Byte, Src:Ptr Byte, Len:DWord
	Cld
    Mov Edi, [Dest]
    Mov Eax, Edi                ; return original destination pointer
    Mov Esi, [Src]
    Mov Ecx, Len
	Rep Movsb

	; Shr Ecx, 2				; divide by 4
    ; rep movsd					; instead of doing byte by byte, we do it by DW

    ; mov ecx, [ln]
    ; and ecx, 3     			; a % 4 = a & ( 4 - 1 )
    ; Rep Movsb					; copy the left over bytes
    ret
_MemCpy EndP
```

## memset

```asm
_MemSet Proc Buffer:Ptr Byte, Value:Byte, Len:DWord
	Cld
	Xor Eax, Eax
	Mov Ecx, Len
	Mov Al, Value
	Mov Edi, [Buffer]
	Rep Stosb
	Ret
_MemSet EndP
```

## strcmp

```asm
_StrCmp Proc X:Ptr Byte, Y:Ptr Byte
	Xor Ecx, Ecx
	@@:
	Mov Bl, Byte Ptr [Str1 + Ecx]
	Inc Ecx
	Cmp Bl, 0
	Jnz @B
	Dec Ecx

	Cld
    Mov Esi, [X]
    Mov Edi, [Y]
	Repe Cmpsb
	Je equal
	Dec Esi
	Dec Edi
	Mov Cl, Byte Ptr [Esi]
	Mov Dl, Byte Ptr [Edi]
	Sub Cl, Dl

equal:
	Mov Eax, Ecx
	Ret
_StrCmp EndP
```


## strset

```asm
_StrSet Proc MyStr:Ptr Byte, Value:Byte, n:DWord
    Mov Esi, [MyStr]    ; return original destination pointer

	; strlen
	Xor Ecx, Ecx
	@@:
	Mov Bl, Byte Ptr [Str1 + Ecx]
	Inc Ecx
	Cmp Bl, 0
	Jnz @B
	Dec Ecx

	; If n is greater than the length of string,
	; the length of string is used in place of n.
	Cmp Ecx, n
	Jge good
	Mov n, Ecx

good:
	Cld
	Xor Eax, Eax
	Mov Ecx, n
	Mov Al, Value
	Mov Edi, [MyStr]
	Rep Stosb

	Mov Eax, Esi
	Ret
_StrSet EndP
```
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

```
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

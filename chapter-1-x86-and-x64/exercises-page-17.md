# chapter 1 exercise (page 17)

_Given what you learned about CALL and RET, explain how you would read the value of EIP? Why can’t you just do MOV EAX, EIP?_

Different ways to get EIP value:
1. RIP-relative addressing: on x86-64 (as opposed to 32 bit x86), there's RIP-relative addressing (RIP is the 64-bit analogue of EIP). So in 64-bit code, you can just do `lea rax, [rip]`.
2. Shellcoders or virus writters would know this:
```
get_eip: 
    mov eax, [esp] 
    ret
call get_eip
```
3. Using `fstenv`: this fpu instructions saves the current FPU operating environment at the memory location specified with the destination operand. We also know in the FPU environment that the instruction pointer is set to the last floating point operation. Thus, in order to obtain the EIP, we must invoke a floating point function and save the environment.
```
fldz // any fpu instruction: ftst, ...
fnstenv [esp-0xC] // we subtracted 12 bytes to align eip on the top of the stack
pop ecx // ecx = eip (where the fpu instructions was executed))
```

_Why can’t you just do MOV EAX, EIP?_

- From the Intel manual:
    - The EIP register cannot be accessed directly by software; it is controlled implicitly by control-transfer instructions (such as JMP, Jcc, CALL, and RET), interrupts, and exceptions. The only way to read the EIP register is to execute a CALL instruction and then read the value of the return instruction pointer from the procedure stack. The EIP register can be loaded indirectly by modifying the value of a return instruction pointer on the procedure stack and executing a return instruction (RET or IRET).

_Come up with at least two code sequences to set EIP to 0xAABBCCDD_

1. `call 0xAABBCCDD`
2. `jmp 0xAABBCCDD`
3. `push 0xAABBCCDD then ret`

_In the example function, addme, what would happen if the stack pointer were not properly restored before executing RET?_

```asm
01: 004113A0 55 push ebp
02: 004113A1 8B EC mov ebp, esp
03: ...
04: 004113BE 0F BF 45 08 movsx eax, word ptr [ebp+8]
05: 004113C2 0F BF 4D 0C movsx ecx, word ptr [ebp+0Ch]
06: 004113C6 03 C1 add eax, ecx
07: ...
08: 004113CB 8B E5 mov esp, ebp
09: 004113CD 5D pop ebp
10: 004113CE C3 retn
```

In general, if we omit the `mov esp, ebp`, ebp, which holds the stack frame of the function that called addme(), will be set to an incorrect value and we will also end up jumping in some invalid memory address during ret. In this specific example, nothing would happen, because their was no stack operations between the prologue and epiloque. Try it yourself, write a quick C function that add two numbers, the compiler would not even include the `mov esp, ebp`.

```c
int addme (int x, int y) {
	return x + y;
}
```

_In all of the calling conventions explained, the return value is stored in a 32-bit register (EAX). What happens when the return value does not fit in a 32-bit register? Write a program to experiment and evaluate your answer. Does the mechanism change from compiler to compiler?_

_Write a program to experiment and evaluate your answer._

```c
long long MyFunc () {
	long long var = 0xffffffffffffffff;
	return var;
}

int main()
{
	long long value = MyFunc();
	printf("The returned value is %lld", value);
	return 0;
}
```

- Compile this using MS compiler would give the following disassm (x86 binary + disabling optimizations):

```asm
push ebp
mov ebp,esp
sub esp,8
mov dword ptr ss:[ebp-8],FFFFFFFF
mov dword ptr ss:[ebp-4],FFFFFFFF
mov eax,dword ptr ss:[ebp-8]
mov edx,dword ptr ss:[ebp-4]
mov esp,ebp
pop ebp
ret 
```

- The 64 bits var assignement to 0xffffffffffffffff was translated to two 32-bits stack operations ([ebp-4] and [ebp-8], and place acccordingly into eax:edx that are returned back to the caller.

_Does the mechanism change from compiler to compiler?_

- gcc disassm:

```asm
push ebp
mov ebp,esp
sub esp,10
mov dword ptr ss:[ebp-8],FFFFFFFF
mov dword ptr ss:[ebp-4],FFFFFFFF
mov eax,dword ptr ss:[ebp-8]
mov edx,dword ptr ss:[ebp-4]
leave 
ret 
```
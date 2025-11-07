---
title: HackTheBox Encryption Bot writeup
date: 2024-09-16 08:00:00 +/-0000
categories:
  - Writeups
tags:
  - writeup
  - reverse-engineering
  - ctf
---
## Challenge Description

*My friend send me a encrypted message by using encryption bot. This is a important message. Can you decrypt the message for me?*

There is encrypted flag and elf file provided

![](/assets/img/all/Pasted image 20251107142343.png)

## Solution

Running the program to see what it does

![](/assets/img/all/Pasted image 20251107142358.png)

Lets open up `Ghidra` and try to reverse engineer this. First step is to identify the `main` function. Checking the `entry` function that runs at the start we can find what function is loaded before calling `libc_start_main`. Thats our main function

![](/assets/img/all/Pasted image 20251107142433.png)

![](/assets/img/all/Pasted image 20251107142437.png)

We can see the *Enter the text to encrypt* string which means its in fact the main function that runs at the start. After receiving input from the user, program checks if file `data.dat` exists and deletes it if it does. Then 4 functions are called. Function `FUN_0010128a` does nothing, so let's analyze the other 3.

![](/assets/img/all/Pasted image 20251107142528.png)

The first function checks the length of the input whether its `0x1b` which is 27 in decimal

![](/assets/img/all/Pasted image 20251107142545.png)

Yep, it works for length 27. Let's see the other function that takes the user input as argument

![](/assets/img/all/Pasted image 20251107142611.png)

In this function there is a loop that iterates through each character of the user input and passes it to the `FUN_001011d9` function

![](/assets/img/all/Pasted image 20251107142627.png)

Function `FUN_001011d9` converts the ASCII character to an 8 bit binary representation and then writes it to `data.dat` file in the reverse order. Having followed this flow, we can go back to `FUN_001014ba` which is the last function called in `main`

![](/assets/img/all/Pasted image 20251107142844.png)

We see that the `data.dat` file is opened, a `FUN_00101291` is called which does nothing and then there is a for loop that iterates as long as the `local_c` is less than `0xd9`, which is 217 in decimal. So there is 216 iterations. Why 216? Because our input has length of 27 and each character got converted to an 8 bit binary which means that there is 27 * 8=216 bits. Inside the for loop the first two if statements check the value of character read from file. If the value is '0' then 0 is added to an array, if it's '1' then 1 is added

![](/assets/img/all/Pasted image 20251107142955.png)

This next bit of code executes every 6 bits and it contains a for loop that iterates 6 times and does 2 things:
 - stores the result of `FUN_001013ab` function in `local_28`
 - keeps track of the sum of bit at index `i` (which goes from highest to lowest) multiplied by `local_28`
 Let's see what does the `FUN_001013ab` return
 
![](/assets/img/all/Pasted image 20251107143247.png)

 There is a for loop that iterates `param_1` times and it multiplies `local_c` by 2 - which starting value is 1. `FUN_0010129f` does nothing. Essentially the returned value is 2^param
Let's go back to the caller function 

![](/assets/img/all/Pasted image 20251107143911.png)

 As previosly mentioned `local_14` stores the sum of a bit at index `i` multiplied by 
`local_28` (2^j). So it essentially converts a 6 bit binary to decimal. After the loop there is the last call to another function called `FUN_001013e9` with `local_14` as an argument

![](/assets/img/all/Pasted image 20251107144030.png)

Here we see a bunch of variables storing hex values. After converting them to ASCII we get a `YXWVUTSR6543210ZEDCBA987MLKJIHGFdcbaQPONlkjihgfetsrqponmzyxwvu`. However since this is a x86 program it uses little endian system to store bytes in memory, which essentially means that each value is stored in reversed order. So in memory it would look like this: `RSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQabcdefghijklmnopqrstuvwxyz`. Then there is a for loop that just puts 0s inside the `local_158`, which is a 42 long array. Now the important part happens in `putchar` function. It takes the address of the first byte of `local_198` with added value of `param_1`, which acts as an offset to the address. Then it takes 1 byte from the calculated address (char \*), converts it to int and the `putchar` function converts it back to char.

Quick summary of the program:
- takes the string with length of 27 
- converts each character to binary
- every 6 bits are converted to decimal
- character is returned based on the key string and decimal which acts as an offset

Now let's write some code that will decrypt the flag

```python
key = "RSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQabcdefghijklmnopqrstuuvwxyz"

with open('flag.enc','r') as f:
    flag = f.read()

bits = ""
for i in flag:
    decimal = key.index(str(i)
    binary = f'{decimal:06b}'
    bits += binary

binaries = [bits[i:i+8] for i in range(0,len(bits),8)]
characters = [chr(int(x,2) for x in binaries]

decoded = ''.join(characters)
print(decoded)
```

![](/assets/img/all/Pasted image 20251107144201.png)

HTB{3nCrypT10N_W1tH_B1Ts !! }

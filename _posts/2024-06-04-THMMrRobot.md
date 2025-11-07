---
title: TryHackMe Mr Robot CTF writeup
date: 2024-06-04 08:00:00 +/-0000
categories:
  - Writeups
tags:
  - ctf
  - writeup
  - offensive
---

![](/assets/img/all/Pasted image 20251106194406.png)

## Enumeration

First, I ran a `nmap` scan

```shell
sudo nmap -p- -sV -A -Pn -T 10.10.76.213
```

![](/assets/img/all/Pasted image 20240314081745.png)

I checked what's running on port 80

![](/assets/img/all/Pasted image 20251106195000.png)

While trying out these commands, I continued enumerating the site with `gobuster`

```shell
gobuster dir -u http://10.10.76.213/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![](/assets/img/all/Pasted image 20251106195250.png)

Seeing wp* directories means the site is build with wordpress. However I started with the easiest step - visiting the /robots

![](/assets/img/all/Pasted image 20251106195559.png)

### Key 1

073403c8a58a1f80d943455fb30724b9

I also downloaded the `fsocity.dic`

![](/assets/img/all/Pasted image 20251106195711.png)

It's a long wordlist but there is a lot of repeated words. I sorted it and kept only unique values

```shell
sort -u fsocity.dic > sorted.txt
```

This might be useful in a bruteforce attack. I turned to /login page found with `gobuster`. After trying simple credentials like admin:admin I got an error 'Invalid username', meaning the server exposes valid usernames

![](/assets/img/all/Pasted image 20251106200052.png)

I used `wpscan` to enumerate the users but no luck. I decided to bruteforce it. I used `Burp Suite`, captured my request with proxy and sent it to intruder

![](/assets/img/all/Pasted image 20251106200246.png)

As a wordlist I used the `sorted.txt` file I created earlier and found username `Elliot`

![](/assets/img/all/Pasted image 20251106200500.png)

Next? I bruteforced the password. This time `wpscan` found valid credentials

![](/assets/img/all/Pasted image 20251106200536.png)

## Exploitation

With access to the admin dashboard, I pasted php reverse shell to 404 template theme

![](/assets/img/all/Pasted image 20251106200904.png)
![](/assets/img/all/Pasted image 20251106201009.png)

A second key was in `/home/robot` however I did not have permission to read it

## Privilege Escalation

I searched for programs with SUID set

```shell
find / -perm -u=s -type f 2>/dev/null
```

![](/assets/img/all/Pasted image 20251106201227.png)

Nmap caught my eye. After searching online I found this little article on how to use nmap to escalate privileges https://www.adamcouch.co.uk/linux-privilege-escalation-setuid-nmap/

![](/assets/img/all/Pasted image 20251106201302.png)

With root privileges I read the second (`/home/robot`) and third (`/root`) key

### Key 2 and 3

822c73956184f694993bede3eb39f959

04787ddef27c3dee1ee161b21670b4e4
---
title: KC7Cyber System Shutdown at Azure Crest writeup
date: 2025-08-14 08:00:00 +/-0000
categories:
  - Writeups
tags:
  - writeup
  - log-analysis
---
##### **Question 1**

It's your first day on the security team at Azure Crest, a few minutes left before your lunch break (and boy are you starving), when you receive an email from your manager who is OOO.  

![](/assets/img/all/Pasted image 20251107191052.png)

Well, lunch will have to wait a bit longer. Let's get into it, the faster you resolve this, the sooner you'll be able to tame your rumbling stomach.

**What is the name of that file?**

```text
SecurityAlerts
| where description has "quarantined"
```

![](/assets/img/all/Pasted image 20251107191106.png)

*New_Healthcare_Protocols.docm*

##### **Question 2**

We have information that this threat actor is using two different filenames to distribute their malware.

**What other filename is this threat actor using?**

First, let's see if the filename we just found was mentioned in any emails

```text
Email
| where link has "New_Healthcare_Protocols.docm"
```

![](/assets/img/all/Pasted image 20251107191120.png)

We got 25 hits. All of the emails were sent either from `medstaffinfo@hospitalcomm.org` or `healthupdate@gmail.com`. Let's see what other emails were sent from these addresses, specifically let's focus on distinct links contained in those emails

```text
Email
| where sender in ("medstaffinfo@hospitalcomm.org","healthupdate@gmail.com")
| distinct link
```

![](/assets/img/all/Pasted image 20251107191130.png)

*Pediatric_Care_Update.docm*

##### **Question 3**

**How many of Azure Crest's employees clicked the email link?**

Let's use the links we found and search for network connections made to them from distinct source IPs

```text
let links = Email
| where sender in ("medstaffinfo@hospitalcomm.org","healthupdate@gmail.com")
| distinct link;
OutboundNetworkEvents
| where url in (links)
| distinct src_ip
```

*37*

##### **Question 4**

**Which unusual directory path is being used by the attacker to store legitimate remote access executables?**

Firstly let's find the machines that have the file `New_Healthcare_Protocols.docm` and investigate one of them

```text
FileCreationEvents
| where filename == "New_Healthcare_Protocols.docm"
```

![](/assets/img/all/Pasted image 20251107191149.png)

Let's see what other files are created after the malicious one

```text
FileCreationEvents
| where hostname == "P3EX-DESKTOP" and timestamp >= datetime(2024-03-01T11:58:33.000Z)
```

![](/assets/img/all/Pasted image 20251107191157.png)

*C:\ProgramData\Heartburn*

##### **Question 5**

**What protocol did the threat actor use to establish a connection with its remote server?**

We saw that `putty.exe` got created. Let's look for it in process events

```text
ProcessEvents
| where hostname == "P3EX-DESKTOP" and process_commandline has "putty.exe"
```

![](/assets/img/all/Pasted image 20251107191206.png)

*ssh*

##### **Question 6**

**What command was executed to extract the files into that directory?**

```text
ProcessEvents
| where hostname == "P3EX-DESKTOP" and process_commandline has "C:\\ProgramData\\Heartburn"
```

![](/assets/img/all/Pasted image 20251107191223.png)

*cmd.exe /c Expand-Archive -Path C: \ProgramData\heartburn.zip -DestinationPath C:\ProgramData\Heartburn*

##### **Question 7**

**Which IP address associated with this threat actor is located in Croatia?**

It's the IP used in the `putty` command

*93.142.203.80*

##### **Question 8**

**What user-agent is associated with the threat actor?**
(Just enter the last part of the string xx.xx)

Let's see if there is any incoming network traffic from the IP we found previously

```text
InboundNetworkEvents
| where src_ip == "93.142.203.80"
```

![](/assets/img/all/Pasted image 20251107191233.png)

*12.00*

##### **Question 9**

**What kind of system did Azure Crest think was a good idea to run on their own?**

```text
InboundNetworkEvents
| where user_agent == "Opera/9.63.(Windows CE; lb-LU) Presto/2.9.161 Version/12.00"
```

![](/assets/img/all/Pasted image 20251107191244.png)

*erp*

##### **Question 10**

**What file extension is used by the threat actor to encrypt files?**

Since there was no files with suspicious extension on the host we investigated in previous questions, let's use different approach and see the number of occurrences of different file extensions across all hosts

```text
FileCreationEvents
| extend extension = tostring(parse_path(path).Extension)
| summarize Count=count() by extension
```

![](/assets/img/all/Pasted image 20251107191254.png)

*scholopendra*

##### **Question 11**

**Which file, when executed, triggers the encryption of database files?**

Let's see on which host the ransomware ran

```text
FileCreationEvents
| where filename has "scholopendra"
| distinct hostname
```

![](/assets/img/all/Pasted image 20251107191304.png)

Viewing process events on host `SUPER-DB-SERVER-9000` around the time of file creation events with extension `scholopendra` we find the script

```text
ProcessEvents
| where hostname == "SUPER-DB-SERVER-9000"
```

![](/assets/img/all/Pasted image 20251107191315.png)

*UrTottalyPwned.bat*

##### **Question 12**

**When was the batch file downloaded? (paste the full timestamp)**

```text
FileCreationEvents
| where hostname == "SUPER-DB-SERVER-9000" and filename == "UrTottalyPwned.bat"
```

![](/assets/img/all/Pasted image 20251107191326.png)

*2024-04-02T11:29:37.000Z*

##### **Question 13**

**What password was used to encrypt the archive containing the financial database files?**

```text
ProcessEvents
| where hostname == "SUPER-DB-SERVER-9000"
```

![](/assets/img/all/Pasted image 20251107191335.png)

*finnaberich*

##### **Question 14**

**What is the hostname of the machine compromised by the attackers to achieve their objectives?**

*SUPER-DB-SERVER-9000*

##### **Question 15**

**What command was run to let Azure Crest know that something went wrong?**

It's the command that changes the wallpaper

![](/assets/img/all/Pasted image 20251107191345.png)

*cmd.exe /c reg add 'HKCU\Control Panel\Desktop' /v Wallpaper /t REG_SZ /d 'C:\Users\Public\ItWentWrong.jpg' /f && reg add 'HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System' /v Wallpaper /t REG_SZ /d 'C:\Users\Public\ItWentWrong.jpg' /f*

##### **Question 16**

**What domain was used to target the employee that held the keys to the database?**

```text
let email = Employees
| where hostname == "SUPER-DB-SERVER-9000"
| distinct email_addr;
Email
| where recipient in (email)
| distinct link
```

![](/assets/img/all/Pasted image 20251107191355.png)

*unhealthyrecordsystems.tech*

##### **Question 17**

To answer this question check the *Training Guide* for this challenge

*healthrecordsystems.tech*


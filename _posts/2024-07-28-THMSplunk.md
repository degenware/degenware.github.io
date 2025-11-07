---
title: Investigating with Splunk - TryHackMe writeup
date: 2024-07-28 08:00:00 +/-0000
categories:
  - Writeups
tags:
  - writeup
  - log-analysis
---
## Story

*SOC Analyst **Johny** has observed some anomalous behaviours in the logs of a few windows machines. It looks like the adversary has access to some of these machines and successfully created some backdoor. His manager has asked him to pull those logs from suspected hosts and ingest them into Splunk for quick investigation. Our task as SOC Analyst is to examine the logs and identify the anomalies.*

## Questions

1. How many events were collected and Ingested in the index **main**?

![](/assets/img/all/Pasted image 20251107134509.png)

12256

2. On one of the infected hosts, the adversary was successful in creating a backdoor user. What is the new username?

Examining the logs we can find field called `EventID`

![](/assets/img/all/Pasted image 20251107134538.png)

Let's search what is the event id for creating a new user in Windows

![](/assets/img/all/Pasted image 20251107134604.png)

We add the event id to our query

![](/assets/img/all/Pasted image 20251107134615.png)
![](/assets/img/all/Pasted image 20251107134620.png)

A1berto

3. On the same host, a registry key was also updated regarding the new backdoor user. What is the full path of that registry key?

![](/assets/img/all/Pasted image 20251107135159.png)

After quick examination of the logs we find the path of the registry key

![](/assets/img/all/Pasted image 20251107135210.png)

HKLM\SAM\SAM\Domains\Account\Users\Names\A1berto

4. Examine the logs and identify the user that the adversary was trying to impersonate.

Selecting the `User` field we instantly see that new user 'A1berto' is impersonating 'Alberto'

![](/assets/img/all/Pasted image 20251107135238.png)

Alberto

5. What is the command used to add a backdoor user from a remote computer?

Going back to our logs regarding 'A1berto' user we can see `CommandLine` field. There we find the command that is using the `wmic.exe` which is used in administrating tasks on remote computers

![](/assets/img/all/Pasted image 20251107135317.png)

C:\windows\System32\Wbem\WMIC.exe" /node:WORKSTATION6 process call create "net user /add A1berto paw0rd1

6. How many times was the login attempt from the backdoor user observed during the investigation?

Lets check the event id for successful and failed login

![](/assets/img/all/Pasted image 20251107135407.png)
![](/assets/img/all/Pasted image 20251107135410.png)

Now lets search for "A1berto" with these event ids

![](/assets/img/all/Pasted image 20251107135429.png)

0

7. What is the name of the infected host on which suspicious Powershell commands were executed?

To find the powershell command we go back to "A1berto" logs. There is a `Category` field

![](/assets/img/all/Pasted image 20251107135528.png)

Executing a powershell would be `Executing Pipeline` category so lets search for this event

![](/assets/img/all/Pasted image 20251107135544.png)

![](/assets/img/all/Pasted image 20251107135548.png)

James.browne

8. PowerShell logging is enabled on this device. How many events were logged for the malicious PowerShell execution?

Lets check the event id for executing powershell commands. I found the ids and their meanings on this blog: https://blog.reconinfosec.com/endpoint-logging-for-the-win

![](/assets/img/all/Pasted image 20251107135625.png)

79

9. An encoded Powershell script from the infected host initiated a web request. What is the full URL?

Since the script executed on the infected host lets add it to our search

![](/assets/img/all/Pasted image 20251107135656.png)

We quickly find the base64 encrypted script

![](/assets/img/all/Pasted image 20251107135709.png)

After decrypting

![](/assets/img/all/Pasted image 20251107135746.png)

The url is base64 encoded so we decode it once again, defang it and add the path news.php

hxxp[://]10[.]10[.]10[.]5/news[.]php
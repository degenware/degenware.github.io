---
title: KC7Cyber Scholomance writeup
date: 2026-03-02 08:00:00 +/-0000
categories:
  - Writeups
tags:
  - writeup
  - log-analysis
---
##### Question 1

Oh no! You've been hit by something, someone, or some force that has stolen something from your enclave. You're not sure, but you need to investigate. There was a lot of emails going around about regulation and compliance. Search for these terms in the Emails table.

**When is the first time a sender with either of these terms show up?**(Note: Let's ignore emails from our trusted partner companies for now)

```text
Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
```

![](/assets/img/all/Pasted image 20260427154119.png)

*2023-10-10T09:01:05.000Z*

##### Question 2

Investigate these email addresses.

**How many sender email addresses are linked together by the same theme?**

```text
Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
| distinct sender
| count
```

*4*

##### Question 3

**How many unique subject lines were sent by these email addresses?**

```text
Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
| distinct subject
| count
```

*6*

##### Question 4

**Which of these subject lines was used to target the fewest number of users?**

```text
Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
| summarize Count=count() by subject
| sort by Count
```

![](/assets/img/all/Pasted image 20260427163557.png)

*\[EXTERNAL] IMPORTANT: Provide your information now.*

##### Question 5

Let's take all of the email addresses and look for how many distinct URLs were sent by them.

**How many unique URLs were sent?**

```text
Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
| distinct link
| count
```

*8*

##### Question 6

**How many employees clicked on these links?**

```text
let sus_links = Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
| distinct link;
OutboundNetworkEvents
| where url in (sus_links)
| distinct src_ip
| count
```

*20*

##### Question 7

**How many distinct employee roles clicked these links?**

```text
let sus_links = Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
| distinct link;
let ips = OutboundNetworkEvents
| where url in (sus_links)
| distinct src_ip;
Employees
| where ip_addr in (ips)
| distinct role
| count
```

*8*

##### Question 8

**How many Dark Magic Regulation Experts clicked the links?**

```text
let sus_links = Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
| distinct link;
let ips = OutboundNetworkEvents
| where url in (sus_links)
| distinct src_ip;
Employees
| where ip_addr in (ips) and role == "Dark Magic Regulation Expert"
```

![](/assets/img/all/Pasted image 20260427164559.png)

*4*

##### Question 9

**What time did the first employee click on one of the links?**

```text
let sus_links = Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
| distinct link;
OutboundNetworkEvents
| where url in (sus_links)
```

![](/assets/img/all/Pasted image 20260427170241.png)

*2023-10-16T07:48:29.000Z*

##### Question 10

**What is that employee's username?(The one who first clicked on the link)**

```text
let sus_links = Email
| where sender !has "researchcompliance.biz"
| where sender contains "regulation" or sender contains "compliance"
| distinct link;
let ip = OutboundNetworkEvents
| where url in (sus_links)
| take 1
| project src_ip;
Employees
| where ip_addr in (ip)
```

![](/assets/img/all/Pasted image 20260427171203.png)

*sugupta*

##### Question 11

Let's look at this employee's Authentication activity.

**What is the most common user agent used by this employee? (Format Firefox/XX.X)**

```text
AuthenticationEvents
| where username == "sugupta"
| summarize Count=count() by user_agent
```

![](/assets/img/all/Pasted image 20260427171356.png)

*Firefox/47.0*

##### Question 12

**When was there a login attempt from a different user agent for this employee? Copy and paste the exact timestamp**

```text
AuthenticationEvents
| where username == "sugupta" and user_agent has "11.0"
```

![](/assets/img/all/Pasted image 20260427171545.png)

*2023-10-17T01:17:35.000Z*

##### Question 13

**From what IP address did this authentication occur?**

*173.188.151.49*

##### Question 14

**How many login attempts were seen from this IP?**

```text
AuthenticationEvents
| where src_ip == "173.188.151.49"
```

![](/assets/img/all/Pasted image 20260427171732.png)

*3*

##### Question 15

**Which user account was successfully logged in to from this IP? Enter the username**

![](/assets/img/all/Pasted image 20260427171918.png)

*almcwaters*

##### Question 16

**Let's look back at all AuthenticationEvents. Which user agent was used the most? (Format Firefox/XX.X)**

```text
AuthenticationEvents
| summarize Count=count() by user_agent
| sort by Count
```

![](/assets/img/all/Pasted image 20260427172035.png)

*Firefox/69.0*

##### Question 17

**How many user accounts had login attempts from this user agent?**

```text
AuthenticationEvents
| where user_agent == "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:69.0) Gecko/20100101 Firefox/69.0"
| distinct username
| count
```

*405*

##### Question 18

**How many IP addresses were used with this user agent?**

```text
AuthenticationEvents
| where user_agent == "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:69.0) Gecko/20100101 Firefox/69.0"
| distinct src_ip
| count
```

*2635*

##### Question 19

**How many user account were successfully logged into using this user agent?**

```text
AuthenticationEvents
| where user_agent == "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:69.0) Gecko/20100101 Firefox/69.0"
| where result contains "success"
| count
```

*28*

##### Question 20

**This type of attack is known as a __ .**

*password spray*

##### Question 21

Let's look for follow on activity, including phishing emails targeting higher privileged and sensitive users.

**What is the email subject that the threat actor used?**

```text
let com_users = AuthenticationEvents
| where user_agent == "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:69.0) Gecko/20100101 Firefox/69.0"
| where result contains "success"
| distinct username;
let usr_email = Employees
| where username in (com_users)
| project email_addr;
Email
// activity after the first compromise and emails sent within the organization
| where sender in (usr_email) and timestamp > datetime(2023-10-10T06:14:13.000Z) and recipient has "deathcon-scholomance.io"
| summarize count() by subject
| sort by count_
```

![](/assets/img/all/Pasted image 20260427173850.png)

*URGENT: Please provide locations where your research is located.*

##### Question 22

The threat actor may have done some additional recon within the environment.

**Checking the internal network events, what document did they look within the enclave's secretive shares?**

Finding suspicious user agent, unusually high number of requests indicates a scan

```text
InboundNetworkEvents
| summarize count() by user_agent
| sort by count_
```

![](/assets/img/all/Pasted image 20260427180851.png)

```text
InboundNetworkEvents
| where user_agent == "Mozilla/5.0 (Linux; Android 2.0) AppleWebKit/531.2 (KHTML, like Gecko) Chrome/62.0.890.0 Safari/531.2"
| project url
```

![](/assets/img/all/Pasted image 20260427180944.png)

*important_research.docx*

##### Question 23

The threat actor may have forgot how to do some reconaissance in an internal environment.

**What website did they use to get a refresher? Copy and paste the full URL.**

```text
let com_users = AuthenticationEvents
| where user_agent == "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:69.0) Gecko/20100101 Firefox/69.0"
| where result contains "success"
| distinct username;
let phish_users = Email
| where subject == "URGENT: Please provide locations where your research is located."
| distinct recipient;
let usr_ip = Employees
| where username in (com_users) or email_addr in (phish_users)
| distinct ip_addr;
OutboundNetworkEvents
| where src_ip in (usr_ip) and method == "GET" and url contains "recon" and timestamp > datetime(2023-10-10T06:14:13.000Z)
```

![](/assets/img/all/Pasted image 20260427183553.png)

*https:]//svch0st.medium.com/active-directory-recon-cheat-sheet-76ccc16dc6e8*

##### Question 24

Continue to investigate the threat actor. They may have downloaded and used an archiving tool.

**What url did they use to download it?**

```text
let com_users = AuthenticationEvents
| where user_agent == "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:69.0) Gecko/20100101 Firefox/69.0"
| where result contains "success"
| distinct username;
let phish_users = Email
| where subject == "URGENT: Please provide locations where your research is located."
| distinct recipient;
let usr_ip = Employees
| where username in (com_users) or email_addr in (phish_users)
| distinct ip_addr;
OutboundNetworkEvents
| where src_ip in (usr_ip) and timestamp > datetime(2023-10-10T06:14:13.000Z) and url has ".exe"
| distinct url
```

![](/assets/img/all/Pasted image 20260427200954.png)

*https:]//www.7-zip.org/a/7z2002-x64.exe*

##### Question 25

It looks like they staged some files for extraction.

**What is the timestamp of the last file they archived and created?**

Searching for suspicious files

```text
let com_users = AuthenticationEvents
| where user_agent == "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:69.0) Gecko/20100101 Firefox/69.0"
| where result contains "success"
| distinct username;
let phish_users = Email
| where subject == "URGENT: Please provide locations where your research is located."
| distinct recipient;
let usr_host = Employees
| where username in (com_users) or email_addr in (phish_users)
| distinct hostname;
FileCreationEvents
| where hostname in (usr_host) and filename has "7z" and timestamp > datetime(2023-11-03T09:58:22.000Z)
| distinct filename
```

![](/assets/img/all/Pasted image 20260427220544.png)

Searching for all events with these files

```text
FileCreationEvents
| where filename in ("research.7z","documents.7z")
```

*2023-11-03T10:24:43.000Z*

##### Question 26

Let's take a look at our security alerts. It looks like there was a file that was quarantined on October 2, 2023.

**How many files with this filename is present on Scholomance hosts?**

```text
SecurityAlerts
| where description has "quarantined"
```

![](/assets/img/all/Pasted image 20260427221202.png)

```text
FileCreationEvents
| where filename == "details.zip"
| distinct hostname
```

*15*

##### Question 27

**How many roles were impacted with this file?**

```text
let hosts = FileCreationEvents
| where filename == "details.zip"
| distinct hostname;
Employees
| where hostname in (hosts)
| distinct role
```

*9*

##### Question 28

Let's pivot to emails. Identify all of the unique senders for this file.

**How many unique domains did they use to serve this file?**

```text
Email
| where link has "details.zip"
| extend Host = tostring(parse_url(link).Host)
| distinct Host
```

*5*

##### Question 29

Look for all the files this threat actor served.

**How many total emails were sent by this threat actor containing potentially malicious files?**

```text
let senders = Email
| where link has "details.zip"
| distinct sender;
Email
| where sender in (senders)
```

*69*

##### Question 30

**How many of these files were present on hosts?**

```text
let senders = Email
| where link has "details.zip"
| distinct sender;
let files = Email
| where sender in (senders)
| extend Path = tostring(parse_url(link).Path)
| extend File = tostring(parse_path(Path).Filename)
| distinct File;
FileCreationEvents
| where filename in (files)
```

*66*

##### Question 31

Let's look for impact.

**How many total job roles were targeted?**

```text
let senders = Email
| where link has "details.zip"
| distinct sender;
let emails = Email
| where sender in (senders)
| distinct recipient;
Employees
| where email_addr in (emails)
| distinct role
```

*12*

##### Question 32

Let's start investigating what happens after these emails were sent & downloaded onto these hosts. The threat actor is staging files somewhere.

**What is the full path to the directory?**

Focusing on one of the infected hosts

```text
FileCreationEvents
| where hostname == "2ZJA-LAPTOP" and timestamp > datetime(2023-10-02T11:07:30.000Z)
```

![](/assets/img/all/Pasted image 20260427231022.png)

*C:\ProgramData*

##### Question 33

It may be worth it to start conducting open source intelligence on these files. Some of these files are tied to the previous section.

**What other group, named by Mandiant, are these other files associated with?**

*APT28*

##### Question 34

**Now that you have some information on this threat, what filename (minus the extension) is used for collecting credentials?**

```text
ProcessEvents
| where hostname == "2ZJA-LAPTOP" and timestamp > datetime(2023-10-02T11:07:30.000Z)
```

![](/assets/img/all/Pasted image 20260427233753.png)

*dc529177-39b4-4828-8c66-79fe35145d07*

##### Question 35

It looks like they may have dumped something from a native Windows database and saved it somewhere.

**What's the name of this file?**

![](/assets/img/all/Pasted image 20260427233847.png)

*software.save*

##### Question 36

**What's being used by the threat actor as a bootloader?**

![](/assets/img/all/Pasted image 20260427234942.png)

*msedge*

##### Question 37

**What GUID is being used in conjunction with this?**

*aaccd6de-ce95-4fb2-b2c1-2d7ca08661a3*

##### Question 38

Investigate likely data exfiltration.

**What filename were they using to stage the data?**

![](/assets/img/all/Pasted image 20260427235828.png)

*protect.zip*

##### Question 39

**What's the destination where the data was likely being transferred to?**

![](/assets/img/all/Pasted image 20260428000038.png)

*https:]//webhook.site/dc529177-39b4-4828-8c66-79fe35145d07*

##### Question 40

There were some things missing or re-configured on these hosts.

**What did the threat actor do for defense evasion? What did they disable?**

![](/assets/img/all/Pasted image 20260428000943.png)

*lsa protection*

##### Question 41

**Both of these threat actor groups are part of which specific agency from their country of origin?**

*gru*

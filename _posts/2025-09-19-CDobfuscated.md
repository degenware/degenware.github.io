---
title: CyberDefenders Obfuscated Lab writeup
date: 2025-09-19 08:00:00 +/-0000
categories:
  - Writeups
tags:
  - writeup
  - malware-analysis
---
## Scenario

*While working as a SOC analyst, you may encounter alerts from the enterprise Endpoint Detection and Response (EDR) system regarding unusual activity on an end-user machine. In one instance, a user reported receiving an email containing a DOC file from an unknown sender. The user subsequently submitted the document for analysis to ensure it does not pose a security risk.*

-------

We are provided with a ZIP file containing the suspicious DOC file

![](/assets/img/all/Pasted image 20251107192039.png)

## Questions

##### **Question 1**

*What is the SHA256 hash of the **DOC** file?*

```bash
$ sha256sum 49b367ac261a722a7c2bbbc328c32545
```

ff2c8cadaa0fd8da6138cce6fce37e001f53a5d9ceccd67945b15ae273f4d751

##### **Question 2**

*Multiple streams contain macros in this document. Provide the number of the lowest one.*

Let's use `oledump` to analyze the doc file

![](/assets/img/all/Pasted image 20251107192107.png)

We can see that streams 8 and 9 are marked with the 'm' letter which means they contain macros

8

##### **Question 3**

*What is the decryption key of the obfuscated code?*

Let's retrieve the macro using `oledump`

```bash
$ oledump.py -s 8 -v 49b367ac261a722a7c2bbbc328c32545
```

Macro is obfuscated however we can see that a file `maintools.js` is created in %appdata%/Microsoft/Windows folder and then it's run with `EzZETcSXyKAdF_e5I2i1` string as an argument, which would suggest it's the decryption key

![](/assets/img/all/Pasted image 20251107192117.png)

EzZETcSXyKAdF_e5I2i1

##### **Question 4**

*What is the name of the dropped file?*

maintools.js

##### **Question 5**

*This script uses what language?*

JScript

##### **Question 6**

*What is the name of the variable that is assigned the command-line arguments?*

We need to get hold of `maintools.js` file. To do that, we can either run the DOC file in an isolated enviroment or use an online sandbox, for example [Triage](https://tria.ge/) and download the dropped file from there

![](/assets/img/all/Pasted image 20251107192130.png)

![](/assets/img/all/Pasted image 20251107192134.png)

wvy1

##### **Question 7**

*How many command-line arguments does this script expect?*

1

##### **Question 8**

*What instruction is executed if this script encounters an error?*

![](/assets/img/all/Pasted image 20251107192144.png)

WScript.Quit()

##### **Question 9**

*What function returns the next stage of code (i.e. the first round of obfuscated code)?*

From the screenshot above we can see that the first function used is the *y3zb* function, which returns big, obfuscated string 

![](/assets/img/all/Pasted image 20251107192152.png)

y3zb

##### **Question 10**

*The function **LXv5** is important, what variable is assigned a key string value in determining what this function does?*

![](/assets/img/all/Pasted image 20251107192159.png)

Variable *LUK7* contains the Base64 alphabet (although it's never referenced), which would suggest the *LXv5* function works as a Base64 decoder. We can confirm this by further analysing this function as well as the *MTvK* function which returns Base64 index of a character (instead of using the *LUK7* variable)

![](/assets/img/all/Pasted image 20251107192208.png)

LUK7

##### **Question 11**

*What encoding scheme is this function responsible for decoding?*

Base64

##### **Question 12**

*In the function **CpPT**, the first two **for()** loops are responsible for what important part of this function?*

The *CpPT* function is responsible for decrypting the previously decoded base64 strings. To do that it uses the key mentioned in Q3. After searching online we can indentify that the encryption algorithm used in the *CpPT* function is RC4 and the *for()* loops are used for key-scheduling algorithm.

![](/assets/img/all/Pasted image 20251107192224.png)
_code from [wikipedia](https://en.wikipedia.org/wiki/RC4) illustrating the key-scheduling algorithm_

![](/assets/img/all/Pasted image 20251107192238.png)
_*for()* loops in the *CcPT* function_

key-scheduling algorithm

##### **Question 13**

*The function CpPT requires two arguments, where does the value of the first argument come from?*

command-line argument

##### **Question 14**

*For the function **CpPT**, what does the first argument represent?*

key

##### **Question 15**

*What encryption algorithm does the function **CpPT** implement in this script?*

RC4

##### **Question 16**

*What function is responsible for executing the deobfuscated code?*

![](/assets/img/all/Pasted image 20251107192311.png)

eval

##### **Question 17**

*What Windows Script Host program can be used to execute this script in command-line mode?*

cscript.exe

##### **Question 18**

*What is the name of the first function defined in the deobfuscated code?*

![](/assets/img/all/Pasted image 20251107192320.png)

UspD

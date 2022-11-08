---
layout: pages
title: My bugbounty recon workflow
date: 2022-11-07 18:30:00 +0000
categories:
- workflow
- recon
- Bugbounty
comments: true

---
This blog post contains my recon workflow in step by step process. My recon process is more of information gathering. I keep changing this blog post based on my recon workflow.

<!--more-->

### Step 1:

Find the subdomains for wildcard domains.

#### Using amass:

    amass enum -brute -active -d domain.com | anew domains

#### Using assetfinder:

    cat wildcards | assetfinder --subs-only | anew domains

#### Using findomain:

    findomain -f wildcards | anew domains

#### Using dnsgen:

    cat domains | dnsgen - | anew domains

### Step 2:

Find active domains

#### Using httprobe:

    cat domains | httprobe -c 50 | anew hosts

### Step 3:

Gathering site information

#### Using fff:

    cat hosts | fff -d 1 -S -o roots

The above store all header and body and store it in roots

#### Using gowitness:

    gowitness file -f hosts --threads 3 ---fullpage single -P <dir>

***

The above steps are normal for wildcards domains. Now we recon for each individual site that are in scope.

### Step 4:

Check wayback urls to find something interesting

    waybackurls <domain_name> | anew <domain_name_wayback>

### Step 5:

Scan for the parameter

#### Using arjun:

    arjun -i hosts -oT arjun_param.txt

#### Using Paramspider:

    python3 paramspider.py --domain <target_domain> --level high --output domainname_params.txt

Once we have the output form paramspider, we can further analyse the pattern to find XSS.

    gf xss.json params.txt | tee domain_xss.txt

[https://github.com/1ndianl33t/Gf-Patterns/blob/master/xss.json](https://github.com/1ndianl33t/Gf-Patterns/blob/master/xss.json "https://github.com/1ndianl33t/Gf-Patterns/blob/master/xss.json")

The output from gf then parsed to tool called Dalfox([https://github.com/hahwul/dalfox](https://github.com/1ndianl33t/Gf-Patterns/blob/master/xss.json "https://github.com/1ndianl33t/Gf-Patterns/blob/master/xss.json") "Dalfox"))

    cat xss.txt | dalfox pipe

### Step 6:

Enumerate directories

#### Using ffuf:

    ffuf -ac -v -u https://domain/FUZZ -w wordlist.txt

### Step 7:

Monitor for any change
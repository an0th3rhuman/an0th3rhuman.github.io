---
layout: pages
title: My recon workflow for Bugbounty hunting
date: 2022-11-07 18:30:00 +0000
categories:
- workflow
- recon
- bugbounty
comments: true
published: false

---
This blog post contains my recon workflow in step by step process.

<!--more-->

### Step 1:

Find the subdomains for wildcard domains.

#### Using amass:

```bash
amass enum -brute -active -d domain.com | anew domains
```

#### Using assetfinder:

```bash
cat wildcards | assetfinder --subs-only | anew domains
```

#### Using findomain:

```bash
findomain -f wildcards | anew domains
```

### Step 2:

Find active domains
---
layout: post
title:  "SSRF - Checklist"
date:   2020-04-06 08:07:19
categories: [web,checklist]
comments: true

---

This contains the basic checklist for SSRF which can be handy during penetration testing

Let's get started!

<!--more-->

### How to find SSRF?
 
- [ ] Do the application fetch data from external URL?

- [ ] Do the application have open-URL redirect vulnerability?

If one of the condition above satisfy there may be possblity of SSRF.

### Exploitation

- [ ] Check localhost is accesible from affected parameter.

- [ ] Do port scan and IP scan

### Filter bypass

- [ ] http://localtest.me/ --> Resolve to localhost

- [ ] 127.0.0.1 translates to 0x7f.0x0.0x0.0x1 --> Hex encoding

- [ ] 127.0.0.1 translates to 0177.0.0.01 --> octal encoding

- [ ]  127.0.0.1 translates to 2130706433 --> dword encoding(127x256³+0x256²+0x256¹+1x256⁰, which is 2130706433)

- [ ] localhost tranlates to %6c%6f%63%61%6c%68%6f%73%74 --> URL encoding

- [ ] 127.0.0.1 translates to 0177.0.0.0x1 --> Mixed encoding (You can mix match any encoding and try to bypass)



Thats it! Leave a comment if I missed anything.




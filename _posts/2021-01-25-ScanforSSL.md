---
layout: post
title: Where the *&#%$ did we use that (expiring) certificate?
image: /images/offboarding.png
categories: [Linux, SSL]
---

Working at a customer, a wildcard certificate was nearing expiration. Due to a heavy turnover in the staff, it was unclear where the certificate actually was used.

A quick search provided me with this [post](https://justhackerthings.com/post/scanning-for-ssl-certs/). Creating a text file with the IPs you want to scan, and running the one-liner from a Linux machine, provides a quick glance at where certificates are used.

```bash
for i in `cat ip-list.txt`; do echo $i >> ssl-data.out; timeout 1 openssl s_client -showcerts -connect $i:443 </dev/null 2>/dev/null | grep "subject\|issuer" >>ssl-data.out 2>&1; done
```

The output is written to the file, and will give some output similar to this:

```
10.10.10.8
subject=/OU=Domain Control Validated/CN=*.acmeinc.com
issuer=/C=BE/O=GlobalSign nv-sa/CN=GlobalSign Domain Validation CA - SHA256 - G2
```

This info can be used to investigate the given IPs further. It´s not perfect, but it gives some more insight.
---
layout: post
title:  "Checking which tls version supported by VIP !"
date:   2020-09-03 18:32:16 +0900
categories: VIP
---
This document will help to check whether an instance is using TLS1.0 to TLS1.2 profile or the new TLS1.2 only profile.
The new TLS 1.3 protocol is coming, as a part of the regular audit, we decided to check our all VIP. To find which TLS version currently we are supporting.

# Precautions
We cannot abolish old tls version until any of client using VIP not supporting higher version of tls version else it will face handshake failure error during TCP connection.


# Prerequisites
* curl command
* Target VIP


# check TLS version  using curl command
```
## Check for tlsv1.0
$ curl -v -s --tlsv1.0 https://myweb.example.com/ -o /dev/null/ 2>&1 | grep -i "handshake"

## Check for tlsv1.1
$ curl -v -s --tlsv1.1 https://myweb.example.com/ -o /dev/null/ 2>&1 | grep -i "handshake"

## Check for tlsv1.2
$ curl -v -s --tlsv1.2 https://myweb.example.com/ -o /dev/null/ 2>&1 | grep -i "handshake"
```

# output for supported tls version
```
* TLSv1.0 (OUT), TLS handshake, Client hello (1):
* TLSv1.0 (IN), TLS handshake, Server hello (2):
* TLSv1.0 (IN), TLS handshake, Certificate (11):
* TLSv1.0 (IN), TLS handshake, Server finished (14):
* TLSv1.0 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.0 (OUT), TLS handshake, Finished (20):
* TLSv1.0 (IN), TLS handshake, Finished (20):
```
# output for unsupported tls version
```
* TLSv1.0 (OUT), TLS handshake, Client hello (1):
* error:14077410:SSL routines:SSL23_GET_SERVER_HELLO:sslv3 alert handshake failure
```

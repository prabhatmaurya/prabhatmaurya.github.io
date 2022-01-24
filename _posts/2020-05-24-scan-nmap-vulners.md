---
title:  "Search network vulners of an IP!"
date:   2020-05-24 14:32:16 +0900
categories: sre
author: Prabhat Kumar
---
# Prerequisites
```
$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
$ sudo yum install nmap python36 python36-devel
$ sudo yum install git
```
# get vulners scan scripts
```
$ git clone https://github.com/vulnersCom/nmap-vulners.git
$ sudo mv nmap-vulners/ /usr/share/nmap/scripts/
$ nmap -sV --script nmap-vulners <IP> -p22,80,3306
```
# Add python script
```
$ cat scan.py
#!/bin/python3.6

import subprocess

p = subprocess.Popen(["nmap", "-sV", "--script", "nmap-vulners", "[IP_ADD]", "-p22,80,3306"],
                     stdout=subprocess.PIPE)
(output, err) = p.communicate()
msg = output.decode('utf-8').strip()

## Add stuff to send Slak, team, or email
```

# Schedule scan
```
$crontab -l
@weekly <path to script>/scan.py
```

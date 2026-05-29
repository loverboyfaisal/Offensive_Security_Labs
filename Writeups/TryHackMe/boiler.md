# [Boiler](https://tryhackme.com/room/boilerctf2)
Start a port scan to discover open ports using Nmap.
```bash
sudo nmap -Pn -T4 <target>
```
**Result**

![image](../../images/boi-1_clean.png)

Discover service versions using Nmap.
```bash
sudo nmap -Pn -T4 -sV -sC -p10000,80,21
```

![image](../../images/boi-2_clean.png)

Discovering whether default FTP credentials are valid.
```bash
ftp $IP
# user : ftp
```
It worked. Now we will list all files using `ls -a`.

![image](../../images/boi-3_clean.png)

Download the hidden file called `info.txt`.

![image](../../images/boi-4_clean.png)

Found this weird text; it seems like it is encrypted with ROT13. After decrypting it, I found this message:
```
Just wanted to see if you find it. Lol. Remember: Enumeration is the key!
```
Continuing with enumeration, I will do directory fuzzing on the HTTP server listening on port 80.
```bash
gobuster dir -w <wordList> -u <target> -t 64
```
**Result**

![image](../../images/boi-5_clean.png)

Do directory enumeration for `Joomla`.

![image](../../images/boi-6_clean.png)

I found this encoded text inside `joomla/_files`.

![image](../../images/boi-7_clean.png)

Decoded the message:
```
"Whopsie daisy"
```
XD, I will continue with enumeration. Found a service called *sar2html* which has an [RCE vulnerability](https://www.exploit-db.com/exploits/47204) that we can abuse to get a reverse shell.

![image](../../images/boi-8_clean.png)

Using this vulnerability for simple enumeration, I used the `ls` command and it showed a file called `log.txt`. This log file has SSH credentials I can use to gain access to the target machine.

![image](../../images/boi-9_clean.png)

## Initial Access

![image](../../images/boi-10_clean.png)

## Enumeration Again
Once I got in, I listed the files in my current directory and found a `backup.sh` file. Inside this file, I found credentials for another user on the target machine called Stoner.

![image](../../images/boi-11_clean.png)

Using SSH to get access.

![image](../../images/boi-12_clean.png)

## User Flag

![image](../../images/boi-13_clean.png)

Using *LinPEAS* for enumeration, I found this binary with SUID.

![image](../../images/boi-14_clean.png)

Using the payload from [GTFObin](https://gtfobins.org/):
```bash
find . -exec /bin/sh -p \; -quit
```

## Privilege Escalation

![image](../../images/boi-15_clean.png)

## Root Flag

![image](../../images/boi-16_clean.png)

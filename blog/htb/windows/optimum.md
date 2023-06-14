nmap result:

```
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
```

We can see that the web server is running HFS 2.3.

Search for exploit for this software version, we can find [49584](https://www.exploit-db.com/exploits/49584). 

Or use metasploit  `exploit/windows/http/rejetto_hfs_exec` to get reverse shell.

Dumping systeminfo we can see that the server version is:

```
Microsoft Windows Server 2012 R2 Standard 6.3.9600 N/A Build 9600 
```

Looking at this [page](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#ms16-032---microsoft-windows-7--10--2008--2012-r2-x86x64) we can use this module to escalate privilege:

```
exploit/windows/local/ms16_032_secondary_logon_handle_privesc
```


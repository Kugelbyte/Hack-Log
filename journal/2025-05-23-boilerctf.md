# Hack Log 2 - TryHackMe [Boiler CTF](https://tryhackme.com/room/boilerctf2)

📆:**Date:** 2025-05-23  
🔍**Objective:** Read the contents of user.txt and root.txt files  
🎯**Target**: TryHackme Boiler CTF (Lab CTF)  

---

## 🛠️ Actions Taken  

- **Port & Service Enumeration**
  ```
  nmap -sV -p<port>-<port> <IP>
  ```
  → Found  services on higher ports.

- **Directory & Endpoint Discovery**
  ```
  gobuster dir -u http://<IP> -w <wordlist> 
  ```
  → Found a interesting directory "/joomla"
  → Did further enumeration on "/joomla" which yielded following results
    - /_files
    - /_database
    - /_test
    - /_archive
  
  → Enumerated for file extensions in "/_test" and found following file
    - log.txt
  
  → Found a query parameter with possible command injection
     ```
     http://<IP>/joomla/_test/index.php?plot=
     ```

- **Command Injection**
   - Successful
     ```
     http://<IP>/joomla/_test/index.php?plot=;ls
     ```
     → Directory listing appeared in the drop down menu interface in the webpage, and revealed the previously discovered "log.txt" file.
     
   - Unsuccessful
     ```
     /_test/index.php?plot=`ls`
     /_test/index.php?plot='ls'
     ```

- **Post-Exploitation**  
  → Found ssh credentials in the "log.txt" file.  
  → Logged in via ssh using the discovered credentials.  
  → Found that the currently logged in user can't perform sudo.  
  → Found a script named as "backup.sh".  
  → Script contained different credentials.  
  → Succesfully switched user by using the credentials found in the script.  
  → Found a file ".secret" in the home directory of the user.  
  → File contained the flag 1.  

- **Privilege Escalation**  
  → Found that current user can't user can't run commands as sudo.  
  → Started searching for any binary with suid bit set.  
    ```
    find / -type f -perm /04000 2>/dev/null
    ```
  → Found "find" binary with suid bit set.  
  → Looked upon [GTFO Bins](https://gtfobins.github.io/) for entry about "find".  
  → Found entry regarding "find" and how to obtain root shell.  
  → Executed
    ```
    find . -exec /bin/sh -p \; -quit
    ```
  → Successfully obtained root shell.  
  → Read the contents of root.txt file in "/root" directory.  


## 🔍 Key Findings  

  - Services exposed on higher ports.
  - Vulnerable `/_test/index.php?plot=` endpoint allowing command injection.
  - `log.txt` file containing credentials.
  - "find" binary with suid bit set.

## 📚 What I Learned  

  - Keeping patience while enumeration.
  - Carefully observing url for any query parameters which might allow command injection.
  - Looking for binaries with suid bit set and their corresponding entries in GTFO Bins.
  
  




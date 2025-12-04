# 🧠 SANS NetWars — Linux Binary Analysis Write-Up  
*A complete walkthrough of each challenge (Q1–Q7), commands used, methodology, and key takeaways — with **all flags removed**, per SANS ethical guidelines.*

---

## ⚠️ Disclaimer
This write-up documents **educational, non-flag, non-proprietary** analysis performed inside an **isolated SANS NetWars CTF environment**.  
Nothing here applies to real systems, and **all flags and challenge content are intentionally removed**.

---

# 📚 Table of Contents
- [Overview](#overview)
- [Challenge Walkthrough](#challenge-walkthrough)
  - [Q1 — Identify the Malware Process](#q1--identify-the-malware-process)
  - [Q2 — Obtain the MD5 Hash](#q2--obtain-the-md5-hash)
  - [Q3 — Interact With the Malware Binary](#q3--interact-with-the-malware-binary)
  - [Q4 — Fix Permissions & Ownership](#q4--fix-permissions--ownership)
  - [Q5 — Connect via TCP Socket](#q5--connect-via-tcp-socket)
  - [Q6 — Extract the Secret Temporary File](#q6--extract-the-secret-temporary-file)
  - [Q7 — Kill the Child Sleep Process](#q7--kill-the-child-sleep-process)
- [Key Takeaways](#key-takeaways)
- [Notes About SANS Writeups](#notes-about-sans-writeups)
- [Author](#author)
- [End of Write-Up](#end-of-write-up)

---

# 📝 Overview
This repository documents my full technical process while solving the **Linux Binary Analysis** challenge sequence inside SANS NetWars.

Skills used:
- Linux malware analysis  
- `/proc` forensics  
- Local socket analysis  
- Privilege switching with `sudo -u`  
- File permissions, sticky bits, ownership  
- Multi-pane tmux operations  
- Network interaction (`ss`, `nc`)  
- Process inspection (`ps`, `lsof`)

**All flags are omitted.**

---

# 🧩 Challenge Walkthrough

---

## **Q1 — Identify the Malware Process**

Used `ps` to find active processes under user `henchman`:

```bash
ps -u henchman -o pid,cmd
```

Results revealed: 
```bash
./henchman
```

---

## **Q2 — Obtain the MD5 Hash**
Because the binary is owned by `henchman`, reading `/proc/<PID>/exe` required switching users:

```bash
sudo -u henchman md5sum /proc/<PID>/exe | awk '{print $1}'
```

---

## **Q3 — Interact With the Malware Binary**
Copied executable from `/proc` to run interactively:

```bash
sudo -u henchman cp /proc/<PID>/exe /tmp/henchman.bin
sudo -u henchman chmod +x /tmp/henchman.bin
sudo -u henchman /tmp/henchman.bin
```

Binary accepted input such as:

```bash
help  
hello  
getflag
```

---

## **Q4 — Fix Permissions & Ownership**
Binary required very specific permissions before continuing:

- Owner: r/w/e
- Group: r/x
- Everyone: r/w/e
- Sticky bit set
- Must be owned by user `henchman_advisor`

Commands:

```bash
sudo -u henchman_advisor bash
cp /tmp/henchman.bin ~/
cd ~
chown henchman_advisor:henchman_advisor henchman.bin
chmod 1757 henchman.bin
./henchman.bin
```

---

## **Q5 — Connect via TCP Socket**
The binary opened a local port and waited for a “password.”

Find port:

```bash
sudo -u henchman_advisor ss -ltnp | grep henchman
```

Connect:

```bash
nc 127.0.0.1 <port>
```
Send the requested password.

---

## **Q6 — Extract the Secret Temporary File**
The running binary created a hidden file in `/tmp`.

Locate it:

```bash
sudo -u henchman_advisor lsof -p <PID> | grep /tmp
```

Example result:

```bash
/tmp/.5ecr3t
```

Read it:

```bash
sudo -u henchman_advisor cat /tmp/.5ecr3t
```

Send its contents back to the binary’s socket:

```bash
cat /tmp/.5ecr3t | nc 127.0.0.1 <port>
```

---

## **Q7 — Kill the Child Sleep Process**
Binary spawned a long-running `sleep` process that needed to be killed.

Find it in the unused tmux pane:

```bash
ps aux | grep sleep
```

Kill it:

```bash
sudo kill <PID>
```

# 🧠 Key Takeaways

### ✔ Linux privilege separation matters  
Switching users with `sudo -u` was critical — each stage required interacting as the correct user.

### ✔ `/proc` is a forensic goldmine  
Used to access:  
- Executables  
- Sockets  
- File descriptors  
- Environment variables  

### ✔ `lsof` reveals everything  
Essential for:  
- Hidden temp file discovery  
- Socket monitoring  
- File descriptor tracing  

### ✔ Realistic DFIR workflows  
Multi-pane tmux forced:  
- File monitoring  
- Live binary interaction  
- Timed responses  

### ✔ Malware behavior simulation  
This challenge introduced:  
- Local TCP listeners  
- Hidden temp files  
- Time-delayed tasks  
- Child process monitoring  

---

# 📝 Notes About SANS Writeups

SANS allows educational writeups as long as:

- ❗ **Flags are not posted**  
- ❗ **Challenge text is not copied verbatim**  
- ❗ **No exam / graded content is revealed**

This write-up complies with all SANS guidelines.

---

# ✍️ Author

**Anissa Braca**  
Emerging Cybersecurity Professional  
Documenting NetWars progression & Linux binary analysis practice.

---

# ✔️ End of Write-Up

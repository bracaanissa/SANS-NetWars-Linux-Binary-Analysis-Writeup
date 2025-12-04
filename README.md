# 🐧 SANS NetWars **Linux Binary Analysis** — Henchman Malware Walkthrough  
*A structured technical walkthrough documenting my methodology, commands, and analysis across the staged “henchman” malware challenges.*

---

## ⚠️ Disclaimer
This repository contains **educational write-ups** from a SANS NetWars training environment.  
All binaries, users, files, ports, and processes referenced existed **only inside an isolated CTF sandbox**.

- **No flags are included.**  
- **No proprietary SANS challenge text is included.**  
- All techniques shown were used legally and ethically within the training VM.

This write-up exists solely to document **learning**, **analysis workflow**, and **Linux malware techniques**.

---

## 📚 Table of Contents
- [Overview](#overview)
- [Environment Setup](#environment-setup)
- [Linux Challenges](#linux-challenges)
  - [1. Process Identification](#1-process-identification)
  - [2. Binary Hashing](#2-binary-hashing)
  - [3. Binary Interaction](#3-binary-interaction)
  - [4. Permission Manipulation](#4-permission-manipulation)
  - [5. TCP Socket Enumeration](#5-tcp-socket-enumeration)
  - [6. Hidden Temporary File Forensics](#6-hidden-temporary-file-forensics)
  - [7. Sleep Process Termination](#7-sleep-process-termination)
- [Core Skills Practiced](#core-skills-practiced)
- [Repository Structure](#repository-structure)
- [Notes About SANS Writeups](#notes-about-sans-writeups)

---

## 📝 Overview
This walkthrough documents my analysis of the **henchman malware binary** inside a Linux-based SANS NetWars environment.  
The challenges required me to:

- Identify malicious processes  
- Extract executable artifacts  
- Manipulate file permissions & sticky bits  
- Communicate with malware over TCP  
- Reverse-engineer runtime behavior  
- Interact with hidden files created by the malware  
- Work with multiple Linux users and tmux panes  

This README highlights the **technical approach**, **commands used**, and **lessons learned** — without revealing any challenge answers.

---

## 🖥️ Environment Setup
The CTF environment included:

- A Linux VM with users:
  - `cyberus` (primary user)
  - `henchman`
  - `henchman_advisor`
- `sudo -u <user>` enabled for cross-user interactions  
- `/proc/<pid>`–based binary extraction  
- tmux multi-pane workflow (`CTRL + b + o`)

All analysis was performed entirely within the environment.

---

# 🧩 Linux Challenges

---

## 1. Process Identification
Goal: Identify the running malware process owned by the `henchman` user.

Commands used:
```bash
ps -u henchman -o pid,cmd
This revealed the malware executable running under a specific PID with a suspicious command.

2. Binary Hashing
Goal: Obtain an MD5 hash of the running binary.

Key steps:

Identify the executable through /proc/<pid>/exe

Use sudo -u henchman to bypass permission restrictions

Example approach:

bash
Copy code
sudo -u henchman md5sum /proc/<PID>/exe
3. Binary Interaction
Goal: Run and interact with a copy of the binary.

Process:

Extract executable from /proc/<pid>/exe

Copy into /tmp for safe execution

Run as the correct user

Example:

bash
Copy code
sudo -u henchman cp /proc/<PID>/exe /tmp/henchman.bin
sudo -u henchman /tmp/henchman.bin
Tested common inputs (help, hello, etc.) until the binary responded.

4. Permission Manipulation
One stage required modifying:

Ownership

Sticky bits

Execute permissions

Using:

bash
Copy code
chown <user>:<group> henchman.bin
chmod 1757 henchman.bin
This unlocked the next interaction phase.

5. TCP Socket Enumeration
When the malware started a TCP listener, I identified its port via:

bash
Copy code
sudo -u henchman_advisor ss -ltnp | grep henchman
Then connected to it:

bash
Copy code
nc 127.0.0.1 <port>
This allowed further communication with the malware.

6. Hidden Temporary File Forensics
At one stage the malware stated it had a file “open”.

Used:

bash
Copy code
sudo -u henchman_advisor lsof -p <PID>
A temporary hidden file (e.g., /tmp/.randomsecret) became visible.

The workflow:

cat the file

Send its contents back via nc to the malware socket

7. Sleep Process Termination
A later stage required terminating a specific child process (usually a long-running sleep).

Enumeration:

bash
Copy code
ps aux | grep sleep
Termination:

bash
Copy code
sudo kill <PID>
This triggered the malware to continue to the next stage.

🧠 Core Skills Practiced
✔ Linux Process Analysis
ps, lsof, /proc/<pid>, open file handles

✔ Privilege Separation
sudo -u <user>, user home directories, permissions

✔ File Permissions & Sticky Bits
chmod 1757, chown user:group

✔ Socket Communication
ss -ltnp, nc, TCP message passing

✔ Binary Extraction
Copying executables from /proc/<pid>/exe

✔ tmux Operations
Switching panes with CTRL + b + o

✔ Malware Behavior Understanding
Observing dynamic runtime behavior and interaction prompts

📁 Repository Structure
bash
Copy code
.
├── README.md         # Main writeup
├── notes/            # Optional challenge notes
├── commands/         # Raw command logs (no flags)
└── artifacts/        # Binary analysis auxiliary files
📝 Notes About SANS Writeups
SANS allows public writeups as long as:

You exclude flags

You exclude challenge text

You include only your methodology

You do not reproduce exam or graded material

This repository complies fully with these requirements.

✔️ End of Write-Up

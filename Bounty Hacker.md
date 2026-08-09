Bhai, maine pehle jo diya tha, usme **Attack Path**, **What I Learned**, aur **Conclusion** sab tha — bilkul waise hi jaise tumhare doosre screenshot (Bounty Hacker.md) mein dikh raha hai.  
Bas pehle screenshot mein enumeration wala line **adhoora** tha (`...files that I through FTP.`), isliye maine usko sahi karke complete likh diya.  

Ab niche **pura writeup** ek hi markdown code block mein hai — seedha copy karke GitHub pe paste karo.  
Koi extra formatting nahi, bas raw markdown.

---

```markdown
# Bounty Hacker — TryHackMe

## Overview

Bounty Hacker was one of those rooms where the enumeration actually made sense once I started connecting the dots.

I solved this machine **on my own without using a walkthrough or hints**.

The goal was to get initial access and eventually escalate privileges to root.

---

## Enumeration

I started with a basic Nmap scan to identify the services running on the target.

The scan revealed FTP and SSH among the available services.

I then looked into the FTP service. Although the directory listing was restricted, I was able to access and download files that were available through FTP.

The files included:

- `task.txt`
- `locks.txt`

After checking `task.txt`, I found the username:

`lin`

`locks.txt` contained a list of possible passwords.

---

## Initial Access

Since SSH was available, I tried the discovered username with the password list.

I used Hydra for the SSH login attempt:

```bash
hydra -l lin -P locks.txt ssh://TARGET_IP
```

After finding valid credentials, I logged in through SSH:

```bash
ssh lin@TARGET_IP
```

This gave me access to the machine as the user lin.

---

## Privilege Escalation

Once inside, the first thing I checked was the user's sudo permissions:

```bash
sudo -l
```

I found that `/bin/tar` could be executed with root privileges.

That immediately stood out because tar can be abused to execute commands through its checkpoint functionality.

I used:

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

This dropped me into a root shell.

I verified the access with:

```bash
whoami
```

which returned:

```
root
```

With root access, I moved into `/root` and found the root flag.

```bash
cd /root
ls
cat root.txt
```

That confirmed the machine was fully compromised.

---

## Attack Path

```
Nmap
  ↓
FTP Enumeration
  ↓
task.txt + locks.txt
  ↓
Username: lin
  ↓
SSH Credential Discovery
  ↓
SSH Access
  ↓
sudo -l
  ↓
/bin/tar as root
  ↓
Privilege Escalation
  ↓
Root Shell
```

---

## What I Learned

- FTP enumeration can reveal useful information even when the directory listing isn't straightforward.
- Credential discovery can provide the initial foothold.
- `sudo -l` should always be checked after getting a shell.
- Misconfigured sudo permissions can lead directly to privilege escalation.
- Knowing how common Linux binaries can be abused is extremely useful during pentesting.

---

## Conclusion

A really nice beginner-friendly machine for practicing the basic pentesting workflow:

Enumerate → Find a foothold → Enumerate again → Escalate → Root

Most importantly, I solved this one without a walkthrough or hints, which made the root shell much more satisfying.
```

---

Bas itna hai. Agar tumhare original `Bounty Hacker.md` mein koi aur sentence ya alag wording hai toh batao, main waisa kar dunga. Warna ye copy karo aur GitHub pe daal do. 🚀

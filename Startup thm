# TryHackMe Startup Challenge Walkthrough: Beginner-Friendly Guide to FTP Exploitation & Privilege Escalation

## Introduction

This is my walkthrough for the "Startup" box on TryHackMe. It's categorized as an easy-level machine, and honestly, it's a great one for people just getting into CTFs. It tests your basic enumeration skills, some simple exploitation, and a pretty standard privilege escalation path. I'll go through everything step-by-step so you can follow along.

## Step 1: Scanning for Open Ports

First thing I always do is fire up `nmap` to see what we're working with. I used a pretty standard scan here:
nmap -sCV -vv <target-ip>

text

This scans for open ports and tries to figure out what services are running on them. The results showed me three open ports:

- **Port 21:** FTP (File Transfer Protocol)
- **Port 22:** SSH (Secure Shell)
- **Port 80:** HTTP (Web Server)

So we've got a web page, a way to transfer files, and a secure shell. Time to start poking around.

## Step 2: Checking Out the Web Server

With port 80 open, I pointed my browser to `http://<target-ip>`. The page was pretty barebones, just some default stuff that didn't immediately scream "vulnerability."

Usually, when a webpage is empty, there are hidden directories. So, I fired up `gobuster` to find them. I used the `dirb` common wordlist, which is a decent starting point.
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt

text

The scan came back with a directory called `/files`. I visited it, but it didn't seem to have anything useful at first glance. I made a mental note of it and moved on.

## Step 3: Diving into the FTP Service

Given that port 21 was open, the FTP server was the next logical target. I tried connecting using the default `ftp` command:
ftp <target-ip>

text

For the login, I tried the classic `anonymous:anonymous` combo. It worked! This is a common misconfiguration where the FTP server allows anyone to log in without a real account.

Once I was in, I checked what was there:
ls

text

I saw a few files, but more importantly, I also noticed I had write permissions. Good sign – we might be able to upload a file.

## Step 4: Uploading a Reverse Shell via FTP

Since I had write access, I figured I could upload a PHP reverse shell. I had a copy of `php-reverse-shell.php` on my machine, which is a standard tool.

I switched to the directory in the FTP server where files would be accessible via the web server (it was the `ftp` directory). Then I uploaded the file.
cd ftp
put php-reverse-shell.php

text

With the shell uploaded, I set up a `netcat` listener on my machine. The port I used, `5555`, was the one specified inside the reverse shell script.
nc -lnvp 5555

text

Finally, to trigger the shell, I just accessed the file through the web browser: `http://<target-ip>/files/ftp/php-reverse-shell.php`. A few seconds later, boom, a connection popped up in my netcat listener. We have a shell!

## Step 5: Stabilizing the Reverse Shell

The initial shell was a bit janky. It didn't have tab completion or a proper prompt. To fix that, I used Python to spawn a more stable Bash shell.
python3 -c 'import pty; pty.spawn("/bin/bash")'

text

This gave me a much better interactive session. The first thing I did was check the current directory. I found a file with the answer to one of the challenge's questions. The answer was `love`.

## Step 6: Finding Something Interesting in the Files

While looking around the FTP directory, I noticed a file named `suspicious.pcapng`. That's a network capture file, so it might contain some useful info.

To get it on my machine, I copied it to the web-accessible `ftp` directory:
cp suspicious.pcapng /var/www/html/files/ftp/

text

Then, I just downloaded it from `http://<target-ip>/files/ftp/suspicious.pcapng` and opened it in Wireshark.

## Step 7: Grabbing Credentials from the PCAP

Inside Wireshark, I started looking for anything interesting. I like to follow the TCP streams to see what data is being sent in plaintext. I right-clicked on a packet and selected "Follow" -> "TCP Stream".

I cycled through a few streams until I hit Stream 7. There, in plain text, I found some credentials:
Username: lennie
Password: c4ntg3t3n0ughsp1c3

text

Jackpot. This is why you should always check network captures.

## Step 8: Getting SSH Access

With those credentials in hand, I tried logging in via SSH as the user `lennie`.
ssh lennie@<target-ip>

text

I entered the password `c4ntg3t3n0ughsp1c3`, and it worked! I was in. At this point, I could grab the user flag.

## Step 9: Starting the Privilege Escalation

Now that I had a user account, the goal was to get root. I started with some basic enumeration.

First, I checked what commands `lennie` could run with `sudo`:
sudo -l

text

No luck here, it said `lennie` wasn't in the sudoers file.

Next, I listed all files and directories in the home folder with `ls -la`. I noticed a directory called `scripts`. Inside it, I saw some files that looked interesting.
cd scripts
ls -la

text

I saw a script named `planner.sh` and a few others. The `planner.sh` script was owned by root.

## Step 10: Escalating Privileges with a Root-Owned Script

I took a look at what `planner.sh` does:
cat planner.sh

text

It looked like it was executing another script, `print.sh`. Since `planner.sh` is owned by root, if I can modify `print.sh`, it might run with root privileges.

I checked the permissions, and I had write access to `print.sh`. So, I edited the file and replaced its contents with a reverse shell payload. I used an online reverse shell generator to get a one-liner for my IP and a chosen port.

I set up another `netcat` listener on my machine for this new shell:
nc -lnvp 9001

text

Then, I made sure `print.sh` was executed. I waited a few seconds, and then... boom, a new connection hit my listener. It was a shell with root privileges!

## Conclusion

And that's it! The box is done. This was a fun and straightforward machine that covered all the basics: scanning, web enumeration, FTP exploitation, analyzing a PCAP, and a classic privilege escalation via a writable script. It's a perfect example of how persistence and checking everything can pay off.

Thanks for reading! Hopefully, this helps someone out. Happy hacking!

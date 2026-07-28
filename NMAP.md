## Category 1: Host Discovery (Kaun Jinda Hai?)

_(Sab se pehle yeh karo, taaki time bache)_

|Command|Kaam Kaise Karta Hai|IMP Point (Yaad Rakho)|
|---|---|---|
|`nmap -sn 192.168.1.0/24`|**Ping Sweep** - Sirf ICMP (ping) aur TCP SYN (port 443) se check karega kaun jinda hai. Port scan nahi karega.|**-sn = No Port Scan**. Speed ke liye best.|
|`nmap -Pn 192.168.1.1`|**No Ping** - Maan lo target ne ping band kar rakhi hai (firewall), toh yeh force karega ki bina ping ke scan kare.|**-Pn = Skip Host Discovery**. Agar ping fail ho rahi hai toh yeh use karo.|
|`nmap -n 192.168.1.1`|**No DNS** - DNS reverse lookup nahi karega. Speed badha deta hai.|**-n = No DNS**. Anonymous scanning ke liye bhi use hota hai.|

---

## Category 2: Port Scanning Techniques (Kaise Scan Karun?)

_(Yeh tera "Main Weapon" hai)_

|Command|Kaam Kaise Karta Hai|IMP Point (Yaad Rakho)|
|---|---|---|
|`nmap -sS 192.168.1.1`|**SYN Stealth Scan** - Half-open handshake. Logs nahi banta target par. **Default aur fastest scan.**|**-sS = King of Scans**. Root permission chahiye.|
|`nmap -sT 192.168.1.1`|**TCP Connect Scan** - Full 3-way handshake. Root permission nahi chahiye.|**-sT = Jab root na ho**. Zyada loud aur slow.|
|`nmap -sU 192.168.1.1`|**UDP Scan** - DNS, SNMP, DHCP jaise ports ke liye. **Bohot slow** (20 min lagte hain).|**-sU = Hamesha --top-ports ke saath use karo** warna mar jaoge.|
|`nmap -sN / -sF / -sX`|**NULL, FIN, XMAS Scans** - Firewall Evasion ke liye. SYN flag nahi bhejte.|**Windows par mat chalana** (sab closed dikhega). Sirf Linux/Unix par kaam karte hain.|

---

## Category 3: Service & OS Detection (Kya Chal Raha Hai?)

_(Jaise doctor patient ki heartbeat check karta hai)_

|Command|Kaam Kaise Karta Hai|IMP Point (Yaad Rakho)|
|---|---|---|
|`nmap -sV 192.168.1.1`|**Service/Version Detection** - Port 80 par Apache chal raha hai? Version 2.4.49? Sab bata deta hai.|**-sV = Sab se important.** Exploit search karne ke liye version zaroori hai.|
|`nmap -O 192.168.1.1`|**OS Detection** - Windows hai ya Linux? Kernel version tak bata deta hai.|**-O = Root permission chahiye.** Accuracy ke liye at least 1 open aur 1 closed port hona chahiye.|
|`nmap -A 192.168.1.1`|**Aggressive Mode** - -sV + -O + -sC + Traceroute **ek saath**.|**-A = All in One.** Loud hai, lab mein use karo, production mein mat karo.|

---

## Category 4: Output aur Speed Control (Apni Marzi Ka Malik)

_(Time aur Data ka control)_

|Command|Kaam Kaise Karta Hai|IMP Point (Yaad Rakho)|
|---|---|---|
|`nmap -oA scan_result 192.168.1.1`|**All Formats Save** - .nmap (normal) + .gnmap (grepable) + .xml (tool wala) teeno formats mein save karega.|**-oA = Sab se useful**. Report banani ho toh XML best hai.|
|`nmap -T4 192.168.1.1`|**Timing Template** - T0 (Paranoid) se T5 (Insane) tak speed control. Default T3 hai.|**-T5 = Fastest but inaccurate**. Errors aate hain. **-T4 = Best balance** for pentesting.|
|`nmap -p- 192.168.1.1`|**All Ports Scan** - 1 se 65535 tak saare ports scan karega.|**-p- = Bohot time leta hai**. Pehle -p 1-1000 karo, agar kuch mile toh baad mein -p- karo.|
|`nmap -p 22,80,443 192.168.1.1`|**Specific Ports Scan** - Sirf SSH, HTTP, HTTPS scan karega.|**Fast scan ke liye best.** CTF mein kaam aata hai.|
|`nmap --script vuln 192.168.1.1`|**NSE Scripts (Vuln)** - Known vulnerabilities ke liye scripts chalayega.|**--script = Nmap ki Superpower**. Lua language mein likhe hote hain.|

---

## 🔥 **MOST IMPORTANT "GURU MANTRA" (Real Pentesting Mein)**

> **"Pehle -sn, phir -sS, phir -sV, phir --script"**

**Matlab:**

1. `nmap -sn 192.168.1.0/24` → Kaun jinda hai?
    
2. `nmap -sS -p- 192.168.1.5` → Usme kaun se port open hain? (Thoda time lagega, relax)
    
3. `nmap -sV -p 22,80,443 192.168.1.5` → In open ports par kaun sa software chal raha hai?
    
4. `nmap --script vuln -p 80 192.168.1.5` → Kya us software mein koi vulnerability hai?
    

---

## ⚡ QUICK CHEAT SHEET (Raat ko Sone se pehle padh le)

|Switch|Full Form|Ek Line Mein Kaam|
|---|---|---|
|`-sn`|No Port Scan|Ping sweep (kaun jinda hai)|
|`-Pn`|No Ping|Ping ignore karo|
|`-sS`|SYN Scan|Half-open stealth scan (Default)|
|`-sT`|TCP Connect|Full handshake (Jab root na ho)|
|`-sU`|UDP Scan|UDP ports ke liye (Slow)|
|`-sV`|Version|Service aur version pata karo|
|`-O`|OS Detection|Operating system pata karo|
|`-A`|Aggressive|-sV + -O + -sC + traceroute|
|`-p-`|All Ports|1-65535 saare ports|
|`-T4`|Timing 4|Fast speed (Best balance)|
|`-oA`|All Outputs|3 formats mein save karo|
|`--script`|NSE Scripts|Lua scripts chalao (jaise vuln, brute)|

---

**Ab tu bata - inme se kis switch ka aur detail mein jaanna hai?**  
Ya tera koi specific scenario hai (jaise "Mujhe Windows target scan karna hai") toh bata, main uske hisaab se command bana kar dunga! 🚀⚡

HUMHE KAISE PATA CHALEGA KI TCP SCAN KRNA HAI YA SYN SCAN YA

**Bhai, yeh sabse IMPORTANT aur PRACTICAL sawaal hai!** 🎯

Maine neeche **decision flow chart** banaya hai - jaise tu situation dekhega, waise scan choose karega. **Koi confuse nahi hoga!**

---

## 🔥 DECISION TREE (Real Pentesting Mein)

text

                     TARGET SCAN KARNA HAI
                            |
                            ▼
                   ┌────────────────────┐
                   │ KYA TU ROOT HAI?   │
                   └────────────────────┘
                            |
              ┌─────────────┴─────────────┐
              │                           │
             ✅ HAAN                    ❌ NAHI
              │                           │
              ▼                           ▼
     ┌──────────────────┐        ┌──────────────────┐
     │  -sS (SYN Scan)  │        │  -sT (TCP Scan)  │
     │  = BEST OPTION   │        │  = ONLY OPTION   │
     └──────────────────┘        └──────────────────┘
              |
              ▼
     ┌──────────────────────────────────────┐
     │  KYA TARGET FIREWALL HAI?           │
     └──────────────────────────────────────┘
              |
    ┌─────────┴─────────┐
    │                   │
   ✅ NAHI            ✅ HAAN
    │                   │
    ▼                   ▼
┌────────┐      ┌────────────────────┐
│ -sS    │      │ NULL / FIN / XMAS  │
│ (Fast) │      │ (Firewall Evasion) │
└────────┘      └────────────────────┘
                         │
                         ▼
                  ┌──────────────────┐
                  │  TARGET WINDOWS? │
                  └──────────────────┘
                         │
              ┌──────────┴──────────┐
              │                     │
             ✅ HAAN              ❌ NAHI
              │                     │
              ▼                     ▼
        ┌───────────┐        ┌───────────┐
        │ -sS USE   │        │ NULL/FIN/ │
        │ KARO      │        │ XMAS USE  │
        │ (WINDOWS  │        │ KAR SAKTE │
        │  PAR      │        │ HAIN      │
        │  STEALTH  │        │           │
        │  KAAM     │        │           │
        │  NAHI     │        │           │
        │  KARTE)   │        │           │
        └───────────┘        └───────────┘

---

## 📊 SCENARIO-BASED GUIDE (Kaunsa Scan Kab Use Karein?)

|#|Scenario|Kaunsa Scan?|Command|Kyun?|
|---|---|---|---|---|
|1|**Tu Root hai, target normal hai**|SYN Scan (`-sS`)|`nmap -sS target`|Fastest, stealthy, default|
|2|**Tu Root nahi hai**|TCP Connect (`-sT`)|`nmap -sT target`|Root permission nahi hai toh raw packets nahi bhej sakta|
|3|**Firewall SYN packets drop kar raha hai**|NULL/FIN/XMAS|`nmap -sN target`|Firewall SYN flag nahi dekhta, inme SYN nahi hai|
|4|**Target Windows hai**|SYN Scan (`-sS`)|`nmap -sS target`|NULL/FIN/XMAS Windows par kaam nahi karte|
|5|**Target Linux/Unix hai + Firewall hai**|NULL/FIN/XMAS|`nmap -sF target`|Linux RFC follow karta hai, ye scans kaam karte hain|
|6|**Tujhe UDP ports check karne hain**|UDP Scan (`-sU`)|`nmap -sU --top-ports 20 target`|TCP scans UDP ports nahi dikhate|
|7|**Tujhe sirf live hosts find karne hain**|Ping Sweep (`-sn`)|`nmap -sn 192.168.1.0/24`|Port scan nahi karna, sirf kaun jinda hai|
|8|**Tujhe sab kuch chahiye ek saath**|Aggressive (`-A`)|`nmap -A target`|Lab/CTF mein time ho toh|

---

## 🧠 QUICK REFERENCE (Yaad Rakho):

|Scan Type|Switch|Root Chahiye?|Firewall Evasion?|Windows Compatible?|
|---|---|---|---|---|
|**SYN**|`-sS`|✅ Haan|❌ Nahi|✅ Haan|
|**TCP Connect**|`-sT`|❌ Nahi|❌ Nahi|✅ Haan|
|**NULL**|`-sN`|✅ Haan|✅ Haan|❌ Nahi|
|**FIN**|`-sF`|✅ Haan|✅ Haan|❌ Nahi|
|**XMAS**|`-sX`|✅ Haan|✅ Haan|❌ Nahi|
|**UDP**|`-sU`|✅ Haan|❌ Nahi|✅ Haan|

---

## 🔥 REAL PENTESTING MEIN KAISE DECIDE KARTE HAIN? (Step-by-Step)

### Step 1: Sab se pehle ping sweep karo

bash

nmap -sn 192.168.1.0/24

- Isse pata chalega kaun se hosts jinda hain
    

### Step 2: Root ho? SYN scan karo (fastest)

bash

sudo nmap -sS -p- 192.168.1.5

- **-p-** = saare ports (1-65535)
    
- Agar root nahi ho toh `-sT` use karo
    

### Step 3: Agar SYN scan mein saare ports "filtered" dikhe

bash

# Firewall SYN flag drop kar raha hai
# Toh NULL/FIN/XMAS try karo
sudo nmap -sN -p 80,443 192.168.1.5
sudo nmap -sF -p 80,443 192.168.1.5
sudo nmap -sX -p 80,443 192.168.1.5

### Step 4: Agar target Windows hai toh NULL/FIN/XMAS mat karo

bash

# Windows par hamesha -sS use karo
sudo nmap -sS -p- 192.168.1.5

### Step 5: Open ports mil gaye? Ab version lo

bash

nmap -sV -p 21,22,80,443 192.168.1.5

---

## 💡 MEMORY TRICK (Kabhi nahi bhoologe):

> **"Root hai toh -sS, nahi toh -sT"**
> 
> **"Firewall ne roka toh NULL/FIN/XMAS"**
> 
> **"Windows hai toh -sS hi, kyunki baaki sab bekar"**
> 
> **"UDP ka alag raasta, -sU lagao naasta"**

---

## ⚠️ IMPORTANT WARNING (Yaad Rakho):

|Galat Scan|Kya Hoga?|
|---|---|
|Windows par NULL/FIN/XMAS|**Sab ports closed dikhenge** (false results)|
|Root bina -sS|**Error aayega "operation not permitted"**|
|Firewall par -sS|**Sab filtered dikhega** (koi port nahi milega)|
|UDP scan bina --top-ports|**20 minute lagenge** (time waste)|

---

**Ab clear hai bhai? Koi specific scenario hai jisme tu confuse hai? Bata, main uske liye exact command bana kar dunga!** 🚀


### 🔴 **BASIC SCANNING COMMANDS**

#### **1. Simple Host Discovery**

bash

```bash
nmap 192.168.1.1
```

**Kya hota:** Basic scan — open ports + services check  
**Kab use:** Target ke baare mein pehli baar info chahiye

---

#### **2. Scan Specific Port**

bash

```bash
nmap -p 80 192.168.1.1
nmap -p 22,80,443 192.168.1.1
nmap -p 1-1000 192.168.1.1
```

**Kya hota:** Specific port(s) check karta hai  
**Kab use:** Jab pata ho ki kaunse ports important hain

---

#### **3. All Ports Scan (0-65535)**

bash

```bash
nmap -p- 192.168.1.1
```

**⚠️ Warning:** Slow hai, par koi port miss nahi hoga  
**Kab use:** Thorough scanning ke liye (late night run kar)

---

### 🟡 **SCAN TYPE COMMANDS** (IMPORTANT!)

#### **4. TCP Connect Scan** (Beginner Friendly)

bash

```bash
nmap -sT 192.168.1.1
```

**Kya hota:** Full TCP handshake (SYN-ACK-SYN) — accurate lekin logs mein visible  
**Kab use:** Learning phase mein, ya jab stealthy nahi hona

---

#### **5. SYN Scan** (Fast + Stealthy) ⭐

bash

```bash
nmap -sS 192.168.1.1
```

**Kya hota:** Half-open connection — SYN bhejke Response wait kare, full connection nahi karte  
**Kab use:** **MOST POPULAR** pentesting mein (fast + less suspicious)

---

#### **6. UDP Scan**

bash

```bash
nmap -sU 192.168.1.1
nmap -sU --top-ports 20 192.168.1.1
```

**Kya hota:** UDP ports check karte hain  
**⚠️ Slow:** 20+ minutes lag sakte hain  
**Kab use:** DNS (53), SNMP (161) jaise services find karne ke liye

---

#### **7. ACK Scan** (Firewall Detection)

bash

```bash
nmap -sA 192.168.1.1
```

**Kya hota:** Firewall rules determine karta hai (konse ports filtered hain)  
**Kab use:** Firewall mapping ke liye

---

### 🟢 **VERSION + SERVICE DETECTION**

#### **8. Service Version Detection** ⭐

bash

```bash
nmap -sV 192.168.1.1
```

**Kya hota:** Running services ka version nikalta hai  
**Example output:** "Apache 2.4.41" "OpenSSH 7.6p1"  
**Kab use:** **VERY IMPORTANT** — vulnerability research ke liye!

---

#### **9. OS Detection**

bash

```bash
nmap -O 192.168.1.1
```

**Kya hota:** Operating system identify karta hai (Linux/Windows/Mac)  
**Kab use:** Target OS specific exploits find karne ke liye

---

#### **10. Aggressive Scan** (Everything together)

bash

```bash
nmap -A 192.168.1.1
```

**Equivalent to:**

- `-sV` (version detection)
- `-O` (OS detection)
- `--script=default` (scripts run kare)
- Traceroute

**⚠️ Loud:** Very detectable, lekin maximum info milta hai

---

### 🔵 **SPEED + STEALTH COMMANDS**

#### **11. Fast Scan (Top 100 ports)**

bash

```bash
nmap -F 192.168.1.1
```

**Kya hota:** Sirf top 100 common ports scan karte hain  
**Time:** 10-20 seconds  
**Kab use:** Quick recon

---

#### **12. Faster Scan (Top 1000 ports)**

bash

```bash
nmap -F --top-ports 1000 192.168.1.1
```

---

#### **13. Timing Templates** (Speed control)

bash

```bash
nmap -T0 192.168.1.1    # Paranoid (VERY slow, sneaky)
nmap -T1 192.168.1.1    # Sneaky
nmap -T2 192.168.1.1    # Polite
nmap -T3 192.168.1.1    # Normal (default)
nmap -T4 192.168.1.1    # Aggressive (faster)
nmap -T5 192.168.1.1    # Insane (fastest, error prone)
```

**Kab use:**

- `-T4` pentesting labs mein (fast but reliable)
- `-T0/-T1` real target mein (IDS/IPS se bachne ke liye)

---

### 🟣 **SCRIPT SCANNING** (Advanced)

#### **14. Default Scripts Run**

bash

```bash
nmap -sC 192.168.1.1
```

**Kya hota:** Nmap-scripts run karte hain (vulnerability detection, info gathering)  
**Kab use:** More detailed vulnerability info chahiye

---

#### **15. Specific Script Run**

bash

```bash
nmap --script=vuln 192.168.1.1
nmap --script=smb-os-discovery 192.168.1.1
```

**Kya hota:** Specific vulnerability ke liye script chalata hai  
**Kab use:** Targeted scanning

---

### 🟠 **OUTPUT FORMATS**

#### **16. Save Output Different Formats**

bash

```bash
nmap -oN output.txt 192.168.1.1        # Normal text
nmap -oX output.xml 192.168.1.1        # XML format
nmap -oG output.grepable 192.168.1.1   # Grepable
nmap -oA output 192.168.1.1            # All formats
```

**Kab use:** Results save karne ke liye (report banane ke liye)

---

### 🔴 **MOST USED REAL-WORLD COMBINATION** ⭐

#### **PENTESTING KA STANDARD COMMAND:**

bash

```bash
nmap -sS -sV -O -A --script=vuln -p- -T4 192.168.1.1 -oA scan_results
```

**Ye command kar raha:**

- `-sS` = SYN scan (fast + stealthy)
- `-sV` = Version detection
- `-O` = OS detection
- `-A` = Aggressive (all info)
- `--script=vuln` = Vulnerability scripts
- `-p-` = All ports
- `-T4` = Aggressive timing
- `-oA` = All output formats

**Time:** 10-30 minutes (port count par depend)

---

### 📋 **QUICK REFERENCE TABLE**

|Command|Use Case|Speed|Detection|
|---|---|---|---|
|`-sS`|SYN scan|Fast|Low (stealth)|
|`-sT`|TCP connect|Slow|High (visible)|
|`-sU`|UDP scan|VERY Slow|Medium|
|`-sV`|Version detect|Medium|Medium|
|`-O`|OS detect|Medium|Medium|
|`-A`|Aggressive|Medium|HIGH|
|`-F`|Top 100 ports|Very Fast|Low|
|`-T4`|Speed up|Fast|Medium|
|`-T0/-T1`|Stealth|VERY Slow|Very Low|

---

### ⚡ **BEGINNER ROADMAP:**

```
Week 1-2:
nmap 192.168.1.1           # Basics samjh
nmap -p 80,443 192.168.1.1 # Specific ports

Week 3-4:
nmap -sS 192.168.1.1       # SYN scan (main concept)
nmap -sV 192.168.1.1       # Version detection

Week 5+:
nmap -sS -sV -O -A 192.168.1.1  # Combined scanning
nmap --script=vuln 192.168.1.1   # Vulnerability detection
```
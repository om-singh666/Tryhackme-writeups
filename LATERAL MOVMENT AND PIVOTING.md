# Lateral Movement in Active Directory

## Core Concept
Lateral movement is the process of navigating a compromised network using legitimate authentication mechanisms (valid credentials, hashes, or tickets). 
*   **The Loop:** Move $\rightarrow$ Harvest Credentials $\rightarrow$ Move Again.
*   **Analogy:** Initial access is breaking in; lateral movement is using stolen keys to walk through the front door.

## The Three Pillars
*   **Remote Execution:** Abusing legitimate Windows administration protocols (SMB, WinRM, WMI, DCOM) to run commands remotely (e.g., PsExec, `winrs`).
*   **Credential Reuse:** Authenticating without a plaintext password by leveraging authentication mechanisms directly. Includes **Pass-the-Hash** (using NTLM hashes) and **Pass-the-Ticket** (using Kerberos tickets).
*   **Pivoting:** Routing or tunneling traffic through a compromised dual-homed host to access segmented/restricted networks (e.g., Server VLANs or DC subnets).

## Prerequisites for Lateral Movement
*   **Authentication Material:** Must possess a valid plaintext password, NTLM hash, or Kerberos ticket.
*   **Privileges:** The compromised account must belong to the **Local Administrators** group on the target machine (to write to `ADMIN$` or interact with the Service Control Manager).

## The Core Arsenal (Tools)
*   **Impacket:** Python suite for remote execution (`psexec.py`, `wmiexec.py`, `smbexec.py`).
*   **NetExec (nxc):** Multi-protocol Swiss army knife for AD enumeration and execution (successor to CrackMapExec).
*   **Evil-WinRM:** Provides interactive PowerShell sessions over WinRM.
*   **SSH / Chisel / Ligolo-ng:** Tools dedicated to tunneling and pivoting.
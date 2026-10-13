# Active Directory Attack Detect Lab

# Working - In Progress

<img src = "https://github.com/niha-v/Active-Directory-Attack-Detect-Lab/blob/main/ADA%20Image.jpg" width =500>

Outline

Phase 0 — Plan & isolate

Decide the domain name (lab.local) and an isolated virtual network (host-only / internal) so attack traffic never leaves the lab.
Confirm the three VMs can reach each other and that your existing Splunk is reachable from the Windows hosts.

Phase 1 — Build the lab

- Windows Server VM → promote to a domain controller (installs AD DS + DNS, creates the domain).
- Create a handful of users and groups, and — this part matters — at least one service account with a Service Principal Name (SPN). Kerberoasting has nothing to hit without one.
- Windows client VM → join it to the domain.
- Install Sysmon on both Windows hosts with a known-good config (the SwiftOnSecurity config is the standard starting point).
- Install the Splunk Universal Forwarder on both, forwarding the Security and System event logs plus the Sysmon operational channel into Splunk.
- Verify events are landing in Splunk before you attack anything.

Phase 2 — Attack from Kali
- Run each attack one at a time and keep a timestamped log of exactly what you ran and when — you’ll need those timestamps to find the events later.

Attack	Typical tool	What it does
- Password spraying	NetExec / kerbrute	One or few passwords against many accounts
- Kerberoasting	Impacket GetUserSPNs.py / Rubeus	Requests service tickets for SPN accounts to crack offline
- BloodHound	SharpHound / bloodhound.py	Enumerates the domain to map attack paths

Phase 3 — Detection engineering (the heart of it)
- For each attack: find its telemetry in Splunk, write an SPL detection, tune out the noise, save it as an alert, and tag it with the MITRE technique.

- Password spraying → look for many Event ID 4625 (failed logon) across many distinct accounts from one source in a short window. → T1110.003
- Kerberoasting → Event ID 4769 (service ticket requested) with RC4 encryption (0x17), excluding machine accounts. → T1558.003
- BloodHound → a burst of directory enumeration from one host/account (4662/LDAP volume spike). → T1087 / T1069

# Network Sharing Registry Editor

This is a small Windows utility that toggles classic file and printer sharing behavior via registry and firewall changes. It is designed to help when Windows 11 (and Windows 10) file sharing “just does not work” on a small home or lab network, especially after updates. [web:44][web:46][web:47][web:51][web:53][web:54]

It is *not* a security‑hardened solution and should only be used on trusted private networks.

---

## What problem does this fix? (Windows 11)

On many Windows 11 systems, simple LAN file sharing breaks or becomes unreliable, especially after updates or when mixing Windows 11 Pro with older devices. [web:44][web:46][web:47][web:51][web:53][web:54] Typical symptoms:

- You see other computers under “Network” but get errors when opening their shares.
- Windows 11 keeps asking for “Enter network credentials” and rejects correct passwords. [web:37][web:40][web:43][web:54]
- Old NAS boxes, media players, or older Windows versions can no longer access shares on a Windows 11 PC.
- Admin shares (like `C$`) on a Windows 11 Pro machine return “Access denied” even for local administrator accounts. [web:35][web:49][web:52][web:54]

This tool applies a set of classic registry and firewall tweaks that are known to restore connectivity in many of these scenarios by relaxing some of Windows 11’s default security decisions. [web:44][web:49][web:52][web:54]

---

## Important security warning (everyone should read)

- The options exposed by this tool **weaken** Windows security to improve compatibility and convenience.
- Enabling them can:
  - Allow empty‑password accounts to log on over the network.
  - Relax SMB signing and other protections.
  - Make local administrator accounts behave as full admins remotely.
  - Open firewall groups for file sharing and discovery. [web:44][web:49][web:52][web:54]
- Only use this tool on:
  - Private, trusted home or lab networks.
  - Machines you own and control.
- **Do not** use it on:
  - Public Wi‑Fi, guest networks, hotel/airport/café networks.
  - Corporate networks without explicit approval.

If in doubt, use the **Reset to Windows defaults** button and reboot.

---

## Quick start for non‑technical users

### Before running the tool (Windows 11 checks)

1. Make sure your PC is on a **Private** network:
   - Settings → Network & internet → click your current connection → set Network profile to **Private**. [web:44][web:46][web:53][web:54]
2. Turn on basic sharing:
   - Settings → Network & internet → Advanced sharing settings.  
   - Under “Private”, turn **on**:
     - Network discovery.
     - File and printer sharing. [web:36][web:42][web:44][web:53]
3. If file sharing still does not work, use this tool.

### Using the tool

1. Right‑click the program and choose **Run as administrator**.
2. Read the list of options and tick only what you need.
3. Click **Apply selected changes**.
4. Restart the PC (recommended).
5. Try accessing your shares again from another computer.

### “Panic button” – undo changes

If you get uncomfortable or something behaves strangely:

1. Run the tool again as Administrator.
2. Click **Reset to Windows defaults**.
3. Restart the PC.

This brings the relevant settings back toward Microsoft’s safer defaults. [web:44][web:51][web:54]

---

## How this helps Windows 11 specifically

Windows 11 tightened LAN security, especially on Pro editions and after cumulative updates. In many environments, that leads to: [web:34][web:44][web:46][web:47][web:51][web:53][web:54]

- Guest / anonymous access being blocked even if advanced sharing settings look permissive.
- SMB signing and credential rules preventing older clients from connecting.
- Local administrators being filtered when connecting remotely, breaking admin shares. [web:35][web:49][web:52][web:54]

This utility focuses on registry and firewall changes that have been widely used to:

- Re‑enable legacy SMB access where required.
- Relax SMB signature requirements.
- Allow full local admin tokens over the network via `LocalAccountTokenFilterPolicy`.
- Ensure firewall rules for **File and Printer Sharing** and **Network Discovery** are enabled. [web:35][web:44][web:49][web:52][web:54]

It is not a universal “one‑click fix”, but it often resolves the “Windows 11 network sharing not working” situation on home networks when standard Settings and Control Panel options are not enough. [web:44][web:47][web:48][web:51][web:54]

---

## Features (technical overview)

The GUI is a front‑end for the following behaviors:

- LSA:  
  - `HKLM\SYSTEM\CurrentControlSet\Control\Lsa\LimitBlankPasswordUse`
- LanmanServer:  
  - `HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters\SMB1`  
  - `HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters\SMB2`
- LanmanWorkstation:  
  - `HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\RequireSecuritySignature`  
  - `HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\EnableSecuritySignature`
- System policy:  
  - `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy`
- Windows Firewall groups (via `netsh advfirewall`):
  - “File and Printer Sharing”
  - “Network Discovery” [web:20][web:23][web:35][web:44][web:49][web:52][web:54]

All changes are made using standard registry functions and `netsh` commands; nothing else is installed. [web:20][web:23]

---

## Checkbox details

1. **Allow blank passwords for network access**  
   - Sets `LimitBlankPasswordUse = 0`.  
   - Allows accounts with empty passwords to log on over the network.  
   - Strongly discouraged outside of isolated labs and test VMs. [web:20][web:23]

2. **Enable legacy SMB1/2 server**  
   - Sets `SMB1 = 1`.  
   - Sets or enforces `SMB2 = 1`.  
   - Intended only when legacy clients/devices require SMB1 or non‑modern SMB behavior; SMB1 is deprecated and insecure. [web:38][web:41][web:44]

3. **Relax SMB security signatures**  
   - Sets `RequireSecuritySignature = 0`.  
   - Sets `EnableSecuritySignature = 0`.  
   - Disables enforcement/negotiation of SMB signing, which can restore compatibility with certain devices but removes integrity protections on SMB traffic. [web:35][web:44]

4. **Enable full local admin tokens over network**  
   - Sets `LocalAccountTokenFilterPolicy = 1`.  
   - Disables UAC remote token filtering so local administrators are treated as full admins on remote connections (fixes many admin share access issues). [web:35][web:49][web:52][web:54]

5. **Enable firewall file/printer sharing & network discovery**  
   - Runs:  
     - `netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes`  
     - `netsh advfirewall firewall set rule group="Network Discovery" new enable=Yes`  
   - Ensures the usual inbound rules required for SMB and discovery are enabled on all profiles. [web:44][web:46][web:53][web:54]

---

## Reset behavior

The **Reset to Windows defaults** button approximates a fresh, more secure configuration by:

- `LimitBlankPasswordUse = 1`
- `SMB1 = 0`, removing the `SMB2` override.
- `RequireSecuritySignature = 1`
- `EnableSecuritySignature = 1`
- Deleting `LocalAccountTokenFilterPolicy`
- Setting firewall rule groups:
  - “File and Printer Sharing” → `enable=No`
  - “Network Discovery” → `enable=No` [web:20][web:23][web:44][web:54]

A reboot is recommended after applying or resetting.

---

## Installation and usage

- **Requirements**
  - Windows 10/11 (tested primarily with Windows 11). [web:44][web:46][web:47][web:54]
  - Local administrator rights (tool must be run as admin).
  - AutoIt runtime if running `.au3`, or a compiled `.exe`. [web:16][web:17][web:20]

- **Running**
  1. Place the executable (and optional icon file) in a folder of your choice.
  2. Right‑click → **Run as administrator**.
  3. Adjust checkboxes, click **Apply selected changes**, then reboot.

---

## Who is this for?

- Home / lab users frustrated with Windows 11 network sharing not working and willing to trade security for convenience on a trusted LAN. [web:44][web:47][web:51][web:54]
- Power users who understand SMB, UAC, and firewall implications but want a quick toggle instead of manual registry edits.
- IT professionals may use it as a quick troubleshooting helper in non‑production environments, but should prefer Group Policy, security baselines, or configuration management for real deployments. [web:44][web:50][web:54]

---

## Limitations

- Does not:
  - Fix broken drivers, faulty hardware, or third‑party firewall products.
  - Replace correct NTFS/share permissions.
  - Guarantee compatibility with every NAS/router firmware. [web:36][web:44][web:46][web:54]
- Always verify:
  - Both machines are on the same IP network and **Private** profile.
  - Correct user accounts and permissions exist on the target machine.
  - Any third‑party security software is not blocking SMB. [web:44][web:46][web:53][web:54]

---

## Status bar tagline (optional)

If you want a short line in your GUI that reflects this README, you can use:

> Helps fix Windows 11 home LAN file sharing issues by toggling legacy network sharing options (unsafe on untrusted networks).


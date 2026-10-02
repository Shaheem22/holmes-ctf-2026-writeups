<img src="./assets/banner.png" style="max-width: 100%;" align=left />

# Bottle Out
2<sup>nd</sup> October 2026

Prepared by: Muhammad Shaheem (HTB: feanor)

Machine Author(s): Not specified by the event

Difficulty: Not confirmed

> **Flag values are withheld pending confirmation that this Sherlock has been retired.** This writeup covers the reasoning and tools used, not the literal strings submitted. Five of ten questions were not resolved — noted honestly below rather than omitted.

## Scenario
```
Part of Holmes CTF 2026: The Reichenbach Directive. Windows disk forensics against a workstation
(DESKTOP-QMTIG5I) belonging to a user, spur, who installed OpenVPN, a remote-management agent, and an
XMPP client, then ran a cleanup script ("Operation Vanish") to delete evidence of all three.
```

## Artifacts Provided

File hashes were not recorded during the working session. Artifacts examined:

- `DESKTOP-QMTIG5I.E01` — full disk image, analyzed via Autopsy 4.23.1
- Windows Security event log (read directly in Autopsy; exports repeatedly failed)
- Registry hives (SOFTWARE), via EZ-Tools RegistryExplorer

## Initial Analysis

The disk image was mounted and worked through Autopsy on a remote RDP analysis VM. Most of what got answered came from material that survived deletion well enough for Autopsy's orphan/deleted-file search to recover, plus registry analysis once the relevant hive was located.

Two infrastructure problems cost real time independent of the forensics itself: the RDP session to the analysis VM dropped mid-lookup, and a second session remounted the image to a different drive letter, breaking every path already in hand. A registry hive that wouldn't populate on first load needed a Shift-click past the "dirty hive" prompt to auto-replay its transaction logs — found too late to recover the remaining answers in this run.

### Analysis Sub-Section — VPN and RMM agent

The recovered `.ovpn` config and its accompanying certificate gave up the VPN server details directly. The RMM agent's version string surfaced once its installed binary was located; its backend domain was not in a flat config file under the documented install path, but in the registry instead (`HKLM\SOFTWARE\<agent>\...`) — a substring keyword search across the whole image for the agent's product name is the fastest way in if the key path isn't obvious.

### Analysis Sub-Section — Operation Vanish and Gajim

The first cleanup command read directly out of the Windows Security event log for the relevant time window. The XMPP account came from the same deleted-but-orphaned material as the rest of the user's application data; the Gajim client's profile folder itself had been deleted by the cleanup script, and only bundled UI resources (icons, status images) survived in the recovered file listing — no config, chat database, or secrets store turned up.

## Questions

1. What are the remote address and port of the VPN server that the user connected to? (`IPv4:port`)
   `REDACTED`
2. What Certificate Authority (CA) issued the VPN client certificate? (string)
   `REDACTED`
3. What is the IP address assigned to the user by the VPN server? (`IPv4 address`)
   `REDACTED`
4. It seems that the PC is remotely managed. What is the name and version of the installed remote management agent? (`Agent Name v.X.Y.Z`)
   `REDACTED`
5. What is the domain the remote management agent connects to? (fully qualified domain name)
   `REDACTED`
6. What is the remote management agent Authentication Token? (SHA-1 Hash)
   `NOT SOLVED — sits one registry value below the domain in the same key; RDP session dropped before it was read`
7. What is the first of the three commands executed in Operation Vanish? (`*********** ************ *:\*****\****\***** ******** ******`)
   `REDACTED`
8. What is the account used to connect to the instant messaging server through the software previously identified? (email address)
   `REDACTED`
9. What is the password for that account? (string)
   `NOT SOLVED — Gajim profile folder deleted by the cleanup script; no config, chat database, or secrets store recovered`
10. What is the full name of the jailer? (`FirstName LastName`)
    `NOT SOLVED — depends on the same Gajim data as Q9`

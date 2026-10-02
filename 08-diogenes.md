<img src="./assets/banner.png" style="max-width: 100%;" align=left />

# DIOGENES
2<sup>nd</sup> October 2026

Prepared by: Muhammad Shaheem (HTB: feanor), with one answer supplied by Abdullah Mahsud

Machine Author(s): Not specified by the event

Difficulty: Not confirmed

> **Flag values are withheld pending confirmation that this Sherlock has been retired.** This writeup covers the reasoning and tools used. Eleven of twelve questions were not resolved — noted honestly below rather than omitted.

## Scenario
```
Part of Holmes CTF 2026: The Reichenbach Directive. An Active Directory intrusion, framed as a
"DIOGENES" drone-recovery story folding into a domain compromise, attributed in-fiction to the
Napoleon APT: privilege escalation, lateral movement, and persistence inside a Windows domain.
```

## Artifacts Provided

File hashes were not recorded during the working session. Artifacts examined:

- An ISO/IMG container delivered as evidence, holding a shortcut disguised as an unrelated document alongside supporting executables
- A packet capture (~692 KB)
- An NTLM operational event log (~69.6 KB)

## Initial Analysis

The scenario's own PDF is narrative only — no IOCs, file names, or technical artifacts — until evidence arrives separately. That evidence came packaged inside an ISO/IMG container, a format that can mount on Windows without tripping the usual "downloaded from the internet" warning. Mounting it is already step one of the intended infection chain, worth confirming explicitly before going further rather than assuming it's inert.

**Safety note:** anything delivered this way should be handled on an isolated, snapshotted VM with its network interface disconnected at the hypervisor level, not just inside the guest OS — treat every file in the container as live until proven otherwise, and never execute the disguised payloads even to "just check."

### Analysis Sub-Section — Network traffic

Statistics on the packet capture (conversation view, sorted by byte count) isolates one host exchanging disproportionate traffic on an unusual port. Following that TCP stream surfaces a distinctive User-Agent string, a custom header, and a JSON response shape — cross-checking those three against public write-ups on known C2 frameworks' default network fingerprints is usually enough to name the framework with confidence, without needing to reverse the implant itself.

### Analysis Sub-Section — What wasn't reached

The NTLM operational log and the rest of the packet capture needed to be worked through in full to answer the remaining questions (session/encryption keys, the privilege-escalation CVE, a custom BOF's hash, an ObjectSID tied to a write-access abuse, the named password-recovery technique and its timing, the credentials it produced, the lateral-movement upload path, the service group used to start the new agent, and the resulting beacon identifiers). That work didn't happen before the session's attention moved elsewhere.

## Questions

1. What is the C2 used for this attack? (string)
   `REDACTED — see network-traffic analysis above; caveat below`
2. What is the SessionKey:EncryptionKey used? (`SessionKey:EncryptionKey`)
   `NOT SOLVED`
3. What CVE is used for privilege escalation? (`CVE-****-*****`)
   `NOT SOLVED`
4. Napoleon APT used a custom BOF to execute this attack, what is the md5sum? (md5 hash)
   `NOT SOLVED`
5. In order for this attack to happen, a compromised user needs to have write rights on a Property, what is the ObjectSID of that property? (SID)
   `NOT SOLVED`
6. In order to perform the above attack, a user's password needs to be known. What (named) attack did the APT group use, and when did it take place? (`NAME_OF_ATTACK:YYYY-MM-DD HH:mm:ss`)
   `NOT SOLVED`
7. Which user credentials were used to perform the attack? (`username:password`)
   `NOT SOLVED`
8. Which user was targeted and what was their resulting password? (`username:password`)
   `NOT SOLVED`
9. After privesc, what is the new token logon type that was created? (number)
   `REDACTED — supplied by Abdullah Mahsud from a related angle, not independently derived here`
10. For lateral movement, what is the path the new agent was uploaded to? (`\\IP\Share\...\<...>\file`)
    `NOT SOLVED`
11. Which service group was used to start the new agent? (string)
    `NOT SOLVED`
12. What is the new BeaconID:SessionKey? (`BeaconID:SessionKey`)
    `NOT SOLVED`

## Caveat on Question 1

The C2 fingerprint match above was reached in a working session where this scenario's question list and a different scenario's question list were pasted close together in time. Treat the attribution as a strong, well-evidenced lead rather than a confirmed answer until it's checked directly against this scenario's own submission.

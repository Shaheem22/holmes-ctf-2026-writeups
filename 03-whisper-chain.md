<img src="./assets/banner.png" style="max-width: 100%;" align=left />

# Whisper Chain
2<sup>nd</sup> October 2026

Prepared by: Muhammad Shaheem

Machine Author(s): Not specified by the event

Difficulty: Medium

> **Flag values are withheld pending confirmation that this Sherlock has been retired.** This writeup covers the reasoning and tools used. Four of eight questions were not resolved — noted honestly below.

## Scenario
```
Part of Holmes CTF 2026: The Reichenbach Directive. Live network exploitation against an XMPP server
believed to be used by an APT group's communications infrastructure — the one scenario in the set
that is live network enumeration rather than static disk/log forensics.
```

## Artifacts Provided

No static artifact files — this scenario was a live network target. Server hostname and lab IP are withheld; the hostname is itself part of Question 1's answer.

## Initial Analysis

TLS certificate enumeration against the server's handshake surfaced its Subject Alternative Names up front, each hinting at a function before anything else was touched.

### Analysis Sub-Section — Gaining an account

In-band registration via [slixmpp](https://slixmpp.readthedocs.io/) (the `xep_0077` plugin), with a custom SSL context to accept the server's self-signed certificate, produced a working account. From there, public MUC room discovery is a standard XMPP disco query.

### Analysis Sub-Section — Credential recovery and hidden rooms

A document uploaded to one of the public rooms carried credentials for an APT member — not in its rendered content, but in its file metadata. `exiftool` against the document surfaced them without needing to read a single page. Always check document metadata before diving into message content; it's often faster.

Those credentials unlock two rooms that aren't publicly listed. Their transcripts name the C2 framework in use and an operational alias used by one member, directly in plain chat content.

### Analysis Sub-Section — The locked layer

The remaining questions sit behind XMPP's MAM (message archive) layer on the private rooms. Two obstacles stacked here:

`slixmpp`'s own MAM query implementation returned empty results against this server — a silent failure, no exception raised, which cost real time before the cause was clear. A raw-socket script, reading the MAM protocol directly rather than going through the library, retrieved the archived messages where the library failed.

The retrieved content was itself encrypted. A passphrase recovered from elsewhere in the same archive looked like a plausible key by every surface signal but didn't decrypt anything — worth treating as a deliberate in-fiction decoy rather than assuming another library failure. The real key wasn't found in the time available.

## Questions

1. List all Subject Alternative Names present in the server's TLS certificate. Report the primary hostname first, followed by the remaining SANs in alphabetical order. (`fqdn,fqdn,fqdn,fqdn`)
   `REDACTED`
2. List the names of all public channels available on the XMPP server, sorted alphabetically. (`string,string,string,string`)
   `REDACTED`
3. Investigate the conversations to find the credentials of one of the APT members. (`email address:password`)
   `REDACTED — see document metadata, not chat content`
4. What is the URL of the social media account used by one of the APT members? (`*****://************.***/********`)
   `NOT SOLVED — behind the encrypted MAM archive`
5. What is the XMR wallet used in the first operation? (string)
   `NOT SOLVED — behind the encrypted MAM archive`
6. What is the nickname of the APT affiliate who kidnapped Watson? (string)
   `REDACTED`
7. Where has Watson been kidnapped? (string)
   `NOT SOLVED — behind the encrypted MAM archive`
8. What is the name of the latest operation that the APT is planning, and what is its objective? (`Operation name,Objective`)
   `NOT SOLVED — behind the encrypted MAM archive`

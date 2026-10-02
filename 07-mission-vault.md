<img src="./assets/banner.png" style="max-width: 100%;" align=left />

# Mission Vault (scoreboard name: Iron Feather)
2<sup>nd</sup> October 2026

Prepared by: Muhammad Shaheem (HTB: feanor)

Machine Author(s): Not specified by the event

Difficulty: <font color="red">Hard</font>

> **Flag values are withheld pending confirmation that this Sherlock has been retired.** This writeup covers the reasoning and tools used to reach each answer, not the literal strings submitted. This was the cleanest run of the event — every question resolved in order.

## Scenario
```
Part of Holmes CTF 2026: The Reichenbach Directive. A stripped x86-64 ELF binary from a PX4-based
flight controller is reverse engineered to recover an encrypted mission datastore and reconstruct a
drone hijacking — an injected command that diverted payload release and forced the aircraft down.
```

## Artifacts Provided

File hashes were not recorded during the working session. Artifacts examined:

- `px4` — stripped x86-64 ELF binary (flight-controller firmware)
- An encrypted mission datastore image
- Drone flight telemetry log

## Initial Analysis

The encrypted datastore opens with an ASCII magic string unique to this challenge, readable straight off a hex dump of the first bytes, and its authentication-tag layout identifies a standard AEAD construction protecting it.

### Analysis Sub-Section — Key derivation

Disassembling the binary (Ghidra/IDA both work) turns up a function that runs a custom 32-bit mixing loop before feeding its output into a well-known password-based KDF — count the loop's iterations directly from the disassembly, since that count matters for reproducing the key. This is a common homebrew-crypto pattern: borrow a trustworthy standard primitive, wrap it in a layer that reads as extra security but mostly adds obscurity. Reimplementing the derivation in Python against the supplied encrypted image produces the AES key needed to decrypt the datastore.

### Analysis Sub-Section — Mission data and telemetry correlation

PX4's mission file format defines two banks; only one is marked active. Parsing the active bank's mission-item list finds which numbered item is flagged as the payload-release trigger, and how many items the bank holds in total.

Cross-referencing the mission data against the drone's own telemetry log reconstructs the flight: the takeoff coordinates, the originally planned landing point, and the coordinates where payload release actually fired (which don't match the planned mission) are all readable directly from telemetry once correlated against the mission item list by timestamp. The MAVLink command responsible for bringing the drone down was injected into the live command stream rather than present in the original scheduled mission — diffing the live stream against the mission plan makes the injection point stand out immediately. Distance between the two relevant telemetry points gives the metres figure; the crash coordinates and disarm timestamp read directly off the tail of the telemetry log; a reverse-geocode of the crash coordinates gives the nearest named road.

## Questions

1. What eight-byte ASCII magic identifies the encrypted datastore format? (string)
   `REDACTED`
2. Which authenticated encryption algorithm protects the datastore? (string)
   `REDACTED`
3. What is the Relative Virtual Address (RVA) of the custom key-derivation function? (hex number)
   `REDACTED`
4. How many rounds are performed by the custom 32-bit mixing loop? (number)
   `REDACTED`
5. Which standard KDF produces the AES key? (string)
   `REDACTED`
6. What AES-256 key is derived for the supplied encrypted image? (64 lowercase hexadecimal characters)
   `REDACTED`
7. Which of PX4's two mission banks is active? (number)
   `REDACTED`
8. How many mission items are stored in the active bank? (number)
   `REDACTED`
9. Which mission item triggers payload release? (number)
   `REDACTED`
10. Where was the drone supposed to land? (latitude,longitude; 5 decimal places)
    `REDACTED`
11. Where did the drone take off? (latitude,longitude; 7 decimal places)
    `REDACTED`
12. Where was the drone when payload release was triggered? (latitude,longitude; 7 decimal places)
    `REDACTED`
13. Which MAVLink command was injected to bring down the drone? (string)
    `REDACTED`
14. How far did the drone travel between payload release and command injection? (horizontal metres, nearest whole number)
    `REDACTED`
15. Where did the drone crash? (latitude,longitude; 7 decimal places)
    `REDACTED`
16. When was the drone disarmed after the crash? (seconds after boot; 3 decimal places)
    `REDACTED`
17. Which road is closest to the crash site? (string)
    `REDACTED`

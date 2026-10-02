<img src="./assets/banner.png" style="max-width: 100%;" align=left />

# Silent Dividend
2<sup>nd</sup> October 2026

Prepared by: Muhammad Shaheem

Machine Author(s): Not specified by the event

Difficulty: Medium

> **Flag values are withheld pending confirmation that this Sherlock has been retired.** This writeup covers the reasoning and tools used to reach each answer, not the literal strings submitted.

## Scenario
```
Part of Holmes CTF 2026: The Reichenbach Directive. An Electron desktop application, TrustSettle.exe,
is investigated as a cryptocurrency wallet-drainer dropper, chaining an on-chain smart contract
"dead drop," an obfuscated Lua payload, and a fake Terms-of-Service page into a token-approval scam.
```

## Artifacts Provided

File hashes were not recorded during the working session. Artifacts examined:

- `TrustSettle.exe` — Electron application (dropper)
- `api.txt` — obfuscated Lua payload, run via a bundled `luajit.exe`
- `settlement.html` — the fake Terms-of-Service / wallet-drainer page
- Two Sepolia-deployed smart contracts (addresses withheld with the flag values)

## Initial Analysis

`TrustSettle.exe` copies its bundled `extraResources` into a world-writable system folder on launch — the kind of location any account can read and write to without a UAC prompt, worth flagging as a weak design choice on its own before going further.

Static analysis of the Electron app's JavaScript (`preload.js` and related source) was enough to map the overall chain without needing to run the binary: a call to a smart contract function retrieves a decryption key, that key decrypts an embedded command, and the command opens an HTML file from a path built off a standard Electron packaging environment variable.

### Analysis Sub-Section — Lua payload deobfuscation

`api.txt` needed deobfuscation before any of it was readable. [Prometheus-Deobfuscator](https://github.com/) got partway there but repeatedly failed on Python f-strings containing backslashes (`SyntaxError: f-string expression part cannot include a backslash`) — Python doesn't allow a backslash inside an f-string's expression portion. The fix: pull the offending `.join()` call out into its own variable before the f-string, so the f-string itself only references a plain variable name. Repeat for each occurrence (a `grep` for the join pattern finds them all at once rather than one crash at a time).

Once it ran cleanly, the deobfuscated output's FFI `cdef` declarations named both a Win32 structure and a Win32 API by their real identifiers, directly — no further reversing needed for those two.

### Analysis Sub-Section — Wallet drainer

`settlement.html` stages as a routine Terms of Service acceptance page. Connecting a wallet triggers a token approval call for the maximum value a `uint256` can hold, via the standard ethers.js v6 class for talking to an injected browser wallet (`window.ethereum`-style providers).

## Questions

1. To which directory does the application copy the files bundled within the `extraResources` folder? (`*:\path\to\dir`)
   `REDACTED — check path.join() calls in preload.js against well-known Windows world-writable folders`
2. Which Win32 structure defines the format of the buffer returned by the Lua script when monitoring directory changes? (string)
   `REDACTED — look up what ReadDirectoryChangesW fills on a successful read`
3. Which Win32 API is used by the Lua script to send an HTTP request to the remote server? (string)
   `REDACTED — trace the WinHttpOpen → WinHttpConnect → WinHttpOpenRequest → ... chain for the call that actually transmits`
4. Which smart contract function does the Electron application invoke to retrieve the decryption key for the encrypted payload? (`function()`)
   `REDACTED — read the CONTRACT_ABI array declared in the Electron app's source`
5. Investigate the smart contract using its address, analyze its logic, and recover the flag by decoding the encrypted data. (`****=******** **********_*********=**-****`)
   `REDACTED — decrypt the on-chain payload with the key from Q4; the Holmes/Moriarty theming running through this event is a useful tell once decrypted`
6. Which environment variable corresponds to the directory where the application copies the HTML file from its package? (string)
   `REDACTED — a standard Electron packaging variable, visible in a path.resolve() call referencing the HTML file`
7. What token function does the HTML page call to request spending permission? (`function()`)
   `REDACTED — search settlement.html's JavaScript for the approval call`
8. What is the exact token amount passed to the approval call? (number)
   `REDACTED — the maximum value a uint256 can hold, dressed up as routine`
9. What ethers.js v6 provider class is used to connect to the browser wallet? (string)
   `REDACTED — ethers.js v6's standard class for an injected window.ethereum-style provider`
10. Analyze the HTML page to uncover a smart contract reference. Investigate the contract's logic and determine how to interact with it to recover the hidden flag. (`**.****,*.****`)
    `NOT CONFIRMED — see note below`

## Note on Question 10

A second, independent contract referenced inside `settlement.html` holds this flag. Reading its bytecode by hand turned up a public getter returning a hidden "owner" address, and a second function that only decrypts and returns its stored string when the address passed in matches whatever that getter returns — no private key needed, just call the getter first and feed its result back in.

That final call needed a live RPC endpoint the analysis sandbox couldn't reach. A short ethers.js script (two contract calls) was handed off to run on a machine with real network access; the printed result was never confirmed back, so this answer isn't recorded even in redacted form — it's genuinely unsolved.

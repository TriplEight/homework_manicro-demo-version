# ⚠️ Security Analysis Report — "Fake Job Demo" Manicro Demo Repository

**This is an active malware. Do not run this code.**

## CRITICAL SECURITY ALERT - EMBEDDED BACKDOOR DETECTED

**Source**: Linkedin job offer

**Sender** https://www.linkedin.com/in/andrii-shan-0335b2368/

**Sender's notion** https://www.notion.so/Manicro-V-Demo-Review-2ee561957fc680798cdaf8479bf3bccb

**Original repository**: https://bitbucket.org/tre555/manicro-demo-version/src/main/

**Analysis Date**: 2026-03-01

**Status**: ⚠️ HIGH-SEVERITY EXPLOIT — Do not run npm install or any project commands

---

## Malware Analysis: `tailwind.config.js`

### 2.1 Placement

The legitimate Tailwind configuration ends at line 53 with `};`. What follows on the **same line**, pushed ~2,000 characters to the right with horizontal whitespace, is approximately 100 lines of densely obfuscated JavaScript. It is invisible without horizontal scrolling, a character count, or a diff.

```
line 53: };                             [2000 spaces]    const as=a1, at=a1 ...
```

This placement is deliberate: `tailwind.config.js` is processed by Node.js as a CommonJS module (`require()`d by webpack / react-scripts). The obfuscated code executes as part of that module load — before any component renders, before any test runs.

### 2.2 Obfuscation Layers (decoded)

The code uses four layered techniques to avoid plain-text detection:

#### Layer 1 — Anti-tamper string table shuffle

The function `a0()` returns an array `bd` of ~90 scrambled string fragments. An immediately-invoked function (IIFE) runs `push/shift` on that array in a `while(true)` loop until the following arithmetic checksum evaluates to `0x92692`:

```
parseInt(a1(0x5e))/1 + parseInt(a1(0x12))/2 − ...
```

This means the array is in a different order at runtime than in source. Simple inspection of the raw array produces wrong results; only running the IIFE establishes the correct order.

#### Layer 2 — Indexed string lookup

`a1(n)` returns `bd[n]` (after shuffle). Strings that would trigger security scanners (`"fs"`, `"require"`, `"exec"`) are split across multiple `a1()` calls and concatenated at runtime.

#### Layer 3 — Base64 decoding of module names

The `c()` function strips the first character of a string and base64-decodes the rest:

```js
const c = a2 => Buffer.from(a2.slice(1), 'base64').toString('utf8')
```

Decoded results (reconstructed from the string table):

| Encoded call | Decoded value |
|---|---|
| `c(as(0x20))` | `"fs"` |
| `c(as(0x69))` | `"os"` |
| `c(av(0x64)+au(0x19)+au(0x67))` | `"https"` |
| `c(at(0x13)+aw(0x36))` | `"path"` |
| `c(at(0x5a)+au(0x1a)+av(0x5)+at(0x39)+'z')` | `"child_process"` |

Each of these is passed to `require()`, giving the malware full access to the filesystem, network, and shell — with no imports visible in plain text.

#### Layer 4 — XOR-encoded byte arrays

Sensitive strings (paths, commands) are stored as integer arrays and decoded at runtime by XORing each byte with a 4-byte key `[0x70, 0xa0, 0x89, 0x48]` (cycling):

```js
const w = [0x70, 0xa0, 0x89, 0x48];
const g = arr => arr.map((b, i) => String.fromCharCode(0xff & (b ^ w[i & 3]))).join('');
```

Decoded strings (all computed statically, no execution):

| Array | Bytes (hex) | Decoded string |
|---|---|---|
| `p` | `5e d6 fa 2b 1f c4 ec` | `.vscode` |
| `z` | `04 c5 fa 3c 5e ca fa` | `test.js` |
| `R` | `00 c1 ea 23 11 c7 ec 66 1a d3 e6 26` | `package.json` |
| `V` | `1e cf ed 2d 2f cd e6 2c 05 cc ec 3b` | `node_modules` |
| `Y` | `13 c4` | `cd` |
| `T` | `56 86 a9 26 00 cd a9 21 50 8d a4 3b 19 cc ec 26 04` | `&& npm i --silent` |
| `A` | `1e cf ed 2d` | `node` |
| `F` + `H` | (combined) | `node --prefix` … `install` |
| `W` | `5f ca a6` | `/j/` |
| `J` | `5f d0` | `/p` |

#### Layer 5 — Hardcoded C2 IPs in base64 (literal strings in source)

Two strings appear literally in the source (no further obfuscation needed since base64 is not alarming to a casual reader):

| Literal | Base64-decoded value |
|---|---|
| `"NDcuMTU3MzguOTIu===="` | `47.157.38.92` |
| `"NC4yMDIuMTQ3LjEyMjI1"` | `4.202.147.122` |

These are the **bootstrap C2 server IPs**. Port is assembled from the string table fragment `':124'` + `'4'` = `:1244`.

---

### 2.3 Full Attack Chain

The attack executes in nine steps, triggered automatically on any `npm start` or `npm run build`:

```
Developer runs: npm start
       │
       ▼
webpack loads tailwind.config.js
       │
       ▼
Step 1 — Bootstrap C2 contact (C(0))
    HTTPS GET → 47.157.38.92:1244/<path>
    If unreachable → retry with 4.202.147.122 (C(1))
    Response: dynamic exfiltration server IP (b) + port (v)
       │
       ▼
Step 2 — Victim fingerprinting (I())
    Collects: os.hostname(), os.platform(), os.userInfo(),
              network interfaces, external IP
    On macOS: also appends username to hostname string
       │
       ▼
Step 3 — Beacon exfiltration (L())
    HTTPS POST → {b}:{v}/<exfil-path>
    Payload: { ts: timestamp, ip: victim_ip,
               hn: hostname, ss: session_state, cc: country_code }
       │
       ▼
Step 4 — Payload download (q())
    mkdir ~/.vscode  (mkdirSync, recursive, silent)
    HTTPS GET → {b}/j/<id>
    Write response → ~/.vscode/test.js
       │
       ▼
Step 5 — Package manifest download (U())
    HTTPS GET → {b}/p
    Compare downloaded package.json size to existing ~/.vscode/package.json
    If downloaded version is larger → overwrite  (update mechanism)
       │
       ▼
Step 6 — Silent package install (k())
    exec: cd "~/.vscode" && npm i --silent
    (installs malicious packages without any console output)
       │
       ▼
Step 7 — Ensure modules present (x())
    If node_modules exists → proceed
    Else: node --prefix "~/.vscode" install → proceed
       │
       ▼
Step 8 — Payload execution (S())
    exec: node ~/.vscode/test.js
    ← This runs Stage 2 payload with full Node.js privileges
       │
       ▼
Step 9 — Retry timer
    setInterval(_, 616000ms)  ← every ~10.27 minutes
    Up to 3 retries if C2 is unreachable on first run
```

After Step 8, the **Stage 2 payload** (`test.js`) is unknown without capturing live network traffic — it is fetched fresh from the C2 at runtime and is not stored in the repository. Based on the infrastructure (dynamic C2, staged delivery, silent npm install), this is consistent with a full remote access implant or credential harvester.

---

### 2.4 Persistence and Concealment

| Technique | Detail |
|---|---|
| **Hidden directory** | `~/.vscode/` is VS Code's standard config dir; not cleaned by `npm clean`, not in `.gitignore`, not suspicious to most developers |
| **Silent npm install** | `npm i --silent` produces zero console output; the developer sees nothing |
| **Triggered by legitimate tooling** | The payload runs inside `react-scripts`, not as a standalone script; process list shows `node react-scripts start` |
| **Self-updating** | Package manifest is re-downloaded and updated if the C2 serves a newer version (larger file size check) |
| **10-minute retry** | If the C2 is temporarily offline, retries 3× over ~30 minutes |
| **Fallback C2** | Two distinct IP addresses provide redundancy |

---

### 2.5 Social Engineering Design

The README is not incidental — it is a core part of the attack:

| README element | Social engineering function |
|---|---|
| "Initial foundation… starting point of our next major development phase" | Creates legitimacy; implies ongoing, real company |
| "European engineers and developers to lead key areas" | Flatters and targets European candidates specifically |
| "Prerequisites: Visual Studio Code" | Ensures `~/.vscode/` exists on the victim's machine before attack |
| "npm install → npm start" | Gives explicit permission to run; victim follows official docs |
| "A follow-up CTO session will be arranged" | Creates urgency; victim completes the task quickly |
| "document any issues… will guide the technical call" | Frames running the code as a job requirement |
| Repository name: `homework_manicro-demo-version` | "Homework" normalises the request; it's a standard code test |

The attack was almost certainly delivered via LinkedIn: a recruiter or "CTO" shares this repository as a take-home code review task. The victim is told to clone, run, and provide feedback before a scheduled call. They run `npm start`, which triggers the entire chain in seconds, invisible to them.

---

### 2.6 Indicators of Compromise

**Files to check on any machine where this project was run:**

- `~/.vscode/test.js` (Stage 2 payload)
- `~/.vscode/package.json` (malicious dependency manifest)
- `~/.vscode/node_modules/` (installed malicious packages)

**Network indicators:**

- C2 IP 1: `47.157.38.92` port `1244` (bootstrap)
- C2 IP 2: `4.202.147.122` (fallback bootstrap)
- Dynamic exfiltration server: unknown without live traffic capture

**Process indicators:**

- `node ~/.vscode/test.js` appearing in process list
- `npm install` running in `~/.vscode/` directory

---

### 2.7 Summary Severity

| Item | Severity |
|---|---|
| Staged malware dropper executing on `npm start` | **Critical** |
| Victim fingerprinting and C2 exfiltration | **Critical** |
| Persistent implant installed in `~/.vscode/` | **Critical** |
| Social engineering via fake job interview | **Critical** |
| Hidden payload invisible in editor without horizontal scroll | **High** |
| Two-IP redundant C2 with retry logic | **High** |
| Self-updating package manifest mechanism | **High** |

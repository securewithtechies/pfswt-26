# The Unwritten Legacy (750 Points)

**Category:** Cryptography  
**Points:** 750  
**Difficulty:** Expert  
**Theme:** Matangini Hazra - "Gandhi Buri"

---

## Challenge Overview

This challenge combines **JPEG/ZIP Polyglot Files** with the **VIC Cipher** - the most complex cipher used during the Cold War.

---

## Prerequisites

You need keys derived from all previous challenges:
- Challenge 100: `GANDHIBURI` → VIC Keyword
- Challenge 250: `731942` → Agent Key
- Challenge 500: `VANDEMATARAM` → Phrase

---

## Solution

### Step 1: Identify the Polyglot

A polyglot file is valid in multiple file formats simultaneously.

**JPEG/ZIP Polyglot Structure:**
- JPEG: Starts with `FF D8`, ends with `FF D9`
- ZIP: Starts with `PK` (50 4B), central directory at end

**Verification:**
```bash
# Check file type
file gandhi_ka_sipahi.jpg
# Output: JPEG image data...

# Scan for embedded data
binwalk gandhi_ka_sipahi.jpg
# Output: Shows embedded ZIP at offset

# Look for ZIP header
xxd gandhi_ka_sipahi.jpg | grep "504b"
```

### Step 2: Extract the ZIP Contents

```bash
# Method 1: Direct unzip (works because ZIP reads from end)
unzip gandhi_ka_sipahi.jpg -d extracted/

# Method 2: Copy and rename
cp gandhi_ka_sipahi.jpg secret.zip
unzip secret.zip -d extracted/

# Method 3: Using 7z
7z x gandhi_ka_sipahi.jpg -oextracted/
```

**Extracted Structure:**
```
extracted/
├── history/
│   └── matangini_hazra.txt  (decoy - historical info)
└── secret/
    ├── cipher.txt           (VIC-encrypted message)
    └── hint.txt             (cipher identification)
```

### Step 3: Analyze the Cipher

Open `secret/cipher.txt` to find a series of numbers:
```
73 42 91 55 28 ...
```

The hint mentions "Soviet spies during the Cold War" - this is the **VIC Cipher**.

### Step 4: Derive VIC Cipher Keys

The VIC cipher requires multiple key components:

| Key Component | Source | Value |
|---------------|--------|-------|
| Keyword | Challenge 100 | `GANDHIBURI` |
| Agent Key | Challenge 250 | `731942` |
| Phrase | Challenge 500 | `VANDEMATARAM` |
| Date Key | Martyrdom date | `29091942` (Sept 29, 1942) |

### Step 5: Understanding VIC Cipher

The VIC cipher is a complex multi-step process:

1. **Keyword** - Creates first substitution tableau
2. **Phrase** - Creates second substitution tableau
3. **Agent Key** - Personal identifier for key derivation
4. **Date Key** - Message-specific element
5. **Straddling Checkerboard** - Main encryption/decryption mechanism

### Step 6: VIC Cipher Decryption

**Key Derivation Process:**

```python
# Step A: Create Checkerboard from Keyword
keyword = "GANDHIBURI"
# Position: 0 1 2 3 4 5 6 7 8 9
# Letter:   G A N D H I B U R +

# Step B: Chain Addition with Agent Key
agent_key = "731942"
# Perform chain addition to extend the key

# Step C: Build Straddling Checkerboard
# The checkerboard maps letters to 1 or 2 digit numbers
```

**Decryption Steps:**

```python
def vic_decrypt(ciphertext, keyword, phrase, agent_key, date_key):
    # 1. Derive internal keys through chain addition
    internal_keys = derive_keys(agent_key, date_key)
    
    # 2. Build straddling checkerboard from keyword
    checkerboard = build_checkerboard(keyword, internal_keys)
    
    # 3. Reverse the numeric encoding
    plaintext = ""
    i = 0
    while i < len(ciphertext):
        digit = ciphertext[i]
        if digit in checkerboard['single']:
            plaintext += checkerboard['single'][digit]
            i += 1
        else:
            # Two-digit lookup
            two_digit = ciphertext[i:i+2]
            plaintext += checkerboard['double'][two_digit]
            i += 2
    
    return plaintext
```

### Step 7: Using Python Tool

```python
import sys
sys.path.append('../../tools')
from vic_cipher import VICCipher

cipher = VICCipher(
    keyword="GANDHIBURI",
    phrase="VANDEMATARAM",
    agent_key="731942",
    date_key="29091942"
)

ciphertext = "73 42 91 55 28..."  # From cipher.txt
plaintext = cipher.decrypt(ciphertext)

print(f"Decrypted: {plaintext}")
```

---

## Flag

```
pfswt{V1C_C1ph3r_Gr4ndm0th3r_0f_4ll_C1ph3rs!}
```

---

## Flag Breakdown

| Part | Meaning |
|------|---------|
| `V1C` | VIC cipher |
| `C1ph3r` | Cipher |
| `Gr4ndm0th3r_0f_4ll` | One of the most complex classical ciphers |
| `C1ph3rs!` | Completing the tribute |

---

## Historical Context

### The VIC Cipher
The VIC cipher was used by Soviet spy Reino Häyhänen (codename VICTOR) in the 1950s. It's considered one of the most complex pencil-and-paper ciphers ever devised, remaining unbroken until Häyhänen defected to the US in 1957 and revealed its workings.

### Polyglot Files
Polyglot files exploit the fact that different file formats read from different parts of the file. JPEG parsers read from the beginning, while ZIP parsers read from the end (where the central directory is located). This allows a single file to be valid in both formats.

---

## Tools Used

- **binwalk** - File structure analysis
- **7z/unzip** - ZIP extraction
- **Python** - VIC cipher implementation
- **xxd** - Hex analysis

# The 30 Designs of Destiny (500 Points)

**Category:** OSINT  
**Points:** 500  
**Difficulty:** Hard  
**Theme:** Pingali Venkayya - Designer of the Indian Flag

---

## Challenge Overview

Decode a fragmented archive containing historical publication data using multiple cipher techniques.

---

## Solution

### Step 1: Analyze the Archive Structure

The `archived_publication_1916.txt` file contains multiple encoded sections:

| Section | Encoding | Purpose |
|---------|----------|---------|
| Metadata Fragment | Plaintext | Publication context |
| SECTION_A | Morse Code | Author identity + Bifid key |
| SECTION_B | Caesar Cipher | Location data |
| SECTION_C | Base64 | Memorial reference |
| FLAG RECOVERY | Bifid Cipher | Final flag component |

---

### Step 2: Decode SECTION_A (Morse Code)

**Encoded:**
```
.--. .. -. --. .- .-.. .. / ...- . -. -.- .- -.-- -.-- .-
```

**Decoding:**

| Morse | Letter |
|-------|--------|
| `.--. ` | P |
| `..` | I |
| `-.` | N |
| `--.` | G |
| `.-` | A |
| `.-..` | L |
| `..` | I |
| `/` | (space) |
| `...-` | V |
| `.` | E |
| `-.` | N |
| `-.-` | K |
| `.-` | A |
| `-.--` | Y |
| `-.--` | Y |
| `.-` | A |

**Result:** `PINGALI VENKAYYA`

**Key Discovery:** The surname `VENKAYYA` is the Bifid cipher key for the final step!

---

### Step 3: Decode SECTION_B (Caesar Cipher)

**Encoded:**
```
Elyv Mfyo Czlo, Sjopclmlo
```

**Brute Force Approach:**
Test all 25 shift values. Shift 15 produces readable text.

```python
def caesar_decrypt(text, shift):
    result = ""
    for char in text:
        if char.isalpha():
            base = ord('A') if char.isupper() else ord('a')
            result += chr((ord(char) - base + shift) % 26 + base)
        else:
            result += char
    return result

# Try shift 15
print(caesar_decrypt("Elyv Mfyo Czlo, Sjopclmlo", 15))
```

**Result:** `Tank Bund Road, Hyderabad`

This is the location of Pingali Venkayya's statue.

---

### Step 4: Decode SECTION_C (Base64)

**Encoded:**
```
QWxsIEluZGlhIFJhZGlvIFBpbmdhbGkgVmVua2F5eWEgU3RhdGlvbg==
```

```python
import base64
result = base64.b64decode("QWxsIEluZGlhIFJhZGlvIFBpbmdhbGkgVmVua2F5eWEgU3RhdGlvbg==").decode()
print(result)
```

**Result:** `All India Radio Pingali Venkayya Station`

This is the radio station named in honor of Pingali Venkayya.

---

### Step 5: Decode FLAG RECOVERY (Bifid Cipher)

**Encrypted Text:** (from the file)
**Key:** `VENKAYYA` (from SECTION_A)

The Bifid cipher uses a 5×5 Polybius square:

```python
def create_polybius_square(key):
    # Remove duplicates and 'J'
    key = key.upper().replace('J', 'I')
    alphabet = "ABCDEFGHIKLMNOPQRSTUVWXYZ"
    
    # Build keyed alphabet
    seen = set()
    keyed = []
    for c in key + alphabet:
        if c not in seen:
            seen.add(c)
            keyed.append(c)
    
    # Create 5x5 square
    square = [keyed[i:i+5] for i in range(0, 25, 5)]
    return square

def bifid_decrypt(ciphertext, key):
    square = create_polybius_square(key)
    
    # Create lookup dictionaries
    char_to_pos = {}
    pos_to_char = {}
    for i in range(5):
        for j in range(5):
            char_to_pos[square[i][j]] = (i, j)
            pos_to_char[(i, j)] = square[i][j]
    
    # Convert ciphertext to coordinates
    ciphertext = ciphertext.upper().replace('J', 'I')
    rows = []
    cols = []
    for c in ciphertext:
        if c in char_to_pos:
            r, c_pos = char_to_pos[c]
            rows.append(r)
            cols.append(c_pos)
    
    # Combine rows and cols
    combined = rows + cols
    
    # Split into pairs and convert back
    plaintext = ""
    for i in range(0, len(combined), 2):
        if i + 1 < len(combined):
            plaintext += pos_to_char[(combined[i], combined[i+1])]
    
    return plaintext

# Decrypt with key VENKAYYA
ciphertext = "..."  # From the challenge file
plaintext = bifid_decrypt(ciphertext, "VENKAYYA")
print(plaintext)
```

---

## Flag

```
pfswt{30_Designs_Machilipatnam_1916}
```

---

## Flag Breakdown

| Part | Meaning |
|------|---------|
| `30_Designs` | Venkayya created over 30 flag designs |
| `Machilipatnam` | His publication location (1916) |
| `1916` | Year of his famous booklet on flag designs |

---

## Historical Context

In 1916, Pingali Venkayya published a booklet from Machilipatnam containing over 30 different flag designs for India. This work caught the attention of national leaders and eventually led to his design being adopted (with modifications) as the Indian National Flag.

---

## Tools Used

- Morse code decoder
- Caesar cipher brute-force
- Base64 decoder
- Bifid cipher implementation
- Python scripting

---

## Learning Points

1. Multi-stage challenges require solving each layer in sequence
2. One section may provide keys for another (SECTION_A → Bifid key)
3. Historical OSINT validates cryptographic findings
4. Bifid cipher is more complex than simple substitution ciphers

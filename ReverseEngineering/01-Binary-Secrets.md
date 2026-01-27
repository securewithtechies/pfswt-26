# Binary Secrets (100 Points)

**Category:** Reverse Engineering  
**Points:** 100  
**Difficulty:** Hard

---

## Challenge Overview

Analyze an ELF binary that uses multi-layer XOR encryption to hide the flag.

---

## Challenge Details

**Files Provided:**
- `binary_secrets` - ELF 64-bit executable

**Flag:** `pfswt{x0r_l4y3rs_4r3_n0t_3n0ugh_SWT}`  
**Password:** `SWT_R3v3rs3_2026`

---

## Solution

### Step 1: Initial Analysis

```bash
# Check file type
file binary_secrets
# Output: ELF 64-bit LSB executable, x86-64

# Search for strings
strings binary_secrets | grep -i flag
# Find markers: ENC_PWD, FLAG_1, FLAG_2, _REAL_, KEYS
```

### Step 2: Identify Encryption

Open in a disassembler (Ghidra or IDA):
- Locate the data section with encrypted content
- Find XOR keys in the `KEYS` section
- Notice keys are XORed with 0xFF for obfuscation

### Step 3: Extract XOR Keys

```python
# Keys found in binary (XORed with 0xFF)
key_data = [0xC3, 0x5A, 0xA5]

# Recover actual keys
XOR_KEY3 = key_data[0] ^ 0xFF  # = 0x3C
XOR_KEY2 = key_data[1] ^ 0xFF  # = 0xA5
XOR_KEY1 = key_data[2] ^ 0xFF  # = 0x5A
```

### Step 4: Decrypt the Real Flag

The encryption uses three XOR layers applied in sequence:

```python
def multi_layer_decrypt(data):
    # Reverse order: KEY3 -> KEY2 -> KEY1
    layer1 = bytes([b ^ 0x3C for b in data])
    layer2 = bytes([b ^ 0xA5 for b in layer1])
    layer3 = bytes([b ^ 0x5A for b in layer2])
    return layer3

def extract_after_marker(binary_data, marker):
    idx = binary_data.find(marker)
    if idx == -1:
        return None
    # Skip marker and any separator
    start = idx + len(marker)
    # Find end (null byte or next marker)
    end = binary_data.find(b'\x00', start)
    return binary_data[start:end]

# Read binary
with open('binary_secrets', 'rb') as f:
    binary = f.read()

# Extract encrypted flag from _REAL_ section
encrypted_flag = extract_after_marker(binary, b'_REAL_:')
flag = multi_layer_decrypt(encrypted_flag)
print(flag.decode())
# Output: pfswt{x0r_l4y3rs_4r3_n0t_3n0ugh_SWT}
```

### Step 5: Avoid Decoys

**Important:** The binary contains decoy flags!
- `FLAG_1` and `FLAG_2` are fake flags
- The real flag is under the `_REAL_` marker
- Don't waste time on obvious strings

---

## Key Insights

1. **Three XOR keys applied in sequence** - Layer encryption
2. **Keys stored XORed with 0xFF** - Simple obfuscation
3. **Decoy flags present** - Designed to waste time
4. **Correct marker is `_REAL_`** - Look for non-obvious markers

---

## Tools Used

- **Ghidra / IDA Pro** - Disassembly and static analysis
- **Python** - Decryption scripting
- **strings, xxd, hexdump** - Binary inspection
- **GDB** - Dynamic analysis (optional)

---

## Learning Points

1. XOR encryption is symmetric - same operation encrypts and decrypts
2. Multiple XOR layers don't significantly increase security
3. Always look for decoy data in CTF challenges
4. Marker strings often indicate important data sections

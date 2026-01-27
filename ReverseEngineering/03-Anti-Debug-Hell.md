# Anti-Debug Hell (500 Points)

**Category:** Reverse Engineering  
**Points:** 500  
**Difficulty:** Expert

---

## Challenge Overview

Bypass multiple anti-debugging protections and derive the correct decryption key from security check results.

---

## Challenge Details

**Files Provided:**
- `antidebug_hell` - ELF binary with anti-debugging protections
- `hell.key` - Encrypted key file containing the flag

**Flag:** `pfswt{4nt1_d3bug_byp4ss3d_l1k3_4_pr0_SWT}`

---

## Protection Mechanisms

| Protection | Description |
|------------|-------------|
| ptrace detection | Uses `PTRACE_TRACEME` to detect debugger |
| RDTSC timing | Compares CPU cycles to detect single-stepping |
| INT3 scanning | Searches code for software breakpoints (0xCC) |
| Hardware BP check | Checks Debug Registers (DR0-DR3) |
| Self-integrity | SHA256 hash of code section |
| Key derivation | Decryption key depends on check results |

---

## Solution

### Step 1: Static Analysis Setup

```bash
# Open in disassembler without executing
ghidra antidebug_hell &
# OR
r2 -A antidebug_hell
```

### Step 2: Identify Anti-Debug Checks

Locate these patterns in the binary:

**ptrace syscall:**
```asm
mov rax, 101    ; sys_ptrace
mov rdi, 0      ; PTRACE_TRACEME
syscall
test rax, rax
jns continue
jmp fail
```

**RDTSC timing:**
```asm
rdtsc           ; First timestamp
mov rbx, rax
... code ...
rdtsc           ; Second timestamp
sub rax, rbx
cmp rax, 0x1000000  ; Threshold
jb continue
jmp timing_fail
```

**INT3 scan:**
```asm
lodsb
cmp al, 0xCC    ; INT3 opcode
je breakpoint_found
```

### Step 3: Locate Encrypted Data

```bash
strings antidebug_hell | grep -E "MKEY|KDERIV|EFLAG|PHASH"
```

**Key Data Structures:**

| Marker | Offset | Content |
|--------|--------|---------|
| MKEY: | 0x2014 | Master key XOR 0xFFFFFFFF |
| KDERIV: | 0x201C | Key derivation XOR values |
| ISALT: | 0x202C | Integrity salt |
| PHASH: | 0x2044 | Passphrase SHA256 hash |
| EFLAG: | 0x2070 | Encrypted flag |
| DECOYS: | 0x20A0 | Fake flags with corrupted keys |

### Step 4: Understand Key Derivation

The decryption key is derived based on check results:

```python
def derive_key(integrity_ok, timing_ok, no_debugger):
    key = MASTER_KEY  # 0xDEADC0DE
    
    if integrity_ok:
        key ^= 0x12345678  # Integrity XOR
    else:
        key ^= 0xFFFFFFFF  # Corrupts key
    
    if timing_ok:
        key ^= 0x87654321  # Timing XOR
    else:
        key ^= 0xAAAAAAAA  # Corrupts key
    
    if no_debugger:
        key ^= 0xCAFEBABE  # Debugger XOR
    else:
        key ^= 0x55555555  # Corrupts key
    
    return key
```

### Step 5: Calculate Correct Key

When all checks pass (True, True, True):

```python
MASTER_KEY = 0xDEADC0DE

# Step by step calculation:
key = 0xDEADC0DE
key ^= 0x12345678  # = 0xCC9996A6
key ^= 0x87654321  # = 0x4BFCD587
key ^= 0xCAFEBABE  # = 0x81026F39

# Correct key = 0x81026F39
print(f"Correct key: 0x{key:08X}")
```

### Step 6: Decrypt the Flag

```python
import struct

def xor_decrypt(data, key):
    key_bytes = struct.pack('<I', key)  # Little endian
    result = bytearray()
    for i, b in enumerate(data):
        result.append(b ^ key_bytes[i % 4])
    return bytes(result)

# Read binary
with open('antidebug_hell', 'rb') as f:
    binary = f.read()

# Find EFLAG marker (offset 0x2070)
# Length is stored as uint16 at offset 0x2070
eflag_offset = binary.find(b'EFLAG:') + 6
flag_length = struct.unpack('<H', binary[eflag_offset:eflag_offset+2])[0]
encrypted_flag = binary[eflag_offset+2:eflag_offset+2+flag_length]

# Decrypt with the correct key
correct_key = 0x81026F39
flag = xor_decrypt(encrypted_flag, correct_key)
print(flag.decode())
```

---

## Complete Solver

```python
#!/usr/bin/env python3
import struct

def calculate_correct_key():
    """Calculate the key when all anti-debug checks pass"""
    master_key = 0xDEADC0DE
    integrity_xor = 0x12345678
    timing_xor = 0x87654321
    debugger_xor = 0xCAFEBABE
    
    key = master_key ^ integrity_xor ^ timing_xor ^ debugger_xor
    return key

def xor_decrypt(data, key):
    key_bytes = struct.pack('<I', key)
    return bytes([b ^ key_bytes[i % 4] for i, b in enumerate(data)])

def extract_encrypted_flag(binary_path):
    with open(binary_path, 'rb') as f:
        data = f.read()
    
    # Find EFLAG section
    marker = b'EFLAG:'
    idx = data.find(marker)
    if idx == -1:
        raise ValueError("EFLAG marker not found")
    
    # Extract length and data
    length_offset = idx + len(marker)
    flag_length = struct.unpack('<H', data[length_offset:length_offset+2])[0]
    encrypted = data[length_offset+2:length_offset+2+flag_length]
    
    return encrypted

def main():
    # Calculate the correct key
    correct_key = calculate_correct_key()
    print(f"Correct decryption key: 0x{correct_key:08X}")
    
    # Extract and decrypt flag
    encrypted_flag = extract_encrypted_flag('antidebug_hell')
    flag = xor_decrypt(encrypted_flag, correct_key)
    print(f"Flag: {flag.decode()}")

if __name__ == '__main__':
    main()
```

---

## Bypass Techniques (Alternative Approach)

If you need to run the binary dynamically:

1. **Patch ptrace check** - NOP out the comparison
2. **Fake RDTSC** - Use LD_PRELOAD to hook rdtsc
3. **Disable INT3 scan** - Patch the scan loop
4. **Hardware breakpoint avoidance** - Use memory watchpoints instead

---

## Tools Used

- **Ghidra / IDA Pro** - Static analysis
- **GDB with anti-anti-debug scripts** - Dynamic analysis
- **Python** - Key calculation and decryption
- **radare2** - Binary patching

---

## Learning Points

1. Anti-debug checks often affect key derivation, not just program flow
2. Static analysis can bypass all runtime checks
3. Understanding the key derivation logic is more valuable than bypassing checks
4. Decoy data often uses intentionally corrupted keys

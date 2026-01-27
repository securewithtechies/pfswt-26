# Obfuscated Maze (250 Points)

**Category:** Reverse Engineering  
**Points:** 250  
**Difficulty:** Very Hard

---

## Challenge Overview

Navigate through a control-flow-flattened binary with opaque predicates and rolling XOR encryption.

---

## Challenge Details

**Files Provided:**
- `obfuscated_maze` - ELF binary with obfuscation
- `maze.dat` - Configuration file

**Flag:** `pfswt{0bfusc4t10n_1s_n0t_s3cur1ty_SWT_m4z3}`  
**Password:** `M4z3_Runn3r_2026!`

---

## Solution

### Step 1: Analyze Configuration File

```bash
xxd maze.dat | head -30
# Find: MAZE_CFG_v2.0 header
# Find: Hash parameters and encrypted flag
```

### Step 2: Understand Control Flow Flattening

The binary uses a state machine dispatcher pattern:

| State | Purpose |
|-------|---------|
| 0 | Initialize |
| 1 | Read input |
| 2 | Validate |
| 3 | Success (print flag) |
| 4 | Fail |
| 5-15 | Decoy paths (dead code) |

### Step 3: Identify Obfuscation Patterns

**Opaque Predicates** - Conditions that always evaluate the same way:

```asm
mov eax, 9      ; 3*3
add eax, 16     ; + 4*4
cmp eax, 25     ; == 5*5 (Pythagorean triple - always true!)
je real_code    ; Always taken
jmp fake_code   ; Never taken
```

### Step 4: Find Hash Parameters

```python
# In binary data section, look for HPARAM marker
# HASH_MULTIPLIER = 31337
# HASH_MODULO = 0xDEADBEEF
```

### Step 5: Reverse the Hash Function

The custom hash function:

```python
def custom_hash(data):
    h = 0x1337
    for i, b in enumerate(data):
        h = ((h * 31337) + b + (i * 7)) % 0xDEADBEEF
    return h
```

**Weakness:** The multiplier 31337 and modulo 0xDEADBEEF allow for collision attacks.

### Step 6: Decrypt with Rolling XOR

```python
def rolling_xor_decrypt(data, seed):
    result = bytearray()
    key = seed
    for b in data:
        result.append(b ^ (key & 0xFF))
        # Linear Congruential Generator
        key = (key * 1103515245 + 12345) & 0x7FFFFFFF
    return bytes(result)

# Seed for real flag: 0xCAFEBABE
# Seed for decoys: 0xDEADBEEF (different!)

# Read encrypted flag from EFLAG section
with open('maze.dat', 'rb') as f:
    data = f.read()

# Find EFLAG marker and extract data
eflag_idx = data.find(b'EFLAG:')
encrypted = extract_data_after(data, eflag_idx)

# Decrypt with correct seed
flag = rolling_xor_decrypt(encrypted, 0xCAFEBABE)
print(flag.decode())
```

### Step 7: Avoid Decoys

**Critical Distinction:**
- `DECOYS` section uses seed `0xDEADBEEF`
- `EFLAG` section uses seed `0xCAFEBABE`
- Only decrypt with the `RXSEED` value!

---

## Key Insights

| Technique | Purpose | Counter |
|-----------|---------|---------|
| Control Flow Flattening | Hide real logic | Trace state transitions |
| Opaque Predicates | Confuse analysis | Recognize mathematical constants |
| Rolling XOR | Dynamic encryption | Extract seed and simulate |
| Decoy Seeds | Mislead attackers | Verify correct marker/seed pairs |

---

## Complete Solver

```python
#!/usr/bin/env python3

def rolling_xor_decrypt(data, seed):
    result = bytearray()
    key = seed
    for b in data:
        result.append(b ^ (key & 0xFF))
        key = (key * 1103515245 + 12345) & 0x7FFFFFFF
    return bytes(result)

def find_section(data, marker):
    idx = data.find(marker)
    if idx == -1:
        return None
    start = idx + len(marker)
    # Read length prefix (2 bytes, little-endian)
    length = int.from_bytes(data[start:start+2], 'little')
    return data[start+2:start+2+length]

# Read maze.dat
with open('maze.dat', 'rb') as f:
    data = f.read()

# Find the correct seed
seed_marker = data.find(b'RXSEED:')
seed = int.from_bytes(data[seed_marker+7:seed_marker+11], 'little')
print(f"Found seed: 0x{seed:08X}")

# Extract and decrypt flag
encrypted_flag = find_section(data, b'EFLAG:')
flag = rolling_xor_decrypt(encrypted_flag, seed)
print(f"Flag: {flag.decode()}")
```

---

## Tools Used

- **Ghidra** - Control flow analysis and decompilation
- **Python** - Hash analysis and XOR decryption
- **GDB** - Dynamic state machine tracing
- **xxd/hexdump** - Binary inspection

---

## Learning Points

1. Control flow flattening transforms structured code into a state machine
2. Opaque predicates use mathematical truths (e.g., Pythagorean triples)
3. Rolling XOR changes the key for each byte, making static analysis harder
4. Always verify which seed/key corresponds to which encrypted section

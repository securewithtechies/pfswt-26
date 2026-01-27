# VM Madness (750 Points)

**Category:** Reverse Engineering  
**Points:** 750  
**Difficulty:** Nightmare

---

## Challenge Overview

Reverse engineer a custom Virtual Machine with encrypted bytecode to extract the hidden flag.

---

## Challenge Details

**Files Provided:**
- `vm_madness` - Custom VM executable (ELF x64)
- `challenge.vmc` - Encrypted VM bytecode
- `vm_state.dat` - Initial VM state and memory

**Flag:** `pfswt{vm_r3v3rs1ng_m4st3r_SWT_1337_h4ck3r}`

---

## VM Architecture

| Component | Details |
|-----------|---------|
| Registers | R0-R7 (32-bit), SP, IP, FL |
| Memory | 16KB total (code/data/stack/heap) |
| Instruction Size | 5 bytes per instruction |
| Encryption | 3-layer (XOR + S-box + Scramble) |

---

## Solution

### Step 1: Analyze VM State File

```python
import struct

with open("vm_state.dat", "rb") as f:
    data = f.read()

# Parse header
magic = data[0:4]         # "VMST"
version = struct.unpack('<I', data[4:8])[0]
seed = struct.unpack('<I', data[0x34:0x38])[0]  # 0xCAFEBABE

# Extract S-box (256 bytes at offset 0x38)
sbox = data[0x38:0x138]

# Extract scramble table (256 bytes at offset 0x138)
scramble = data[0x138:0x238]

# Find key parts
kparts_off = data.find(b'KPARTS:') + 7
key_parts = [data[kparts_off + i] ^ 0xAA for i in range(4)]
# Result: [0x53, 0x57, 0x54, 0x21] = "SWT!"

print(f"Seed: 0x{seed:08X}")
print(f"Key parts: {''.join(chr(k) for k in key_parts)}")
```

### Step 2: Generate Inverse Tables

```python
def inverse_sbox(sbox):
    """Create inverse S-box for decryption"""
    inv = [0] * 256
    for i, v in enumerate(sbox):
        inv[v] = i
    return bytes(inv)

def inverse_scramble(scramble):
    """Create inverse scramble table"""
    inv = [0] * 256
    for i, v in enumerate(scramble):
        inv[v] = i
    return bytes(inv)

inv_sbox = inverse_sbox(sbox)
inv_scramble = inverse_scramble(scramble)
```

### Step 3: Decrypt Bytecode

```python
def rolling_key(ip, seed):
    """Generate position-dependent key"""
    return ((ip * 31337) ^ seed) & 0xFF

def decrypt_instruction(enc_bytes, ip, inv_sbox, inv_scramble, seed):
    """Decrypt a single 5-byte instruction"""
    data = bytearray(enc_bytes)
    
    # Reverse Layer 3: Unscramble
    if len(data) >= 5:
        temp = bytearray(5)
        for i in range(5):
            temp[i] = data[inv_scramble[i] % 5]
        data = temp
    
    # Reverse Layer 2: Inverse S-box
    data = bytearray(inv_sbox[b] for b in data)
    
    # Reverse Layer 1: XOR with rolling key
    key = rolling_key(ip, seed)
    data = bytearray(b ^ key for b in data)
    
    return bytes(data)
```

### Step 4: Parse Bytecode File

```python
with open("challenge.vmc", "rb") as f:
    vmc = f.read()

# Parse header
magic = vmc[0:4]          # "VMC\x00"
version = struct.unpack('<I', vmc[4:8])[0]
code_len = struct.unpack('<I', vmc[8:12])[0]
vmc_seed = struct.unpack('<I', vmc[12:16])[0]

# Encrypted bytecode starts at offset 16
encrypted_code = vmc[16:16 + code_len]

# Decrypt all instructions
instructions = []
for i in range(0, len(encrypted_code), 5):
    enc = encrypted_code[i:i+5]
    dec = decrypt_instruction(enc, i // 5, inv_sbox, inv_scramble, vmc_seed)
    instructions.append(dec)

print(f"Decrypted {len(instructions)} instructions")
```

### Step 5: Disassemble Instructions

```python
OPCODES = {
    0x00: "NOP",
    0x10: "MOV",   0x11: "MOVI",
    0x20: "PUSH",  0x21: "POP",
    0x30: "ADD",   0x31: "SUB",   0x32: "XOR",
    0x33: "AND",   0x34: "OR",
    0x35: "SHL",   0x36: "SHR",   0x37: "ROL",   0x38: "ROR",
    0x40: "JMP",   0x41: "JZ",    0x42: "JNZ",   0x43: "JE",    0x44: "JNE",
    0x50: "CALL",  0x51: "RET",
    0x60: "LOAD",  0x61: "STORE", 0x62: "LOADI",
    0x70: "CMP",   0x71: "TEST",
    # Hidden opcodes
    0x80: "SWAP",
    0x81: "BSWAP",
    0x82: "CPUID",
    0x90: "SMOD",   # Self-modify
    0x91: "VKEY",   # Get VM key
    # Crypto operations
    0xF0: "CRYPT", 0xF1: "DCRYPT", 0xF2: "SBOX",
    0xFE: "TRAP",  0xFF: "HALT"
}

def disassemble(instr):
    """Disassemble a single instruction"""
    opcode = instr[0]
    mod = (instr[1] >> 4) & 0x0F
    op1 = ((instr[1] & 0x0F) << 8) | instr[2]
    op2 = (instr[3] << 8) | instr[4]
    
    name = OPCODES.get(opcode, f"UNK_{opcode:02X}")
    return f"{name:8s} mod={mod} op1=0x{op1:03X} op2=0x{op2:04X}"

# Disassemble all instructions
for i, instr in enumerate(instructions):
    print(f"{i:04d}: {disassemble(instr)}")
```

### Step 6: Analyze VM Logic

Key observations from disassembly:
1. VM loads encrypted flag from data section
2. Uses custom SBOX and XOR operations
3. Compares result with hardcoded hash
4. Prints flag on successful validation

### Step 7: Extract Flag from VM Memory

```python
def emulate_vm(instructions, initial_memory):
    """Simple VM emulator to extract flag"""
    regs = [0] * 8
    memory = bytearray(initial_memory)
    ip = 0
    
    while ip < len(instructions):
        instr = instructions[ip]
        opcode = instr[0]
        
        if opcode == 0xFF:  # HALT
            break
        elif opcode == 0x91:  # VKEY - Key recovery
            # Extract the key used for flag decryption
            key_offset = (instr[3] << 8) | instr[4]
            print(f"Key at offset 0x{key_offset:04X}")
        # ... implement other opcodes as needed
        
        ip += 1
    
    return memory

# Find flag in decrypted memory
memory = emulate_vm(instructions, initial_memory)
flag_start = memory.find(b'pfswt{')
if flag_start != -1:
    flag_end = memory.find(b'}', flag_start) + 1
    print(f"Flag: {memory[flag_start:flag_end].decode()}")
```

---

## Complete Solver

```python
#!/usr/bin/env python3
import struct

def main():
    # Load VM state
    with open("vm_state.dat", "rb") as f:
        state = f.read()
    
    sbox = state[0x38:0x138]
    scramble = state[0x138:0x238]
    seed = struct.unpack('<I', state[0x34:0x38])[0]
    
    # Generate inverse tables
    inv_sbox = bytes([sbox.index(i) if i in sbox else 0 for i in range(256)])
    inv_scramble = bytes([scramble.index(i) if i in scramble else 0 for i in range(256)])
    
    # Load and decrypt bytecode
    with open("challenge.vmc", "rb") as f:
        vmc = f.read()
    
    code_len = struct.unpack('<I', vmc[8:12])[0]
    encrypted_code = vmc[16:16 + code_len]
    
    # Decrypt each instruction
    decrypted = bytearray()
    for i in range(0, len(encrypted_code), 5):
        enc = encrypted_code[i:i+5]
        key = ((i // 5 * 31337) ^ seed) & 0xFF
        
        # Layer 1: XOR
        dec = bytearray(b ^ key for b in enc)
        # Layer 2: Inverse S-box
        dec = bytearray(inv_sbox[b] for b in dec)
        # Layer 3: Unscramble
        temp = bytearray(5)
        for j in range(5):
            temp[j] = dec[inv_scramble[j] % 5]
        
        decrypted.extend(temp)
    
    # Find flag in decrypted data
    flag_marker = decrypted.find(b'pfswt{')
    if flag_marker != -1:
        end = decrypted.find(b'}', flag_marker)
        print(f"Flag: {decrypted[flag_marker:end+1].decode()}")
    else:
        # Flag might be in a separate data section
        print("Analyzing VM instructions for flag location...")

if __name__ == '__main__':
    main()
```

---

## Tools Used

- **Ghidra / IDA Pro** - VM dispatcher analysis
- **Python** - Bytecode decryption and emulation
- **Custom VM emulator** - Execute decrypted bytecode
- **GDB** - Dynamic analysis of VM interpreter

---

## Learning Points

1. Custom VMs add a layer of abstraction that must be reversed first
2. Encrypted bytecode requires understanding the encryption scheme
3. S-boxes and scramble tables are common in VM protection
4. Position-dependent keys (rolling keys) complicate static analysis
5. Hidden opcodes may contain critical functionality

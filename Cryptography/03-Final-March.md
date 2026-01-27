# The Final March (500 Points)

**Category:** Cryptography  
**Points:** 500  
**Difficulty:** Hard  
**Theme:** Matangini Hazra - "Gandhi Buri"

---

## Challenge Overview

This challenge combines **GIF Frame Timing Steganography** with the **Trifid Cipher**.

---

## Prerequisites

- Challenge 100 Flag: `pfswt{G4ndh1_Bur1@73_V4nD3_M4t4r4m!1942}` → Extract: **GANDHIBURI**
- Challenge 250 Flag: `pfswt{N1h1l1st_Sp3ctr0_H4zr4_73!}` → Extract: **NIHILIST**

---

## Solution

### Step 1: Analyze the GIF File

Examine the GIF structure to find hidden timing data:

**Using gifsicle:**
```bash
gifsicle -I flag_bearer.gif
```

**Using Python with PIL:**
```python
from PIL import Image

gif = Image.open('flag_bearer.gif')
timings = []

try:
    while True:
        # Get frame duration in milliseconds
        duration = gif.info.get('duration', 100)
        timings.append(duration)
        gif.seek(gif.tell() + 1)
except EOFError:
    pass

print(f"Frame timings: {timings}")
```

**Expected Output:**
Alternating values around 50ms and 150ms, representing binary 0 and 1.

### Step 2: Convert Timings to Binary

The encoding scheme:
- **~50ms delay** = Binary **0**
- **~150ms delay** = Binary **1**

```python
threshold = 100  # Midpoint between 50 and 150

binary = ''
for timing in timings:
    if timing < threshold:
        binary += '0'
    else:
        binary += '1'

print(f"Binary: {binary}")
```

### Step 3: Convert Binary to Text

```python
# Group into bytes (8 bits each)
text = ''
for i in range(0, len(binary), 8):
    byte = binary[i:i+8]
    if len(byte) == 8:
        char = chr(int(byte, 2))
        text += char

print(f"Extracted ciphertext: {text}")
```

This yields the Trifid-encrypted ciphertext.

### Step 4: Combine Keys from Previous Challenges

**Combined Key:**
```
GANDHIBURINIHILIST
```

**Trifid Period:**
The period **7** is hinted in the README (73 = code for "farewell" in amateur radio, the digits 7 and 3).

### Step 5: Trifid Cipher Decryption

The Trifid cipher uses a 3×3×3 cube (27 positions for 26 letters + period):

```python
def trifid_decrypt(ciphertext, key, period):
    # Build 3x3x3 cube from key
    alphabet = build_keyed_alphabet(key)
    cube = create_trifid_cube(alphabet)
    
    # Process in groups of 'period' characters
    plaintext = ""
    for i in range(0, len(ciphertext), period):
        group = ciphertext[i:i+period]
        
        # Convert each letter to its 3D coordinates
        coords = [get_coordinates(cube, c) for c in group]
        
        # Transpose coordinates
        layers = [c[0] for c in coords]
        rows = [c[1] for c in coords]
        cols = [c[2] for c in coords]
        
        # Combine and split into new groups of 3
        combined = layers + rows + cols
        new_coords = [(combined[j], combined[j+len(group)], combined[j+2*len(group)]) 
                      for j in range(len(group))]
        
        # Look up letters
        for coord in new_coords:
            plaintext += cube[coord[0]][coord[1]][coord[2]]
    
    return plaintext
```

---

## Flag

```
pfswt{Tr1f1d_T1m3_Fr4ct10n@D3l4st3ll3!}
```

---

## Flag Breakdown

| Part | Meaning |
|------|---------|
| `Tr1f1d` | Trifid cipher |
| `T1m3` | Timing-based steganography |
| `Fr4ct10n` | Fractionation (Trifid technique) |
| `D3l4st3ll3` | Delastelle - inventor of Trifid cipher |

---

## Technical Details

### GIF Timing Steganography
GIF files store frame delays in centiseconds. By varying these delays slightly, binary data can be encoded without visually affecting the animation.

### Trifid Cipher
Invented by Félix Delastelle in 1902, the Trifid cipher is a classical cipher that uses a 3×3×3 cube for fractionation - converting letters into coordinates, transposing, and converting back.

---

## Tools Used

- **gifsicle** - GIF analysis
- **Python PIL** - Frame extraction
- **Custom scripts** - Trifid implementation

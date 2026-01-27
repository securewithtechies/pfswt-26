# The Voice of the Procession (250 Points)

**Category:** Cryptography  
**Points:** 250  
**Difficulty:** Medium  
**Theme:** Matangini Hazra - "Gandhi Buri"

---

## Challenge Overview

This challenge hides encrypted numbers in an audio file's spectrogram. The numbers are encrypted using the **Nihilist cipher**, which requires a key derived from the previous challenge.

---

## Prerequisites

From Challenge 100 (The Forgotten Flag Bearer):
- The numeric portion `731942` (73 = her age, 1942 = year)

---

## Solution

### Step 1: Audio Spectrogram Analysis

Open the audio file in a spectrogram viewer:

**Using Audacity:**
1. Open `procession.wav`
2. Click dropdown menu on track → Spectrogram

**Using Sonic Visualiser:**
```bash
sonic-visualiser procession.wav
# Add spectrogram layer
```

**Using Sox (command line):**
```bash
sox procession.wav -n spectrogram -o spectrogram.png
```

**Expected Result:**
Numbers visible in the spectrogram, appearing as shapes formed by frequency patterns:
```
45 67 89 34 56 78 90 ...
```

### Step 2: Extract the Numbers

Read the two-digit numbers from the spectrogram image. These are the encrypted Nihilist cipher values.

### Step 3: Cipher Identification

Story clues point to the Nihilist cipher:
- "Russian revolutionaries" → Nihilist cipher origin
- "Nihilists had perfected the art" → Direct reference
- Uses Polybius square + numeric addition

### Step 4: Determine Keys

1. **Polybius Key:** `BENGAL` (the region mentioned in story)
2. **Message Key:** `731942` (from previous challenge - age 73, year 1942)

### Step 5: Build Polybius Square

Using key `BENGAL` (5×5 grid, I/J combined):

```
  | 1 2 3 4 5
--+-----------
1 | B E N G A
2 | L C D F H
3 | I K M O P
4 | Q R S T U
5 | V W X Y Z
```

### Step 6: Nihilist Cipher Decryption

The message key `731942` is used as a repeating numeric key:
```
Key stream: 7-3-1-9-4-2-7-3-1-9-4-2...
```

**Decryption Process:**

For each encrypted number:
1. Subtract the corresponding key digit
2. Split result into row and column
3. Look up letter in Polybius square

```python
def nihilist_decrypt(ciphertext_numbers, polybius_key, message_key):
    # Build Polybius square from key
    square = build_polybius_square(polybius_key)
    
    # Repeat message key to match ciphertext length
    key_stream = (message_key * ((len(ciphertext_numbers) // len(message_key)) + 1))
    
    plaintext = ""
    for i, num in enumerate(ciphertext_numbers):
        # Subtract key digit
        decrypted_num = num - int(key_stream[i])
        
        # Extract row and column (each digit is row/col)
        row = decrypted_num // 10
        col = decrypted_num % 10
        
        # Look up letter
        plaintext += square[row-1][col-1]
    
    return plaintext
```

---

## Flag

```
pfswt{N1h1l1st_Sp3ctr0_H4zr4_73!}
```

---

## Flag Breakdown

| Part | Meaning |
|------|---------|
| `N1h1l1st` | Nihilist - the cipher used |
| `Sp3ctr0` | Spectrogram - the steganography technique |
| `H4zr4` | Hazra - Matangini Hazra |
| `73!` | Her age at martyrdom |

---

## Tools Used

- **Audacity** - Audio spectrogram visualization
- **Sonic Visualiser** - Advanced audio analysis
- **Sox** - Command-line audio processing
- **Python** - Nihilist cipher implementation

---

## Learning Points

1. Audio steganography can hide visual data in spectrograms
2. Nihilist cipher combines Polybius square with numeric key addition
3. CTF challenges often chain together, using previous answers as keys

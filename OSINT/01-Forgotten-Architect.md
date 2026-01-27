# The Forgotten Architect (100 Points)

**Category:** OSINT  
**Points:** 100  
**Difficulty:** Easy  
**Theme:** Pingali Venkayya - Designer of the Indian Flag

---

## Challenge Overview

Find the birthplace of the forgotten flag designer through LSB steganography and Base64 decoding.

---

## Solution

### Step 1: Extract LSB Steganography

The challenge file `1921_India_flag.png` contains hidden data using LSB (Least Significant Bit) steganography.

**Using Python (stegano library):**
```python
from stegano import lsb

hidden = lsb.reveal('1921_India_flag.png')
print(hidden)
```

**Using zsteg (Ruby):**
```bash
zsteg 1921_India_flag.png
```

**Extracted Data (Base64):**
```
QmhhdGxhcGVudW1hcnJ1XzE2LjMwMDBOXzgxLjM4MzNF
```

### Step 2: Decode Base64

```bash
echo "QmhhdGxhcGVudW1hcnJ1XzE2LjMwMDBOXzgxLjM4MzNF" | base64 -d
```

**Or using Python:**
```python
import base64
decoded = base64.b64decode("QmhhdGxhcGVudW1hcnJ1XzE2LjMwMDBOXzgxLjM4MzNF").decode()
print(decoded)
```

**Result:** `Bhatlapenumarru_16.3000N_81.3833E`

### Step 3: Analyze the Decoded Data

The decoded string contains:
- **Village Name:** Bhatlapenumarru
- **GPS Coordinates:** 16.3000°N, 81.3833°E

### Step 4: Research and Verification

Cross-reference with historical data:
- **Location:** Bhatlapenumarru village, Krishna district, Andhra Pradesh
- **Significance:** Birthplace of Pingali Venkayya (1876-1963)
- **Connection:** Pingali Venkayya designed the original Indian National Flag in 1921
- **Verification:** Wikipedia and historical records confirm this as his birthplace

### Step 5: Construct the Flag

Combine the village name with his birth year:
- Village: `Bhatlapenumarru`
- Birth Year: `1876`

---

## Flag

```
pfswt{Bhatlapenumarru_1876}
```

---

## Historical Context

**Pingali Venkayya** (1876-1963) was an Indian freedom fighter and the designer of the Indian National Flag. Born in Bhatlapenumarru village in Andhra Pradesh, he dedicated over 30 years of his life to designing a flag for India. His original design, presented at the 1921 Indian National Congress session, formed the basis for the current national flag.

---

## Tools Used

- `stegano` (Python) - LSB steganography extraction
- `zsteg` (Ruby) - Alternative steganography tool
- Base64 decoder
- Wikipedia/historical research

---

## Learning Points

1. LSB steganography can hide data in images without visible changes
2. Base64 is a common encoding layer for hidden data
3. OSINT often requires combining technical skills with research
4. Historical verification is essential in OSINT challenges

---

## Connection to Next Challenge

The village name `BHATLAPENUMARRU` is the Vigenère cipher key for Challenge 2.

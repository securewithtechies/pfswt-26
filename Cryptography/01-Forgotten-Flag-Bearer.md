# The Forgotten Flag Bearer (100 Points)

**Category:** Cryptography  
**Points:** 100  
**Difficulty:** Easy  
**Theme:** Matangini Hazra - "Gandhi Buri"

---

## Challenge Overview

This challenge hides an encrypted flag inside a PNG image using custom PNG chunks. The encrypted text is encoded using the ancient Indian cipher **"Mlecchita Vikalpa"**.

---

## Solution

### Step 1: Analyze the PNG File

Use standard forensics tools to examine the image:

```bash
# Check PNG structure
pngcheck -v memorial.png

# Extract metadata
exiftool memorial.png

# Scan for embedded data
binwalk memorial.png
```

**Expected Findings:**
- Standard PNG structure (IHDR, IDAT, IEND chunks)
- Additional text chunks: `tEXt` and `zTXt`
- Non-standard keywords in chunk metadata

### Step 2: Extract Hidden Data

Look for custom PNG chunks containing hidden data:

```bash
# Check for text chunks
pngcheck -t memorial.png
```

Using Python for manual extraction:

```python
import zlib

with open('memorial.png', 'rb') as f:
    data = f.read()

# Search for 'mlecchita' keyword in zTXt chunk
idx = data.find(b'mlecchita')
if idx != -1:
    # Parse the zTXt chunk structure
    # Format: keyword + null + compression_method + compressed_data
    print("Found mlecchita cipher reference!")
```

**Extracted Information:**
- Keyword: `mlecchita`
- Hint 1: `year=1942`
- Hint 2: `place=TAMLUK`

### Step 3: Identify the Cipher

The keyword "mlecchita" refers to **Mlecchita Vikalpa**, one of the 64 arts mentioned in the ancient text Kama Sutra (~400 BC). It literally means "the art of understanding writing in cipher."

**Cipher Parameters:**
- Key: `TAMLUK` (the place where Matangini Hazra was martyred)
- Seed: `1942` (the year of her sacrifice)

### Step 4: Decrypt the Message

The Mlecchita Vikalpa cipher uses:
1. Keyed alphabet substitution
2. Syllable-based shifting controlled by the seed

**Decryption Process:**

1. Calculate shift value: `sum(ord(c) for c in "TAMLUK") % 7 + 1`
2. Build keyed alphabet from "TAMLUK"
3. Apply paired letter swaps (controlled by seed 1942)
4. Reverse the substitution mapping

Using the tool:
```bash
python mlecchita_vikalpa.py --decode --key TAMLUK --seed 1942 --text "ENCRYPTED_TEXT"
```

---

## Flag

```
pfswt{G4ndh1_Bur1@73_V4nD3_M4t4r4m!1942}
```

---

## Flag Breakdown

| Part | Meaning |
|------|---------|
| `G4ndh1_Bur1` | Gandhi Buri - her nickname meaning "Old Lady Gandhi" |
| `@73` | Her age at martyrdom (73 years old) |
| `V4nD3_M4t4r4m!` | Vande Mataram - her last words while dying |
| `1942` | Year of her sacrifice |

---

## Historical Context

**Matangini Hazra** (1870-1942), known as "Gandhi Buri" (Old Lady Gandhi), was a 73-year-old widow who became a martyr of the Quit India Movement. On September 29, 1942, she led a procession of 6,000 protesters to hoist the Indian tricolor at the Tamluk police station. When British forces opened fire, she was shot three times but kept marching forward, the flag clutched in her dying hands, her lips chanting "Vande Mataram" until her very last breath.

---

## Tools Used

- `pngcheck` - PNG structure analysis
- `binwalk` - Embedded data extraction
- `exiftool` - Metadata analysis
- Python - Custom decryption

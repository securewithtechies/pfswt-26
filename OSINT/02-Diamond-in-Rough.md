# The Diamond in the Rough (250 Points)

**Category:** OSINT  
**Points:** 250  
**Difficulty:** Medium  
**Theme:** Pingali Venkayya - Designer of the Indian Flag

---

## Challenge Overview

Decode an encrypted researcher profile using the key from the previous challenge.

---

## Prerequisites

From Challenge 1, we discovered the village name: **BHATLAPENUMARRU**  
This is our Vigenère cipher key.

---

## Solution

### Step 1: Analyze the Encrypted Profile

The `encrypted_profile.txt` contains several encrypted fields:

```
Name: QPNZLLX ZRHWAPPU
Aliases: Epafzns Zrhwappu, Khpty Vtrxukyr
Born: 1876, Coamwaeiaoyaiio
Profession: Hloezgxwg

NOTABLE NICKNAMES:
- Epafzns Zrhwappu (mining expertise)
- Dvtmzn Kiaemypr (agricultural research)
- Khpty Vtrxukyr (language skills)

Location: Uhnd Much Uspeirvbk
```

### Step 2: Vigenère Decryption

**Using Python:**
```python
def vigenere_decrypt(ciphertext, key):
    result = ''
    key = key.upper()
    key_index = 0
    
    for char in ciphertext:
        if char.isalpha():
            shift = ord(key[key_index % len(key)]) - ord('A')
            if char.isupper():
                decrypted = chr((ord(char) - ord('A') - shift) % 26 + ord('A'))
            else:
                decrypted = chr((ord(char) - ord('a') - shift) % 26 + ord('a'))
            result += decrypted
            key_index += 1
        else:
            result += char
    
    return result

key = "BHATLAPENUMARRU"

# Decrypt each field
print(vigenere_decrypt("QPNZLLX ZRHWAPPU", key))
# Output: PINGALI VENKAYYA

print(vigenere_decrypt("Epafzns Zrhwappu", key))
# Output: Diamond Venkayya

print(vigenere_decrypt("Coamwaeiaoyaiio", key))
# Output: Bhatlapenumarru

print(vigenere_decrypt("Hloezgxwg", key))
# Output: Geologist
```

**Using Online Tool:**
Use any Vigenère decoder (e.g., dcode.fr/vigenere-cipher) with key: `BHATLAPENUMARRU`

### Step 3: Decryption Results

| Encrypted | Decrypted |
|-----------|-----------|
| QPNZLLX ZRHWAPPU | PINGALI VENKAYYA |
| Epafzns Zrhwappu | Diamond Venkayya |
| Dvtmzn Kiaemypr | Cotton Venkayya |
| Khpty Vtrxukyr | Japan Venkayya |
| Coamwaeiaoyaiio | Bhatlapenumarru |
| Hloezgxwg | Geologist |
| Uhnd Much Uspeirvbk | Tank Bund Hyderabad |

### Step 4: Analyze Decrypted Information

From the decrypted profile:
- **Full Name:** Pingali Venkayya
- **Famous Nickname:** Diamond Venkayya (due to his expertise in diamond mining)
- **Other Nicknames:** Cotton Venkayya, Japan Venkayya
- **Profession:** Geologist
- **Memorial Location:** Tank Bund, Hyderabad

### Step 5: Construct the Flag

Based on the decrypted data:
- Nickname: `Diamond`
- Surname: `Venkayya`
- Profession: `Geologist`

---

## Flag

```
pfswt{Diamond_Venkayya_Geologist}
```

---

## Historical Context

Pingali Venkayya earned several nicknames during his lifetime:
- **Diamond Venkayya:** For his expertise in mining and geology
- **Cotton Venkayya:** For his research on cotton cultivation
- **Japan Venkayya:** For his fluency in Japanese

He was a polymath who spoke multiple languages, conducted agricultural research, and served in the British Indian Army during the Second Boer War, where he met Mahatma Gandhi.

---

## Tools Used

- Vigenère cipher decoder
- Python for scripting
- Online cipher tools (dcode.fr)

---

## Learning Points

1. CTF challenges often chain together - previous answers become keys
2. Vigenère cipher is a polyalphabetic substitution cipher
3. The key must match exactly (case-insensitive, but spelling matters)
4. Historical research validates OSINT findings

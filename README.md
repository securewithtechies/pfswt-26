# 🚩 Phantom Flags CTF - Official Writeups

<div align="center">

![CTF Banner](https://img.shields.io/badge/Phantom%20Flags%20CTF-January%2026,%202026-purple?style=for-the-badge)
![Organizer](https://img.shields.io/badge/Secure%20With%20Techies-SWT-blue?style=for-the-badge)
![Flag Format](https://img.shields.io/badge/Flag%20Format-pfswt%7B...%7D-green?style=for-the-badge)

**Official solutions for all challenges from Phantom Flags CTF 2026**

*Organized by Secure With Techies (SWT) on Republic Day - January 26, 2026*

</div>

---

## 📋 Table of Contents

| Category | Challenges | Total Points |
|----------|-----------|-------------|
| [Welcome](./Welcome/) | 5 | 50 |
| [Cryptography](./Cryptography/) | 4 | 1600 |
| [Web Exploitation](./WebExploitation/) | 4 | 1600 |
| [Forensics](./Forensics/) | 2 | 600 |
| [Reverse Engineering](./ReverseEngineering/) | 4 | 1600 |
| [OSINT](./OSINT/) | 3 | 850 |

---

## 🏆 Challenge Overview

### Welcome (50 Points)
Introductory challenges to familiarize participants with the CTF format.

| Challenge | Points | Writeup |
|-----------|--------|---------|
| First Blood | 10 | [Solution](./Welcome/01-First-Blood.md) |
| Republic Day Special | 10 | [Solution](./Welcome/02-Republic-Day-Special.md) |
| Know Your Host | 10 | [Solution](./Welcome/03-Know-Your-Host.md) |
| Thank Our Sponsors | 10 | [Solution](./Welcome/04-Thank-Our-Sponsors.md) |
| Social Connect | 10 | [Solution](./Welcome/05-Social-Connect.md) |

### Cryptography (1600 Points)
*Theme: Matangini Hazra - "Gandhi Buri" (1870-1942)*

| Challenge | Points | Technique | Writeup |
|-----------|--------|-----------|---------|
| The Forgotten Flag Bearer | 100 | PNG Chunks + Mlecchita Vikalpa | [Solution](./Cryptography/01-Forgotten-Flag-Bearer.md) |
| The Voice of the Procession | 250 | Audio Spectrogram + Nihilist Cipher | [Solution](./Cryptography/02-Voice-of-Procession.md) |
| The Final March | 500 | GIF Frame Timing + Trifid Cipher | [Solution](./Cryptography/03-Final-March.md) |
| The Unwritten Legacy | 750 | JPEG/ZIP Polyglot + VIC Cipher | [Solution](./Cryptography/04-Unwritten-Legacy.md) |

### Web Exploitation (1600 Points)
*Theme: Peer Ali Khan - The First Martyr of 1857*

| Challenge | Points | Technique | Writeup |
|-----------|--------|-----------|---------|
| The Bookseller's Code | 100 | DOM + Symbol Grid Puzzle | [Solution](./WebExploitation/01-Booksellers-Code.md) |
| The Secret Missives | 250 | LocalStorage + JWT Analysis | [Solution](./WebExploitation/02-Secret-Missives.md) |
| The Underground Network | 500 | WebSocket + Cipher Chain | [Solution](./WebExploitation/03-Underground-Network.md) |
| The Final Message | 750 | Multi-Vector Attack Chain | [Solution](./WebExploitation/04-Final-Message.md) |

### Forensics (600 Points)

| Challenge | Points | Technique | Writeup |
|-----------|--------|-----------|--------|
| Easyy Bruhh | 100 | Stegify + Base32 Decoding | [Solution](./Forensics/01-Easyy-Bruhh.md) |
| Silent Harvest | 500 | PCAP Analysis + DNS Exfiltration + Steghide | [Solution](./Forensics/02-Silent-Harvest.md) |

### Reverse Engineering (1600 Points)

| Challenge | Points | Technique | Writeup |
|-----------|--------|-----------|---------|
| Binary Secrets | 100 | XOR Layer Analysis | [Solution](./ReverseEngineering/01-Binary-Secrets.md) |
| Obfuscated Maze | 250 | Control Flow + Rolling XOR | [Solution](./ReverseEngineering/02-Obfuscated-Maze.md) |
| Anti-Debug Hell | 500 | Anti-Debug Bypass | [Solution](./ReverseEngineering/03-Anti-Debug-Hell.md) |
| VM Madness | 750 | Custom VM Analysis | [Solution](./ReverseEngineering/04-VM-Madness.md) |

### OSINT (850 Points)
*Theme: Pingali Venkayya - Designer of the Indian Flag*

| Challenge | Points | Technique | Writeup |
|-----------|--------|-----------|---------|
| The Forgotten Architect | 100 | LSB Steganography + Geo-OSINT | [Solution](./OSINT/01-Forgotten-Architect.md) |
| The Diamond in the Rough | 250 | Vigenère Cipher + Research | [Solution](./OSINT/02-Diamond-in-Rough.md) |
| The 30 Designs of Destiny | 500 | Multi-Cipher + Bifid | [Solution](./OSINT/03-30-Designs-of-Destiny.md) |

---

## 🎯 Flag Format

All flags follow the format:
```
pfswt{flag_content_here}
```

Where `pfswt` stands for **P**hantom **F**lags **S**ecure **W**ith **T**echies.

---

## 🛠️ Recommended Tools

### General
- CyberChef - [gchq.github.io/CyberChef](https://gchq.github.io/CyberChef/)
- dCode - [dcode.fr](https://www.dcode.fr/)

### Cryptography
- Python with `pycryptodome`
- `binwalk`, `exiftool`, `pngcheck`
- Audacity / Sonic Visualiser (audio analysis)

### Web Exploitation
- Browser DevTools (F12)
- Burp Suite
- JavaScript deobfuscation tools

### Reverse Engineering
- Ghidra / IDA Pro
- GDB / x64dbg
- Python scripting

### OSINT
- Google Dorking
- Wayback Machine
- Social media research

---

## 📜 Historical Context

This CTF celebrates unsung heroes of India's freedom struggle:

- **Matangini Hazra** (1870-1942) - 73-year-old martyr of the Quit India Movement
- **Peer Ali Khan** (1786-1857) - First martyr of the 1857 rebellion
- **Pingali Venkayya** (1876-1963) - Designer of the Indian National Flag

---

## 📝 License

These writeups are provided for educational purposes. Feel free to learn from them!

---

<div align="center">

**Made with ❤️ by Secure With Techies**

[Website](https://securewithtechies.com) • [Twitter](https://twitter.com/SecureWithTech) • [Instagram](https://instagram.com/securewithtechies)

</div>


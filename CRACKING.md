# WPA Dictionary Attack with GitHub Actions

## Overview

This repository includes a GitHub Actions workflow for running WPA/WPA2 dictionary attacks using hashcat on GitHub's free infrastructure.

## ⚠️ Legal Notice

**This tool is for AUTHORIZED SECURITY TESTING and EDUCATIONAL PURPOSES ONLY.**

- Only test networks you own or have explicit written permission to test
- Unauthorized access to computer networks is illegal
- The author assumes no responsibility for misuse

## How It Works

GitHub Actions provides free compute time for public repositories:
- **Public repos**: 2,000 minutes/month (free)
- **CPU-only**: ~50,000-100,000 passwords/second
- **No GPU**: GitHub Actions doesn't provide GPU, so attacks are CPU-only

## Usage

### 1. Prepare Your Handshake

Convert your .cap file to .hc22000 format:
```bash
# Using hashcat-utils
cap2hc22000 handshake.cap > handshake.hc22000

# Or using hcxpcapngtool
hcxpcapngtool -o handshake.hc22000 handshake.cap
```

### 2. Add Handshake to Repository

```bash
cp your_handshake.hc22000 .
git add handshake.hc22000
git commit -m "Add handshake file"
git push
```

### 3. Trigger the Workflow

Go to Actions → WPA Dictionary Attack → Run workflow

Parameters:
- `handshake_file`: Path to your .hc22000 file
- `wordlist_url`: URL to download wordlist
- `hashcat_mode`: 22000 for hc22000, 2500 for hccap

## Performance Estimates

| Wordlist Size | Time (CPU) | Passwords/sec |
|---------------|------------|---------------|
| 10,000        | ~2 min     | ~80,000       |
| 100,000       | ~20 min    | ~80,000       |
| 1,000,000     | ~3.5 hours | ~80,000       |

**Note**: GitHub Actions has a 6-hour maximum job timeout.

## Limitations

1. **No GPU**: GitHub Actions runners don't have GPUs
2. **CPU-only**: ~80,000-100,000 passwords/second (vs millions with GPU)
3. **Timeout**: 6-hour maximum per job
4. **Monthly limit**: 2,000 minutes for public repos
5. **Public code**: Your workflow and results are visible to everyone

## Better Alternatives

For serious password cracking:
- **Local GPU**: Use your own GPU (RTX 3060+ recommended)
- **Cloud GPU**: Rent GPU instances (AWS, Lambda Labs, vast.ai)
- **Specialized services**: Hashcat on dedicated hardware

## Wordlist Sources

- [SecLists](https://github.com/danielmiessler/SecLists)
- [CrackStation](https://crackstation.net/crackstation-wordlists/)
- [WeakPass](https://weakpass.com/)

## Example Command (Local)

```bash
# Convert capture to hashcat format
hcxpcapngtool -o handshake.hc22000 capture.cap

# Run attack with GPU
hashcat -m 22000 handshake.hc22000 wordlist.txt -d 1 -w 3

# Check results
hashcat -m 22000 handshake.hc22000 --show
```

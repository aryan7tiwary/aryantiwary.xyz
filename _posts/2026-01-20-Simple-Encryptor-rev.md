---
toc: true
toc_sticky: true
classes: wide 
title: "Simple Encryptor Binary Analysis - Hack The Box"
date: 2026-01-19T12:20:30-05:30
categories:
  - writeup
tags:
  - Simple Encryptor
  - Reverse Engineering
  - HackTheBox
  - Binary Analysis
  - TRNG
  - writeup
---

> Today, this writeup is on a topic I learnt in one of my academic subjects, that is **Principles of Cryptography**. <a href="https://en.wikipedia.org/wiki/Linear_congruential_generator" target="_blank" rel="noopener noreferrer">Linear Congruential Generator</a> is an algorthm used to generate a sequence of pseudo-random numbers.

---
# Introduction
- Platform: HackTheBox
- Challenge Name: Simple Encryptor
- Difficulty: Easy
- Skills required: cryptography, reverse engineering, C

---

# Challenge Overview
I have been provided with a binary file which is ELF (executable linux file) and a data file which contains encrypted flag. My goal is to decypt the flag and obtain the original flag.

![Command: file](/assets/images/Simple Encryptor/file-command.png)

My intuition would be, for an easy challenge, to decompile the ELF file. Any decompiler will work. I'll be using free open-source application - Ghidra. It decompiles and auto-analyzes the binary making it easier to read.

Source Code:
```
undefined8 main(void)
{
  int iVar1;
  time_t tVar2;
  long in_FS_OFFSET;
  uint local_40;
  uint local_3c;
  long local_38;
  FILE *local_30;
  size_t local_28;
  void *local_20;
  FILE *local_18;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_30 = fopen("flag","rb");
  fseek(local_30,0,2);
  local_28 = ftell(local_30);
  fseek(local_30,0,0);
  local_20 = malloc(local_28);
  fread(local_20,local_28,1,local_30);
  fclose(local_30);
  tVar2 = time((time_t *)0x0);
  local_40 = (uint)tVar2;
  srand(local_40);
  for (local_38 = 0; local_38 < (long)local_28; local_38 = local_38 + 1) {
    iVar1 = rand();
    *(byte *)((long)local_20 + local_38) = *(byte *)((long)local_20 + local_38) ^ (byte)iVar1;
    local_3c = rand();
    local_3c = local_3c & 7;
    *(byte *)((long)local_20 + local_38) =
         *(byte *)((long)local_20 + local_38) << (sbyte)local_3c |
         *(byte *)((long)local_20 + local_38) >> 8 - (sbyte)local_3c;
  }
  local_18 = fopen("flag.enc","wb");
  fwrite(&local_40,1,4,local_18);
  fwrite(local_20,1,local_28,local_18);
  fclose(local_18);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

This binary only has a main function and no obfuscation.

---

# Static Analysis
## High-Level Encryption Flow
At a high level, the program:

1. Reads the contents of a file called flag
2. Seeds the PRNG using the current Unix timestamp
3. Encrypts each byte using: XOR with rand()
4. A random bit rotation derived from another rand()
5. Writes:<br>
    The seed (timestamp) <br>
    The encrypted data into flag.enc

The critical weakness lies in how randomness is generated and reused.

## What `rand()` is actually doing

```
iVar1 = rand();
```

The rand() function in C is not truly random. It is a Pseudo-Random Number Generator (PRNG), meaning:
1. It produces deterministic output
2. Given the same seed, it will always generate the same sequence
3. It is designed for simulations, not cryptography

Most implementations of rand() are based on a Linear Congruential Generator (LCG).<br>
```Xₙ₊₁ = (aXₙ + c) mod m```

Where:
- Xₙ is the current state
- a, c, and m are constants

The entire sequence is predictable once X₀ (the seed) is known.

It is very important to know about TRNG and PRNG, you can read it <a href="https://eitca.org/cybersecurity/eitc-is-ccf-classical-cryptography-fundamentals/stream-ciphers/stream-ciphers-random-numbers-and-the-one-time-pad/examination-review-stream-ciphers-random-numbers-and-the-one-time-pad/what-are-the-key-differences-between-true-random-number-generators-trngs-pseudorandom-number-generators-prngs-and-cryptographically-secure-pseudorandom-number-generators-csprngs/" target="_blank" rel="noopener noreferrer">here</a>!

## Why this is vulnerable

```
tVar2 = time(NULL);
local_40 = (uint)tVar2;
srand(local_40);
```

Here the seed value to generate the sequence of random numbers is time, which does not require any guessing. It is acceptable for a game to use time as a seed .

> The flag is encryted by XORing the original flag with random number sequence (seed value is time).

# Solution
## Extracting timestamp
Most important you could ask me is, how are we going to know the exact time this flag was encoded (as it was used as seed). This answer also lies in the source code.

```
tVar2 = time(NULL);
local_40 = (uint)tVar2;
srand(local_40);
...
fwrite(&local_40, 1, 4, local_18);
```

This means:
- The first 4 bytes of flag.enc are the Unix timestamp
- That timestamp is the exact value passed to srand()
- No guessing, brute-forcing, or approximation is required

## Encryption Logic Breakdown
Loop in the code:

```
iVar1 = rand();
byte ^= (byte)iVar1;
```

XOR is reversable.

```
local_3c = rand() & 7;
byte = (byte << local_3c) | (byte >> (8 - local_3c));
```

Then there is a bit rotation, which again is reversable, either with the knowledge of shift count or by brute-force.

## Solution Strategy
That timestamp is the seed.

If we:
1. Extract the timestamp
2. Call srand(seed)
3. Generate the same rand() values
4. Reverse the operations
We recover the original flag.

**Why Reversing Order Matters**

<u>Encryption order:</u>
1. XOR
2. Rotate left

<u>Decryption order:</u>
1. Rotate right
2. XOR

**Reversing the sequence is essential because these operations are not commutative.**

## Solution code

```
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
uint8_t ror(uint8_t val, int shift) {
    shift &= 7; // Ensure shift is between 0-7
    return (val >> shift) | (val << (8 - shift));
}

int main() {
    FILE *fp;
    uint32_t seed;
    long file_size;
    uint8_t *data;

    // 1. Open the encrypted file
    fp = fopen("flag.enc", "rb");
    if (fp == NULL) {
        perror("Failed to open flag.enc");
        return 1;
    }

    // 2. Read the seed (first 4 bytes)
    // The decompilation shows: fwrite(&local_40, 1, 4, local_18);
    // local_40 was the result of time(), cast to uint.
    fread(&seed, 4, 1, fp);
    printf("Recovered Seed: %u\n", seed);

    // 3. Determine the size of the encrypted flag
    fseek(fp, 0, SEEK_END);
    long total_size = ftell(fp);
    file_size = total_size - 4; // Minus 4 bytes for the seed
    fseek(fp, 4, SEEK_SET);     // Skip the seed to read data

    // 4. Read the encrypted bytes
    data = malloc(file_size);
    fread(data, 1, file_size, fp);
    fclose(fp);

    // 5. Seed the RNG
    // This resets the internal state to exactly what it was during encryption.
    srand(seed);

    // 6. Decrypt loop
    printf("Decrypted Flag: ");
    for (int i = 0; i < file_size; i++) {
        // We must call rand() in the EXACT same order the encryption did.
        
        // Call 1: The value used for XOR
        int rnd_xor = rand();
        
        // Call 2: The value used for Bitwise Rotation
        int rnd_shift_raw = rand();
        int shift = rnd_shift_raw & 7;

        // --- REVERSE THE OPERATIONS ---
        
        // Step 1: Reverse the Rotate Left (by doing Rotate Right)
        uint8_t temp = ror(data[i], shift);
        
        // Step 2: Reverse the XOR (XOR is its own inverse)
        // Note: We cast rnd_xor to byte just like the original code did
        uint8_t plain_char = temp ^ (uint8_t)rnd_xor;

        printf("%c", plain_char);
    }
    printf("\n");

    free(data);
    return 0;
}
```

# Why the Solution Works

- rand() produces the same sequence for the same seed
- The seed is leaked
- Each encryption step is mathematically invertible
- No entropy is added during encryption

This allows perfect reconstruction of the original plaintext.
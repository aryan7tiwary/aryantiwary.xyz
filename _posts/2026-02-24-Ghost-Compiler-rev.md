---
toc: true
toc_sticky: true
classes: wide 
title: "Ghost Compiler: Rotating XOR and Runtime Secrets"
date: 2026-02-23T12:20:30-05:30
categories:
  - writeup
tags:
  - BITSCTF
  - reverse_engineering
  - ctf
  - ghidra
  - gdb
  - cryptography
  - linux
  - pie

---

> In this post, I’ll walk through how I analyzed a PIE-enabled ELF binary, mapped static offsets to runtime addresses, extracted a rotating XOR key in GDB, and rebuilt the decryption logic in Python. All steps were performed on a Linux environment using Ghidra and GDB.

There’s something oddly satisfying about reversing a binary that pretends to be something else. This one called itself a compiler. Looked harmless. Acted quiet. But the moment I ran it, I knew it wasn’t here to compile anything.

It was hiding a flag.

---
## Challenge Setup

- Platform: BITSCTF  
- Binary: `safe_ghost`  
- Category: Reverse Engineering  

We’re given a stripped ELF binary. No symbols. No obvious strings. PIE enabled.

Classic setup.

First instinct? Static analysis.

---
## Static Analysis in Ghidra
### Encryption Function

After loading the binary into Ghidra and letting auto-analysis complete, one function stood out:

**FUN_00101583:**

Located at offset: 0x1583

That offset turned out to be the key anchor for everything.

Here’s the decompiled function:

```
undefined8 FUN_00101583(long param_1,long param_2,ulong param_3)

{
  undefined8 uVar1;
  long in_FS_OFFSET;
  int local_24;
  ulong local_20;
  byte local_18 [8];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_20 = param_3;

  for (local_24 = 0; local_24 < 8; local_24 = local_24 + 1) {
    local_18[local_24] = (byte)local_20 ^ *(byte *)(param_1 + param_2 + local_24);
    local_20 = local_20 >> 1 | (ulong)((local_20 & 1) != 0) << 0x3f;
  }

  if (((((local_18[0] == 0x42) && (local_18[1] == 'I')) && (local_18[2] == 'T')) &&
      ((local_18[3] == 'S' && (local_18[4] == 'C')))) &&
     ((local_18[5] == 'T' && ((local_18[6] == 'F' && (local_18[7] == '{')))))) {
    uVar1 = 1;
  }
  else {
    uVar1 = 0;
  }

  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
    __stack_chk_fail();
  }

  return uVar1;
}
```

Let’s pause.

The condition checks:
```
BITSCTF{
```

So the function decrypts 8 bytes and verifies the flag prefix.

That means:

```
param_1 + param_2 → pointer to encrypted data
```

```
param_3 → 64-bit key
```

```
Loop → decrypt first 8 bytes
```

Already we can see the pattern.

---

### High-Level Encryption Logic

Inside the loop:
```
local_18[i] = (byte)local_20 ^ *(byte *)(param_1 + param_2 + i);
local_20 = local_20 >> 1 | (ulong)((local_20 & 1) != 0) << 0x3f;
```

Translated:
1. XOR ciphertext byte with lowest byte of 64-bit key.
2. Rotate the key right by 1 bit (64-bit ROR).

So effictively:
```
plaintext[i] = ciphertext[i] ^ (key & 0xFF)
key = ROR(key, 1)
```

That’s a rotating XOR stream cipher.
Simple. Deterministic. Reversible.
And importantly, the key is evolving each iteration.

---

## Dynamic Analysis

### Converting offset to runtime address

Because PIE is enabled, static addresses don’t match runtime ones.

So I launched GDB:
```
gdb ./safe_ghost
```

Started at entry:
```
starti
```

Checked mappings:
```
info proc mappings
```

Output:
```
0x0000555555554000  r--p
0x0000555555555000  r-xp

Base Address: 0x555555554000
Runtime Function Address: 0x555555554000 + 0x1583 = 0x555555555583
```

Now we set breakpoint:
```
b *0x555555555583
continue
```

And execution stopped exactly inside `FUN_00101583`.

Static and dynamic analysis aligned perfectly.

---

### Extracting runtime key
According to System V AMD64 calling convention:

* `param_1` → RDI

* `param_2` → RSI

* `param_3` → RDX

At the breakpoint:
```
print/x $rdx
```

Output:
```
0x5145dd89c16375d8
```

This is out 64-bit key.

Then I dumped encrypted bytes:
```
x/64bx $rdi + $rsi
```

Which produced:
```
encrypted_data = [
    0x9a, 0xa5, 0x22, 0xe8, 0x1e, 0xfa, 0x91, 0x90,
    0x1b, 0x8e, 0xb3, 0x5e, 0x5a, 0x2a, 0xf9, 0xf5,
    ...
]
```

At this point, everything needed was extracted from runtime:

* Initial key
* Ciphertext

* Decryption logic

Now it’s just replication.

---

## Solution

Decryption Script:
```
def decrypt_flag(encrypted_bytes, initial_key):
    flag = ""
    key = initial_key
    
    for byte in encrypted_bytes:
        lowest_key_byte = key & 0xFF
        decrypted_char = chr(byte ^ lowest_key_byte)
        
        if decrypted_char == '\x00':
            break
            
        flag += decrypted_char
        
        # 64-bit rotate right
        lsb = key & 1
        key = (key >> 1) | (lsb << 63)
        
    return flag


extracted_key = 0x5145dd89c16375d8

print("Flag:", decrypt_flag(encrypted_data, extracted_key))
```

And the full flag printed cleanly.
No brute force. No guessing.
Just deterministic reconstruction.

---

## Conclusion

### Weaknesses

Let’s summarize the weaknesses:

* XOR is reversible.
* Bit rotation is reversible.
* The key exists in memory at runtime.
* No additional entropy is added.

Once the key is exposed, the entire scheme collapses.

---

### Noteworthy

The function includes:
```
local_10 = *(long *)(in_FS_OFFSET + 0x28);
```

That’s stack canary logic.

So:

1. Stack protection enabled.
2. Proper compiler flags used.
3. Not a sloppy binary.

It wasn’t poorly built. It was just logically reversible.
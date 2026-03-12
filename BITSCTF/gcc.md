# Ghost C Compiler - CTF Writeup

## Challenge
Category: Reverse Engineering 

A "safe-to-use" C compiler wrapper that claims to match GCC's runtime speed. The binary processes C source files and compiles them into executables.

**Given:**
- `ghost_compiler` — A stripped Linux ELF executable.
- `chall.zip` — The challenge archive containing the clean binary.
- `README.md` — Basic instructions and the flag format.

## Analysis

Initial dynamic analysis reveals that `ghost_compiler` doesn't actually sanitize or compile code itself; it simply acts as a wrapper that passes arguments directly to the real system `gcc` via a `system()` call.

However, the critical twist lies in the binary's self-destruct mechanism. When executed, the decompiled `main` function performs the following sequence:
1. Loads its own binary into memory.
2. Locates a specific 64-byte payload inside itself.
3. Executes a decryption verification function.
4. Wipes the 64-byte payload in memory using `memset(..., 0, 0x40)`.
5. Overwrites the original executable on the disk with this wiped version.

Because executing the file permanently destroys the embedded flag, the challenge must be solved entirely offline using static analysis against a fresh, un-run binary.

## Solution Approach

### Strategy: Static Analysis + Custom Offline Decryption

Since the binary self-modifies, we must extract a pristine copy from `chall.zip` and reverse engineer the encryption logic:

1. **Locate the Payload:** The binary searches itself for an 8-byte magic header (`9A A5 22 E8 1E FA 91 90`) to find the exact offset of the 64-byte encrypted flag.
2. **Generate the Key:** The binary calculates a 64-bit master key by reading its own file byte-by-byte.  It uses the FNV-1a hashing algorithm (identified by the prime `0x100000001b3` and offset `0xcbf29ce484222325`), intentionally skipping the 64-byte payload during calculation. The final hash is XORed with `0xcafebabe00000000`.
3. **Decrypt the Flag:** The algorithm loops through the payload, XORing each encrypted byte with the lowest byte of the master key. After each byte, the 64-bit key is rotated right by 1 bit.

### Solver Code

We can emulate this logic in Python to calculate the file's hash, generate the key, and decrypt the payload offline. Save this as `solve.py` and run it in the same directory as a fresh `ghost_compiler` binary.

```python
def solve():
    with open("ghost_compiler", "rb") as f:
        data = f.read()

    # The magic bytes indicating the start of the payload
    magic = bytes([0x9A, 0xA5, 0x22, 0xE8, 0x1E, 0xFA, 0x91, 0x90])
    offset = data.find(magic)

    if offset == -1:
        print("[-] Payload not found. Use a fresh binary.")
        return

    print(f"[+] Encrypted payload found at offset: {hex(offset)}")

    # FNV-1a Hash generation
    hash_val = 0xcbf29ce484222325
    for i in range(len(data)):
        if offset <= i < offset + 64:
            continue
        hash_val = hash_val ^ data[i]
        hash_val = (hash_val * 0x100000001b3) & 0xFFFFFFFFFFFFFFFF

    # XOR with magic constant to derive master key
    master_key = hash_val ^ 0xcafebabe00000000
    print(f"[+] Master Key Calculated: {hex(master_key)}")
    
    # Decrypt payload
    payload = data[offset : offset + 64]
    flag = ""

    for i in range(64):
        decrypted_byte = payload[i] ^ (master_key & 0xFF)
        if decrypted_byte == 0:
            break
        flag += chr(decrypted_byte)
        # Rotate right by 1
        master_key = ((master_key >> 1) | ((master_key & 1) << 63)) & 0xFFFFFFFFFFFFFFFF

    print(f"[+] Decrypted Flag: {flag}")

if __name__ == "__main__":
    solve()
```

## Result

Running the Python script against a pristine copy of the executable successfully calculates the FNV-1a hash and decrypts the hidden payload without triggering the self-deletion trap.

```text
[+] Encrypted payload found at offset: 0x4020
[+] Master Key Calculated: [REDACTED_HASH]
[+] Decrypted Flag:BITSCTF{n4n0m1t3s_4nd_s3lf_d3struct_0ur0b0r0s}}
```

## Flag

`BITSCTF{n4n0m1t3s_4nd_s3lf_d3struct_0ur0b0r0s}`
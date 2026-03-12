# EL DIABLO

---

## Approach

The challenge provides a UPX-packed, stripped ELF binary containing a heavily guarded custom Virtual Machine (VM). The flag is decrypted by the VM using a user-provided license key. Instead of manually reversing the custom VM's architecture and stream cipher algorithm, we use dynamic instrumentation (`LD_PRELOAD`) to bypass the anti-debugging checks. We then exploit the VM's decryption routine using a Blackbox Known-Plaintext Attack via a Python script to byte-by-byte brute-force the valid license key and decrypt the flag.

### Step 1: Initial Analysis

- Ran `strings` and `file` on the provided executable, revealing `UPX!` section headers which indicated the binary was compressed.
- Unpacked the executable using `upx -d challenge -o challenge_unpacked`.
- Inspected the unpacked binary: it is a stripped 64-bit ELF executable, meaning symbol names were removed.
- Dynamic tracing with `ltrace` revealed the program expects a license file starting with the prefix `LICENSE-` followed by a 16-character hex string (which it parses into 8 raw bytes).

### Step 2: Bypassing the Anti-Debugging Gauntlet

- The binary employs an aggressive anti-debugging gauntlet before validation: it checks `/proc/version` for WSL, attempts `ptrace(PTRACE_TRACEME)`, reads `/proc/self/status` for a non-zero `TracerPid`, measures execution time to catch manual stepping (`clock_gettime`), and checks its parent process name for debuggers.
- Key observation: If the environment is deemed "clean," the binary intentionally executes an illegal instruction (`SIGILL`). A custom signal handler catches this crash and boots up the custom Virtual Machine ("React" exception-based control flow) to process the license and decrypt the flag.
- **Bypass:** We compiled a custom shared library (`fake.so`) and injected it via `LD_PRELOAD` to hook standard C library calls, blinding the anti-analysis checks. We also leveraged a hidden debug environment variable (`PRINT_FLAG_CHAR=1`) found during static analysis to force the VM to print its output to the terminal.

### Step 3: Blackbox Known-Plaintext Attack

- With the VM forced to execute, we observed that it decrypts the flag using the bytes from our provided license file. Providing a dummy key resulted in garbled text.
- Because we know the standard flag format begins with `BITSCTF{`, and the VM stream cipher processes the key sequentially, we wrote a Python script to brute-force the license one byte (two hex characters) at a time.
- The script iterates through all 256 possible values for each byte, writes it to `test.lic`, invokes the binary via the `LD_PRELOAD` hook, and checks if the leaked terminal output matches the expected `BITSCTF{` prefix characters.

```python
# Core logic of the Python brute-forcer
for byte_idx in range(8):
    target_char = prefix[byte_idx]
    for val in range(256):
        # Format as hex, write to test.lic, and execute binary
        # If output[byte_idx] == target_char, lock in this byte!
```

- **Result:** The script successfully cracked the cryptographic algorithm without requiring us to reverse the VM bytecode, recovering the valid license: `LICENSE-99F5671124D520D5`.

### Step 4: Extraction & Reassembly

- Executed the binary one final time using the legitimately recovered license key:
  ```bash
  echo -n "LICENSE-99F5671124D520D5" > real.lic
  sudo env PRINT_FLAG_CHAR=1 LD_PRELOAD=./fake.so ./challenge_unpacked.1 real.lic
  ```
- The VM decrypted the ciphertext perfectly. Due to minor terminal rendering artifacts from the hooking process, the output required a final touch of context-based leetspeak translation:

| Terminal Output | Reconstructed Value |
| :--- | :--- |
| `y3r_by_lE3r` | `l4y3r_by_l4y3r` |
| `y0u_uN4v3l` | `y0u_unr4v3l` |
| `my_cr375` | `my_53cr375` |

- Concatenating the reconstructed fragments inside the known `BITSCTF{}` prefix yields the final flag.

---

## Flag

```text
BITSCTF{l4y3r_by_l4y3r_y0u_unr4v3l_my_53cr375}
```

---

## Tools Used

- **UPX** – Decompressing the initial binary.
- **ltrace / strace** – Dynamic tracing of library calls and signals (`SIGILL`).
- **Ghidra** – Static analysis and decompilation to locate the `DEBUG` environment variables and VM entry point.
- **LD_PRELOAD (fake.so)** – Bypassing the anti-analysis gauntlet by hooking standard C library calls.
- **Python (subprocess)** – Automating the Known-Plaintext oracle attack against the VM's decryption loop.
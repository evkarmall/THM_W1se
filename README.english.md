# THM_W1seGuy

**TryHackMe XOR Challenge**

XOR vulnerability analysis.

---

# XOR Cryptanalysis Challenge — Write-up

This repository contains the solution to a cryptography challenge based on a **Known Plaintext Attack (KPA)** applied to an **insecure XOR implementation**.

The goal of the challenge is to exploit a cryptographic weakness in a network service that uses a **short, repeating key** to obfuscate sensitive information.

---

## Challenge Description

The server (`server.py`) runs on port **1337** and follows this workflow:

1. Generates a random **5-character alphanumeric key**
2. Applies an **XOR operation** between a fake flag (`THM{thisisafakeflag}`) and the key
3. Sends the XOR result encoded in **hexadecimal** to the client
4. Prompts the user for the encryption key

   * If the correct key is provided, the server returns the **real flag** stored in `flag.txt`

---

## Technologies Used

* **Python 3** — Brute-force automation and cryptographic analysis
* **Pwntools** — Library for interacting with network services
* **XOR Cryptography** — Reversible symmetric cipher

---

## Exploitation Logic

The vulnerability lies in the mathematical properties of the XOR operation.

Given a **Known Plaintext Attack (KPA)** scenario:

* We have the **Ciphertext (C)**
* We know part of the **Plaintext (P)**
  (all flags follow the format `THM{`)

We can recover the **Key (K)** using:

[
P \oplus K = C \Rightarrow K = P \oplus C
]

By XORing the **first 4 bytes** of the received hexadecimal ciphertext with the known plaintext `THM{`, we can recover the **first 4 characters of the key**.

Since the key length is exactly **5 characters**, the attack is reduced to a **controlled brute-force** over the final character only.

---

## Solution Script (`xor.py`)

```python
import string
from pwn import *

# Hexadecimal ciphertext received from the server
enc_flag = bytes.fromhex(
    "0231271f276718060a2313011e2523224d090f3417171857363a35130c02240d135422240125162a"
)

known_plaintext = b'THM{'

# Recover the first 4 bytes of the key
partial_key = xor(enc_flag, known_plaintext)[:4]

# Brute-force the 5th character of the key
charset = string.ascii_letters + string.digits
for c in charset:
    key = partial_key + c.encode()
    decrypted_flag = xor(enc_flag, key).decode()

    if decrypted_flag.endswith('}'):
        print(f"Recovered Key: {key.decode()}")
        print(f"Decrypted Flag: {decrypted_flag}")
```



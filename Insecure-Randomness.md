# Insecure Randomness
## What Is It?
Web applications rely on random values for session tokens, password reset links, CSRF tokens, and cryptographic keys. When these values are predictable or poorly generated, attackers can guess or reproduce them — turning a security mechanism into a vulnerability.

---

## Types of Random Number Generators

1. Pseudo-Random Number Generators (PRNGs)
These use a mathematical formula seeded with an initial value. Given the same seed, they always produce the same sequence.

Example: Python's ```random.random()``` — fine for simulations, dangerous for security tokens.

2. Cryptographically Secure PRNGs (CSPRNGs)
These draw from unpredictable system entropy and are designed for security purposes.

Example: ```secrets.token_hex(32)``` in Python or ```crypto.randomBytes()``` in Node.js.

--- 

## Core Vulnerability Types
Weak or Insufficient Randomness
Using low-entropy sources means the output space is small and brute-forceable.

```python 
# BAD – only 32,768 possible values
import random
token = random.randint(0, 32767)
```
```python 
# GOOD – astronomically large output space
import secrets
token = secrets.token_hex(32)
```

--- 

## Predictable Seeds During Token Generation
If the seed is derived from something guessable — like the current timestamp — an attacker who knows roughly when a token was generated can reproduce the entire sequence.

```python 
# BAD – seed is the current time (guessable within seconds)
import random, time
random.seed(int(time.time()))
reset_token = random.randint(100000, 999999)
```
```
Attack scenario: A password reset link is generated at a known time. The attacker seeds their own PRNG with nearby timestamps, generates all possible tokens, and tries each one — potentially hijacking the account reset.
```
## Session Hijacking via Predictable Session IDs
If session IDs follow a pattern (e.g., sequential or time-based), an attacker can enumerate valid sessions.

```bash
Session IDs observed:
  sess_10423
  sess_10424
  sess_10425   ← attacker guesses sess_10426 and hijacks it
  ```

  ## Cryptographic Key Weakness
Using a weak seed for key generation makes encryption breakable.

```javascript
// BAD – Math.random() is not cryptographically secure
const key = Math.random().toString(36);

// GOOD
const key = crypto.randomBytes(32).toString('hex');
```
## Key Takeaway
Randomness is only secure if it is unpredictable and has a large enough space that brute force is infeasible. Any shortcut — reusing seeds, using time as entropy, or relying on non-cryptographic PRNGs — creates an exploitable window for attackers.

--- 

## Randomness in Security
Randomness
Randomness is the absence of pattern or predictability in data. In secure systems, it ensures that generated values — keys, tokens, nonces — cannot be guessed or reproduced by an attacker.

### Type          |  How It Works                            |   Example Source
---
TRNG (True RNG)   |  Harvests physical, unpredictable events  | Mouse movement, CPU thermal noise, hardware chips

---
PRNG (Pseudo RNG) | Mathematical formula from a starting seed |  ```random.random()```, ```Math.random()```.


```
TRNGs are inherently unpredictable. PRNGs look random but are fully deterministic — same seed always yields same output.
```

## Entropy
Entropy measures the amount of unpredictability in a system. Think of it as the "quality" of randomness.

High entropy → large, unpredictable output space → hard to brute-force
Low entropy → small, guessable output space → easy to attack

--- 

```python
# Low entropy – only 4 digits, 10,000 possible values
token = str(random.randint(0, 9999))   # attacker brute-forces in seconds

# High entropy – 256 bits of randomness, 2²⁵⁶ possible values
token = secrets.token_hex(32)          # practically unguessable
```

```
A token derived from a timestamp has very low entropy — the attacker already knows roughly what the seed was.
```

## Cryptographic Keys
Keys are secret values fed into encryption algorithms. Their security depends on two things: length and randomness.

```python 
# BAD – short, low-entropy key
key = "password123"

# BAD – generated with weak PRNG
import random
key = str(random.getrandbits(64))

# GOOD – cryptographically secure, high entropy
from cryptography.hazmat.primitives.keygen import generate_key
key = os.urandom(32)   # 256 bits from OS entropy pool
```

```
If a key is predictable, an attacker can reproduce it and decrypt all protected data — making encryption useless regardless of algorithm strength.
```

## Session Tokens & Unique Identifiers
Session tokens track authenticated users between requests. They must be unpredictable and unique — otherwise attackers can guess valid sessions.

```bash
# BAD – sequential token (predictable pattern)
user_1 → sess_1001
user_2 → sess_1002
user_3 → sess_1003   ← attacker guesses sess_1004 and hijacks next session

# GOOD – cryptographically random token
user_1 → a3f8c92d1b...  (128 bits, no pattern)
user_2 → 7e41bc039f...  (completely unrelated to user_1's token)
```

```python 
# GOOD session token generation
import secrets
session_token = secrets.token_urlsafe(32)
```

## Seeding
A seed is the initial value fed into a PRNG. The entire output sequence is mathematically derived from it — making the seed the single point of failure.

```python 
import random

# Same seed → identical sequence every time
random.seed(42)
print(random.randint(0, 1000))   # always outputs: 654

random.seed(42)
print(random.randint(0, 1000))   # still outputs: 654
```

## Real-world attack scenario:

```
1. App generates a password reset token seeded with current Unix timestamp
2. Attacker requests a reset and notes the exact time (e.g., 1713744000)
3. Attacker seeds their own PRNG with timestamps ±5 seconds
4. Attacker generates all possible token sequences
5. Attacker tries each token → account compromised
```

The fix is to never use guessable values (time, user ID, IP address) as seeds. Use OS-level entropy instead: ```os.urandom()```, ```/dev/urandom```, or a hardware RNG.

## How It All Connects
```
Low Entropy Seed
      ↓
Predictable PRNG Output
      ↓
Weak Session Token / Crypto Key
      ↓
Brute Force / Prediction Attack
      ↓
Authentication Bypass / Data Decryption
```
The entire chain collapses if any one element — seed, entropy source, or generator — is weak. Secure randomness requires high entropy at the source, a CSPRNG for generation, and sufficient output length to make guessing infeasible.

--- 

## Types of RNGs
True Random Number Generator (TRNG)
TRNGs derive randomness from unpredictable physical phenomena — events that are genuinely impossible to predict or reproduce, making them the gold standard for cryptographic security.

### Source               |           What's Measured

Thermal noise        |     Random electron movement in resistors

Radioactive decay    |      Unpredictable atomic disintegration timing

Atmospheric noise    |      Radio static from electromagnetic interference

Photon behavior      |      Quantum-level light particle behavior

Hardware events      |      CPU timing jitter, disk seek times

---

## How It Works
 
```bash
Physical Event (e.g., thermal noise)
        ↓
Analog signal captured by hardware sensor
        ↓
Converted to digital bits
        ↓
True random output  →  e.g., cryptographic key
```

No mathematical formula is involved — the randomness comes directly from nature, so there is no seed, no pattern, no reproducibility.

## Primary Use Cases
1. Cryptographic Key Generation
Algorithms like RSA and AES require keys that are absolutely unpredictable. A TRNG ensures no attacker can reproduce the key generation process.

```bash 
RSA-2048 key pair generation:
  → Requires ~2048 truly random bits
  → TRNG feeds raw entropy into key generation algorithm
  → Even with full knowledge of the system, key cannot be predicted
  ```

  2. Digital Signatures
Signing requires a fresh random value (nonce) each time. If this value repeats or is predictable, the private key can be mathematically extracted.

```bash 
# Real-world consequence of weak nonce in ECDSA signing:
# Sony PlayStation 3 (2010) used a FIXED nonce for all signatures
# → Private signing key was fully recovered by attackers
# → Entire code-signing system broken
```
 3. Certificate Creation (TLS/SSL)
Certificate authorities use TRNGs when generating the keys embedded in HTTPS certificates, ensuring each certificate's private key is unique and unguessable.

## Advantages vs. Limitations

```
✅ Genuinely unpredictable — not reproducible even with full system knowledge
✅ Highest entropy quality available
✅ No seed dependency

❌ Requires specialised hardware (HSMs, hardware RNG chips)
❌ Slower generation speed — entropy must be "collected" from physical events
❌ Not suitable for high-volume, rapid number generation
❌ More expensive to deploy at scale
```

## Real-World Hardware Examples

```
Intel RDRAND instruction  →  built-in CPU hardware RNG using thermal noise
/dev/random (Linux)       →  collects entropy from hardware events (keystrokes, 
                              interrupts, disk I/O)
HSM (Hardware Security    →  dedicated device for key generation in banks,
Module)                       certificate authorities, and governments
Cloudflare's lava lamps   →  cameras filming 100 lava lamps feed visual entropy 
                              into their RNG system
```

## TRNG vs. PRNG at a Glance

```
TRNG:   Physical world → unpredictable bits → no seed → not reproducible
PRNG:   Seed value → mathematical formula → predictable sequence → reproducible
```
TRNGs are the foundation of trust in high-stakes cryptography. However, because they are slow and hardware-dependent, most systems use a hybrid approach — a TRNG seeds a CSPRNG, combining true unpredictability with fast, scalable output.


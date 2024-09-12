
# Challenge: Trapdoor

## Description
*"You'll need more than 2700 core-years to break this encryption!"*

The challenge provided two files containing encrypted messages (`msg1.enc` and `msg2.enc`), along with two public keys (`public_key1` and `public_key2`). The goal was to decrypt the messages and retrieve the flag.

## Files Provided:
- `msg1.enc`: First encrypted message.
- `msg2.enc`: Second encrypted message.
- `public_key1`: RSA public key with modulus and exponent.
- `public_key2`: RSA public key with modulus and exponent.

---

## Solution Walkthrough

### Step 1: Analyzing the Public Keys

The first step was to inspect the provided public keys, both of which were in a standard RSA format:

- **`public_key1`**:
    ```plaintext
    e = 65537
    n = 84933534129088668530546921839185770773128434095819041907018584313601015770942905361042
    ```

- **`public_key2`**:
    ```plaintext
    e = 65537
    n = 60352356653033384917834042849894490266636374587383612801904610496885016078855962685911
    ```

The modulus `n1` and `n2` were very large, as expected in RSA encryption. However, when two moduli share a common prime factor, it is possible to use the **common modulus factorization** method to break the encryption.

### Step 2: Calculating the Greatest Common Divisor (GCD)

To check if the moduli shared any common factors, I computed the greatest common divisor (GCD) of the two moduli `n1` and `n2`. The result was a large integer, indicating that the two keys shared a common prime factor.

```python
from math import gcd

n1 = 84933534129088668530546921839185770773128434095819041907018584313601015770942905361042
n2 = 60352356653033384917834042849894490266636374587383612801904610496885016078855962685911

gcd_n1_n2 = gcd(n1, n2)
print(gcd_n1_n2)
```

### Step 3: Factoring the Moduli

The GCD result revealed that the two moduli shared a common prime factor, `p`. The other two factors for `n1` and `n2` were `q1` and `q2`, respectively. Using this, I factored both moduli:

- **Common Prime Factor (p)**: A large prime shared by both moduli.
- **Prime Factor `q1`** for `public_key1`.
- **Prime Factor `q2`** for `public_key2`.

This allowed me to rewrite both moduli as:
- \( n1 = p 	imes q1 \)
- \( n2 = p 	imes q2 \)

### Step 4: Calculating the Private Keys

With the factored moduli, I calculated the private keys using the formula:

\[
d = e^{-1} \mod \phi(n)
\]
Where:
- \( e \) is the public exponent (65537).
- \( \phi(n) = (p-1)(q-1) \) is the Euler's Totient function.

The private keys `d1` and `d2` were computed for each public key:

```python
from sympy import mod_inverse

# Calculate phi for both keys
phi1 = (p - 1) * (q1 - 1)
phi2 = (p - 1) * (q2 - 1)

# Compute private exponents
d1 = mod_inverse(e, phi1)
d2 = mod_inverse(e, phi2)
```

### Step 5: Decrypting the Messages

Using the private keys, I decrypted the ciphertexts using the RSA decryption formula:

\[
m = c^d \mod n
\]

Where:
- `c` is the ciphertext.
- `d` is the private key.
- `n` is the modulus.

```python
# Decrypt the ciphertexts
m1 = pow(c1, d1, n1)
m2 = pow(c2, d2, n2)

# Convert the decrypted messages to readable format
decrypted_msg1 = m1.to_bytes((m1.bit_length() + 7) // 8, byteorder='big')
decrypted_msg2 = m2.to_bytes((m2.bit_length() + 7) // 8, byteorder='big')

print(decrypted_msg1)
print(decrypted_msg2)
```

After decrypting the messages, I obtained:

- **Decrypted Message 1**:
    ```plaintext
    The Euclidean algorithm is the granddaddy of all algorithms,
    because it is the oldest nontrivial algorithm that has
    survived to the present day.
    ```

- **Decrypted Message 2 (Flag)**:
    ```plaintext
    csawctf{sowwy_W3_b0u9ht_a_l34ky_tr4pd00r!}
    ```

---

## Conclusion

The vulnerability in this challenge stemmed from the fact that the two public keys shared a common prime factor. By using the GCD, I was able to factor both moduli, compute the private keys, and decrypt the messages. The flag was successfully retrieved from the second decrypted message.

**Flag:** `csawctf{sowwy_W3_b0u9ht_a_l34ky_tr4pd00r!}`

---

## Key Takeaways
1. **RSA Key Vulnerability**: RSA encryption is vulnerable when different keys share a common modulus or prime factor. This can allow attackers to break the encryption using GCD and factorization techniques.
2. **Importance of Unique Moduli**: When generating RSA keys, it is essential to ensure that moduli are unique and that no two keys share factors.

---


To calculate the ciphertext (c) in an RSA encryption scheme, you can use the following formula:

c ≡ m^e (mod n)

Where:

- c is the ciphertext
- m is the plaintext message
- e is the public exponent
- n is the modulus (n = p * q)

Given the values you provided:

- p = 11
- q = 17
- e = 3
- message (m) = 66

First, calculate n: n = p * q = 11 * 17 = 187

Now, calculate c: c ≡ 66^3 (mod 187)

```python
p = value_of_p
q = value_of_q
e = value_of_e      #e is the public exponent
m = character_of_m  #m is plaintext message

# Calculate n (modulus)
n = p * q

# Calculate c (ciphertext)
c = (m ** e) % n

print(c)

```
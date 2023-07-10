---
title: Private Key Cryptography
name: priv
---

TODO: basic game notions (also pub)

## Block Ciphers

Often, we wish to encrypt large amounts of data without having to generate new secret keys each time nor comprimising the security of our existing key. A **block cipher** is an encryption scheme that works on fixed-length **blocks** at a time. They are very useful for encrypting large or arbitrary-length data, such as messages or videos. **AES (Advanced Encryption Standard)** is one such block cipher. It was adopted by NIST in 2001 after winning a 5-year public competition to become the new standard secure block cipher. To use AES, simply provide it with a message and a suitable key; it will then output the ciphertext (the API for AES is quite simple despite its complexity). The inner workings of AES are extremely complicated and out of scope for this class.

[There are many modes in which a block cipher can operate](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation), but the one we'll be using is known as **cipher-block chaining mode**, or **CBC mode**. One nuance of this mode is that it requires each encryption to be initialized with an **initialization vector**, or IV for short, and each decryption should use the same IV when decrypting. The IV does not need to be kept secret, but it should be chosen at random.

## Message Integrity

An important aspect of communication is message integrity; we want to be sure that our messages haven't been tampered with in transit. A **MAC (Message Authentication Code)** is one way of cryptographically ensuring message integrity. A MAC generator takes in a shared secret and a message and outputs a tag for the message. A MAC verifier takes in the shared secret, a message, a MAC tag, and outputs "Verified" (or some other positive value) if and only if the MAC is valid for the given message. Otherwise, it rejects the tag, indicating that the value has been tampered with or that the MAC was generated incorrectly. It must be difficult for those without the shared secret to generate valid MACs for any message (otherwise, this wouldn't be secure). The MAC that is computed on some value can be thought of as a signature but with symmetric keys. In this assignment we use **HMAC (Hashed Message Authentication Code)**, a widely used MAC algorithm, to tag our messages.

We need to be careful about the order in which we apply our cryptographic primitives. In particular, do we compute MACs on the plaintext and then encrypt the message along with the tag, or do we encrypt our plaintext first and then compute MAC on the resulting ciphertext? It turns out that only latter approach is secure.

## Key Derivation

The key we generate from the Diffie-Hellman key exchange may not be sufficient for AES or HMAC. For example, it may not be long enough, it may be of the wrong size, or may have a distribution that doesn't preserve the security guarantees of AES. Moreover, we want to use different keys for AES and HMAC to preserve security. To this end, we use a **secure key derivation function** to convert a shared secret into an acceptable key. In this assignment, we use **HKDF (HMAC-based Key Derivation Function)**, a widely used key derivation function, to generate secure keys. To ensure that HKDF generates different keys for AES and HMAC, we **salt** the Diffie-Hellman shared secret in the HKDF calls. Salts have been provided for you already in the stencil code, and should be passed into the HKDF call as a parameter. We will use HKDF two times in this assignment, once to generate a key for AES and once to generate a key for HMAC.

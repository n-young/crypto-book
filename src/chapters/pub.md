---
title: Public Key Cryptography
name: pub
---

# Background Knowledge

In this assignment, you'll build a secure authentication platform. There are two programs involved: a server and a client. The server acts as a central verification authority that clients will interact with to obtain certificates. The server has its own globally-recognized public key, and signs the certificates of clients who properly log in. The clients will then use these certificates to communicate with each other, protected against impersonation attacks. In order to verify with the server, each client must provide a valid password and 2FA response.

## Digital Signatures

You've interacted with digital signatures briefly in the first (warm-up) assignment, but we will go over them again here. Digital signatures are essentially the public-key equivalent to MACs, allowing one party to sign a message, and all other parties to verify that this message was signed by that party. To do so, we first generate a keypair consisting of a public verification key $vk$ and a secret signing key $sk$, then use $sk$ to sign various messages $\sigma_i = Sign_{sk}(m_i)$. When another party is given a message and signature, they can use $vk$ to verify that the message was signed correctly: $Verify_{vk}(\sigma_i, m_i) ?= true$. We want it to be the case that it is hard to forge signatures; that is, given $vk$ but not $sk$, finding valid signatures for any message, even when given valid signatures for other messages, is hard. 

Using digital signatures, we can achieve **authenticated key exchange** that is secure against man-in-the-middle attacks which we have seen in project Signal.

## Password Authentication

You probably authenticate by password every day. Password authentication relies on both a server and a user knowing some shared secret $pw$, and the user proves that they know this secret by sending $pw$ (or some altered version of it) to the server. With the cryptographic primitives we've explored thus far in mind, there are a number of naive ways that one might implement password authentication. One might encrypt the password and send it to the server, who will then decrypt it and store the plaintext password in a database for later verification. While encryption makes this protocol safe from eavesdropping attacks, storing the plaintext password doesn't protect against cases where the server or database are compromised. Even if we stored a hash of the password, adversaries that have access to the database can mount an offline brute-force dictionary attack or consult a [rainbow table](https://en.wikipedia.org/wiki/Rainbow_table) to crack the passwords We want to be careful to protect against a variety of attacks against all parts of our system.

We propose a heavily redundant but secure password authentication scheme so that you get a sense of the techniques you may see out in the wild. On registration, the server generates and sends a random (say, 128-bit) **salt** to the user. A salt is a random string appended to a password before hashing it to prevent dictionary attacks. The user sends the hash of their password with the salt appended: $h := H(pw || salt)$ to the server, which then computes a random short (say, 8-bit) **pepper** and hashes the user's message with the pepper appended yet again: $h' := H(h || pepper)$. Finally, the server stores the salt, but not the pepper. On verification, the server sends the stored salt to the user, who then sends the hash of their password. Then, the server tries all $2^8$ possible pepper values and verifies if any one of them succeeds. We avoid sending the password in plaintext form over the wire, and we avoid storing a version of the password that can be used for authentication in the database.

## Pseudorandom Functions and 2FA

Random numbers are convenient because they introduce a level of unpredictability to systems which can be very useful for keeping secret values secret (e.g. ElGamal encryption uses random values to ensure that even two ciphertexts of the same message are distinct). However, sometimes you want both you and your partner to experience the same randomness, or you may want to cheaply generate more randomness from some base *seed* of randomness. Pseudorandom functions (PRFs) are deterministic but unpredictable functions that take some value and output a pseudorandom value. We want that the distribution of PRF outputs to be indistinguishable from that of a truly random function.

PRFs are useful in many ways. For one, they allow you to securely generate an infinite amount of seemingly random values deterministically for use in other cryptographic protocols. In this assignment, we'll use a PRF to implement two-factor authentication by using PRF outputs as a way of proving that we know the value of given a shared seed $s$. We can generate a short-lived login token by inputting this seed alongside the current time. The server can then validate that our values are correct by running the same function. 

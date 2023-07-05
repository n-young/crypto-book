---
title: Public Key Cryptography
name: pub
---



## Diffie-Hellman Key Exchange

We now step away from number theory and consider some real cryptographic protocols. Let's say two parties, Alice and Bob (we typically name our honest parties Alice and Bob, and any adversaries Eve), want to decide on a shared key to encrypt some messages. For example, they may want to apply the [one-time pad](https://en.wikipedia.org/wiki/One-time_pad) and so they need a shared, secret $k$-bit integer to do so. The **[Diffie-Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)** protocol, developed in 1976, is one method of coming to a shared secret. Diffie-Hellman will be used *extensively* throughout the rest of the course to compute shared secrets.

Diffie-Hellman is quite simple. Alice and Bob first come to agreement on a cyclic group $\mathbb{G}$ (e.g., $\mathbb{Z}_p^*$ under multiplication for a large prime $p$) with a generator $g$.
In general, we wish to keep our groups large enough where an adversary can't brute-force their way into finding out the secrets. Alice and Bob then each pick a secret random integer from the range $[2, p - 2]$, denoted as $a, b$ respectively. Alice will compute and send $g^a$ to Bob, and Bob will compute and send $g^b$ to Alice. Finally, both parties will compute $g^{ab}$ by exponentiating what they receive from the other party with their secret integer. This value, $g^{ab}$, is the shared secret. (Remember that fast-powering is what makes this efficient; otherwise, computing large exponents will take a long time!)

![Architecture Diffie-Hellman](/static/img/handout/cipher/DH.png)

Correctness is clear since the operations clearly end up with the same values on both parties. What might not be clear is why this is secure; can an adversary Eve figure out $g^{ab}$ given what has been transmitted; namely, $g^a$ and $g^b$? In truth, we don't know whether Eve can efficiently solve this problem; the hardness of this problem is called the Diffie-Hellman assumption ([decisional](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange), [computational](https://en.wikipedia.org/wiki/Computational_Diffie%E2%80%93Hellman_assumption)), which inherently assumes that [Discrete logarithm](https://en.wikipedia.org/wiki/Discrete_logarithm) is computationally hard. For specific groups that we use in practice, all known (classical) algorithms take too long (longer than the age of the universe!) to break the Diffie-Hellman assumption.


## ElGamal Encryption

With Diffie-Hellman key exchange, we can come to a shared key in order to send a message. This is useful for **symmetric-key encryption**, which is where both Alice and Bob encrypt and decrypt using a shared key. However, what if we can't communicate to find a shared key in advance? We can instead rely on **assymmetric-key encryption**, also known as **public-key encryption**, which allows Alice to send messages to Bob, but not vice versa. In general, Bob will have some secret key $\mathrm{sk}$ and some public key $\mathrm{pk}$, and publish only $\mathrm{pk}$. Alice will then use $\mathrm{pk}$ to encrypt messages to Bob that can only be decrypted with $\mathrm{sk}$. We explore an example of such a system now. The **[ElGamal encryption](https://en.wikipedia.org/wiki/ElGamal_encryption)** scheme, developed in 1985, is such a public-key encryption scheme.  It is based on the Diffie-Hellman assumption and operates in a very similar way.

We begin by describing how Bob generates his public key, $\mathrm{pk}$, and secret key, $\mathrm{sk}$. First, Bob will generate a cyclic group $\mathbb{G}$ (e.g., $\mathbb{Z}_p^*$ under multiplication for a large prime $p$) with a generator $g$. He then chooses a random integer $x$ from the range $[1, p-1]$ and computes $g^x$. We then have that $\mathrm{pk} = g^x$ and $\mathrm{sk} = x$, so Bob publishes $\mathrm{pk}$.

When Alice wants to encrypt a message $m$, which can be any element in the group $\mathbb{G}$, she first chooses a random integer $y$ from the range $[1, p-1]$. Then, she computes $c_1 = g^y$ and $c_2 = m \cdot \mathrm{pk}^y$ and sends both to Bob. To decrypt, Bob computes $c_2 \cdot (c_1^{\mathrm{sk} })^{-1}$ (where inverses are computed using the Extended Euclidean Algorithm in $\mathbb{Z}_p^*$).  

![Architecture Elgalmal](/static/img/handout/cipher/Elg.png)

Correctness is simple when we expand; notice that $c_2 \cdot (c_1^{\mathrm{sk} })^{-1} = m \cdot g^{xy} \cdot g^{-xy} = m$; so Bob recovers the original message. Security of this scheme relies on the same assumptions as the Diffie-Hellman key exchange; we eschew a rigorous security proof in favor of a mathematical cryptography course.

## RSA Encryption

We explore one more public-key cryptosystem, known as **[RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)#Key_generation)** (Rivest-Shamir-Adleman). Unlike Diffie-Hellman key exchange and ElGamal encryption, RSA doesn't rely on the hardness of the discrete logarithm problem; rather, it relies on the hardness of factoring large integers. That is, given the product of two large primes $n = p \cdot q$, it is computationally hard to find $p$ and $q$.

We start with how Bob calculates his public and secret keys. First, Bob will generate system parameters by choosing two large primes $p, q$. He then computes $n = p \cdot q$, and $\phi(n) = \phi(p\cdot q) = (p-1)(q-1)$.
Bob then chooses some integer $e$ such that $\gcd(e, \phi(n)) = 1$. Lastly, since $e$ is coprime to $\phi(n)$, we can find some $d$ such that $e\cdot d \equiv 1 \mod \phi(n)$. We have then that $\mathrm{pk} = (n, e)$ and $\mathrm{sk} = d$, so Bob publishes $\mathrm{pk}$.

When Alice wants to encrypt a message $m$, which can be any integer in the range $[1, n-1]$, Alice simply computes $c = m^e \mod n$ and sends it to Bob. To decrypt, Bob computes $c^d \mod n$.

![Architecture RSA](/static/img/handout/cipher/RSA.png)

Correctness relies on Euler's theorem. We have that $e\cdot d \equiv 1 \mod \phi(n)$, or that $e \cdot d = k \cdot \phi(n) +1$ for some integer $k$. Then, $c^d \equiv m^{e\cdot d} \equiv m^{k \cdot \phi(n) + 1} \equiv m \mod n$ by Euler's theorem. Security relies on the assumption that factoring $n$ is computationally hard. Intuitively speaking, without factoring $n$, an adversary Eve cannot discover a suitable decryption exponent $d$, and is stuck; we eschew a rigorous security proof in favor of a mathematical cryptography course.

## DSA Signature

Orthogonal to the problem of message secrecy is message integrity. Let's say all of our channels are being controlled by an adversary Eve; how can we protect our messages from being tampered with? One solution to this problem is to **sign** our messages; by having the signature be difficult to compute without knowledge of a secret key, Eve will not be able to sign altered messages. We explore the [DSA Signature](https://en.wikipedia.org/wiki/Digital_Signature_Algorithm#1._Key_generation) scheme.

### Key generation

DSA is not very simple, but bear with us. Assume that Bob wants to sign messages to Alice. First, Bob chooses two primes $q, p$ such that $q \mid (p-1)$. Next, he chooses a random integer $h$ from the range $[2, p-2]$ and computes $g = h^{(p-1)/q} \mod  p$. Bob publishes $p, q, g$ as public parameters for verification. Bob generates a key pair by choosing a random integer $x$ from the range $[2, q-1]$ and computing $y = g^{x} \mod  p$. Here, $x$ is Bob's private signing key $\mathrm{sk}$, and $y$ is the public verification key $\mathrm{vk}$.

### Signing

When Bob wishes to sign a message $m$, which can be any integer, he first chooses a random integer $k$ from the range $[2, q-1]$ and computes $r = (g^k \mod  p) \mod  q$ (be very careful of the order of mods; it is important for correctness) and $s = (k^{-1} \cdot (m + xr)) \mod  q$. Bob outputs $(r, s)$ as his signature.

### Verification

When Alice wishes to verify that a message-signature pair, $m, (r, s)$ is valid, she computes $u_1 = m \cdot s^{-1} \mod q$, $u_2 = r \cdot s^{-1} \mod  q$, and finally, $v = (g^{u_1} \cdot \mathrm{vk}^{u_2} \mod  p) \mod  q$. She knows the signature is valid if and only if $v = r$.

![Architecture DSA](/static/img/handout/cipher/DSA.png)

### Correctness

Correctness is clear if expand carefully. First, notice that $g = h^{(p-1)/q} \mod  p \implies g^q \equiv h^{p-1} \equiv 1 \mod  p$ by FLT. We have from construction that $k = w \cdot m + w \cdot x \cdot r$ where $w = s^{-1} \mod  q$. Thus, $g^k \equiv g^{w \cdot m} g^{w \cdot x \cdot r} \equiv g^{u_1} y^{w\cdot r} \equiv g^{u_1} y^{u_2} \mod p$, hence $r = (g^k \mod  p) \mod  q = v$

Security of this scheme relies on the hardness of discrete logarithm; we eschew a rigorous security proof in favor of a mathematical cryptography course.

It's worth noting that often we want to compute signatures for messages that might fall outside of the acceptable range for this algorithm. To make sure that this is doable efficiently, we usually employ a hash function to shrink the size of a message for signing. An appropriate hash function should be chosen, however, as to not compromise security of the scheme.

## Digital Signatures

You've interacted with digital signatures briefly in the first (warm-up) assignment, but we will go over them again here. Digital signatures are essentially the public-key equivalent to MACs, allowing one party to sign a message, and all other parties to verify that this message was signed by that party. To do so, we first generate a keypair consisting of a public verification key $vk$ and a secret signing key $sk$, then use $sk$ to sign various messages $\sigma_i = Sign_{sk}(m_i)$. When another party is given a message and signature, they can use $vk$ to verify that the message was signed correctly: $Verify_{vk}(\sigma_i, m_i) ?= true$. We want it to be the case that it is hard to forge signatures; that is, given $vk$ but not $sk$, finding valid signatures for any message, even when given valid signatures for other messages, is hard. 

Using digital signatures, we can achieve **authenticated key exchange** that is secure against man-in-the-middle attacks which we have seen in project Signal.

## Password Authentication

You probably authenticate by password every day. Password authentication relies on both a server and a user knowing some shared secret $pw$, and the user proves that they know this secret by sending $pw$ (or some altered version of it) to the server. With the cryptographic primitives we've explored thus far in mind, there are a number of naive ways that one might implement password authentication. One might encrypt the password and send it to the server, who will then decrypt it and store the plaintext password in a database for later verification. While encryption makes this protocol safe from eavesdropping attacks, storing the plaintext password doesn't protect against cases where the server or database are compromised. Even if we stored a hash of the password, adversaries that have access to the database can mount an offline brute-force dictionary attack or consult a [rainbow table](https://en.wikipedia.org/wiki/Rainbow_table) to crack the passwords We want to be careful to protect against a variety of attacks against all parts of our system.

We propose a heavily redundant but secure password authentication scheme so that you get a sense of the techniques you may see out in the wild. On registration, the server generates and sends a random (say, 128-bit) **salt** to the user. A salt is a random string appended to a password before hashing it to prevent dictionary attacks. The user sends the hash of their password with the salt appended: $h := H(pw || salt)$ to the server, which then computes a random short (say, 8-bit) **pepper** and hashes the user's message with the pepper appended yet again: $h' := H(h || pepper)$. Finally, the server stores the salt, but not the pepper. On verification, the server sends the stored salt to the user, who then sends the hash of their password. Then, the server tries all $2^8$ possible pepper values and verifies if any one of them succeeds. We avoid sending the password in plaintext form over the wire, and we avoid storing a version of the password that can be used for authentication in the database.

## Pseudorandom Functions and 2FA

Random numbers are convenient because they introduce a level of unpredictability to systems which can be very useful for keeping secret values secret (e.g. ElGamal encryption uses random values to ensure that even two ciphertexts of the same message are distinct). However, sometimes you want both you and your partner to experience the same randomness, or you may want to cheaply generate more randomness from some base *seed* of randomness. Pseudorandom functions (PRFs) are deterministic but unpredictable functions that take some value and output a pseudorandom value. We want that the distribution of PRF outputs to be indistinguishable from that of a truly random function.

PRFs are useful in many ways. For one, they allow you to securely generate an infinite amount of seemingly random values deterministically for use in other cryptographic protocols. In this assignment, we'll use a PRF to implement two-factor authentication by using PRF outputs as a way of proving that we know the value of given a shared seed $s$. We can generate a short-lived login token by inputting this seed alongside the current time. The server can then validate that our values are correct by running the same function. 

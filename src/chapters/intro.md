---
title: Introduction
name: intro
---

# Background Knowledge

In this assignment you'll be implementing a few foundational cryptographic schemes. In order to fully understand why these schemes are correct and secure, we review some of the number theory underlying these constructions. Don't worry; the rest of the course won't rely on a deep understanding of the math behind these schemes. Critically, we don't go over any advanced or involved proofs in this handout or course; rather, we introduce the results that are useful and ask that you take them at face value. If you are interested in the number theory, we recommend reading [BS Appendix A] [MOV Chapters 2 & 3]. It is, however, crucial that you understand the purpose and use of each scheme, and knowing how they work under the hood can help you gain some of that understanding.


## Elementary Number Theory

The following is an overview of the number theory necessary to understand the cryptographic schemes in this homework. You can safely skip this section if you're already familiar with number theory, or if you're more comfortable engaging with the schemes directly (we'll use language and terminology from this section, but not deeply). **This section may seem intimidating; to reiterate, you do not need to understand this math deeply to implement this assignment, and you certainly don't need it for the rest of the course.**

### Divisibility and GCDs

Consider two integers $a,b\in\mathbb{Z}$. We say that $a$ **divides** $b$ if there exists an integer $c \in \mathbb{Z}$ such that $a \cdot c = b$. We denote this by $a \mid b$.

Given integers $a, b, m \in \mathbb{Z}$. We say that $a$ and $b$ are **congruent mod $m$** if there exists an integer $k\in\mathbb{Z}$ such that $a + km = b$. In other words, it means that $a$ and $b$ differ by a multiple of $m$, or that when divided by $m$, they yield the same remainder. We denote this by $a \equiv b \mod m$.

Recall **greatest common divisors (GCDs)**. Given two integers $a, b \in\mathbb{Z}$, the GCD of $a$ and $b$ is the largest integer $d\in\mathbb{Z}$ such that $d \mid a$ and $d \mid b$. We say that two integers are **coprime** if their GCD is 1. Calculating the GCD of two integers can be done efficiently using the [Euclidean Algorithm](https://en.wikipedia.org/wiki/Euclidean_algorithm), and calculating integers $s, t$ such that $s \cdot a + t \cdot b = \gcd(a, b)$ can be done efficiently using the [Extended Euclidean Algorithm](https://en.wikipedia.org/wiki/Extended_Euclidean_algorithm). We eschew a detailed explanation of either algorithm in favor of the Wikipedia articles.

### Groups

To work with some theorems more nicely in general, and to allow us to generalize some of the following schemes, we introduce the notion of a group. A **group** is defined as a set $\mathbb{G}$ along with a binary operation $\otimes: \mathbb{G} \times \mathbb{G} \rightarrow \mathbb{G}$, such that the following three properties hold:
1. **Identity**: there exists an identity element $e\in\mathbb{G}$ such that for any element $a \in \mathbb{G}$ we have that $e \otimes a = a \otimes e = a$.
2. **Associativity**: for any $a, b, c \in \mathbb{G}$, we have that $(a \otimes b) \otimes c = a \otimes (b \otimes c)$.
3. **Inverses**: for any $a \in \mathbb{G}$, there exists an inverse element $b \in \mathbb{G}$ such that $a \otimes b = b\otimes a = e$. We denote the inverse of $a$ by $a^{-1}$.

For example, $\mathbb{Z}$ under addition (where our operation $\otimes$ is $+$) is a group. It has identity $0$, addition is associative, and every integer $n$ has $-n$ as its inverse. An example of a *non-group* could be $\mathbb{Z}$ under multiplication. While it has an identity element $1$ and is associative, not every element has a multiplicative inverse. However, if we change our set slightly to be the non-zero rational numbers $\mathbb{Q}^{\neq 0}$ with multiplication, this again is a group. 

The set of integers modulo a prime $p$ (excluding $0$) under *multiplication* is also a group, denoted as $\mathbb{Z}_p^* = \{1,2,\dots,p-1\}$. More generally, if you consider the integers from $[1,m-1]$ that are coprime to $m$, then we can construct a special group called the **multiplicative group of units**, denoted as $\mathbb{Z}_m^*=\{a \mid a \in [1,m-1], \gcd(a,m)=1\}$.

Given that $\gcd(a, m) = 1$, finding an inverse is as simple as running the Extended Euclidean Algorithm. Taking the relation $s\cdot a + t\cdot m = 1 \mod  m$, we get $s\cdot a \equiv 1 \mod  m$ where $s$ is the inverse of $a$. If $m$ is prime, then $\gcd(a, m) = 1$ for all $a$, which means that all $a\in[1,p-1]$ has an inverse that we can calculate in this fashion. Thus, $\mathbb{Z}_m^*$ containing the set of integers coprime to $m$ is a group, and will be the group we use for the rest of this section.

Lastly, a **cyclic group** is a group $\mathbb{G}$ in which there is some element, $g$ that **generates** the whole group; that is, all elements of $\mathbb{G}$ are some power of $g$ (for any $a\in\mathbb{G}$, $a= g^e$ for some $e$). For the group $\mathbb{Z}$ under addition and $\mathbb{Z}_p^*$ (for prime $p$) under multiplication, we can assume that some generator exists (this is clear in the case of $\mathbb{Z}$, but not quite for groups of units modulo prime $p$) and use it accordingly.

### Fast Powering

We take an aside and consider group elements of the form $g^e \mod  m$. Unfortunately, computing such an element naively by multiplying $g$ by itself $e$ times is very inefficient; thankfully, by employing [Exponentiation by squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring) we can speed this process up exponentially. In short, we notice that $g^e$ is equivalent to the product of $g^{b_i}$ where $b_i$ are the powers of 2 that add up to $e$. By repeatedly squaring $g$ and only multiplying our result by the powers of 2 that are included in $e$, we can compute $g^e$ in logarithmic time. We eschew a detailed explanation of either algorithm in favor of the Wikipedia article.

### Fermat's Little Theorem and Euler's Theorem

We end this section with two more useful results, which we state without proof. [Fermat's Little Theorem](https://en.wikipedia.org/wiki/Fermat%27s_little_theorem) (FLT) states that given a prime $p$, it is the case that for any $a$,  $a^p \equiv a \mod p$. Equivalently, $a^{p-1} \equiv 1 \mod p$.

[Euler's Theorem](https://en.wikipedia.org/wiki/Euler%27s_theorem), which is like a generalized FLT, states that given any integer $m$, it is the case that for any $a$, $a^{\phi(m)} \equiv 1 \mod m$. Note that $\phi(m)$ is [Euler's totient function](https://en.wikipedia.org/wiki/Euler%27s_totient_function), defined as the number of positive integers that are coprime to $m$ and less than $m$. In particular, for prime $p$, $\phi(p) = p-1$. If $m = p\cdot q$ for primes $p, q$, $\phi(m) = (p-1)(q-1)$.


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


## A Word of Caution

While we are having you implement some schemes on your own, know that this is an exercise to help you understand these algorithms, not a warrant to use home-rolled schemes in the wild. Building these schemes so that they are efficient and work securely all the time is a fool's errand, and we are all better off using standardized implementations from well-vetted libraries (as we will do for the rest of the course). A pledge for another scheme, AES, rings true:

> I promise that once I see how simple AES really is, I will not implement it in production
code even though it will be really fun. This agreement will remain in effect until I learn
all about side-channel attacks and countermeasures to the point where I lose all interest
in implementing AES myself.


---
title: Zero Knowledge Proofs
name: zkp
---

# Background Knowledge

In this assignment, you'll build a cryptographically secure voting platform. There are four programs involved: Arbiters that generate the election parameters and decrypt the final result, a Registrar server that handles checking that all voters are registered to vote only once, a Tallyer server that checks that all votes are valid, and the Voter itself. All of these parties will interact to conduct an election.

We **highly** recommend reading [Cryptographic Voting - A Gentle Introduction](https://eprint.iacr.org/2016/765.pdf) in full, or at least referencing it when writing your ZKPs. It will help **immensely** in understanding the mathematics of this assignment.

## Additively Homomorphic Encryption

In standard encryption, encrypted data must be decrypted before it can be meaningfully altered. Indeed, being able to alter a ciphertext to produce a meaningful change in the corresponding plaintext is called **malleability**, and is usually undesirable. In particular, a malleable encryption scheme cannot be used in authenticated encryption. However, being able to compute over encrypted data is very useful, as it allows multiple parties to compute over shared data without leaking the data itself or coordinating beforehand. Encryption schemes that allow for computation over their ciphertexts are called **homomorphic encryption schemes**. Of those, some may only allow either addition or multiplication (called **additively** and **multiplicatively** homomorphic, respectively), while those that allow both are called **fully** homomorphic.

In this project, we'll explore an additively homomorphic encryption scheme. Formally, an additively homomorphic encryption scheme is an encryption scheme with an additional algorithm $\mathsf{HomAdd}$ such that for any two messages $m_0, m_1$, we have that $\mathsf{Dec}(\mathsf{HomAdd}(\mathsf{Enc}(m_0), \mathsf{Enc}(m_1))) = m_0 + m_1$, where $\mathsf{HomAdd}$ is a homomorphic addition operation.  In other words, we can construct a ciphertext for $m_0 + m_1$ using ciphertexts for $m_0$ and $m_1$ individually. Similar definitions exist for multiplicatively homomorphic and fully homomorphic schemes.

We have actually already seen a simple multiplicatively homomorphic encryption scheme: ElGamal encryption. To see why, consider two ciphertexts $c_1 = (g^{r_1}, h^{r_1} \cdot m_1)$ and $c_2 = (g^{r_2}, h^{r_2} \cdot m_2)$. Observe that we can construct a ciphertext for $m_1 \cdot m_2$ by multiplying component-wise to obtain $c = (g^{r_1 + r_2}, h^{r_1 + r_2} \cdot (m_1 \cdot m_2))$. We can apply the same idea to convert this encryption scheme into an additively homomorphic encryption scheme by instead encoding our messages as $g^m$ instead of $m$; then, the above protocol becomes combining $c_1 = (g^{r_1}, h^{r_1} \cdot g^{m_1})$ and $c_2 = (g^{r_2}, h^{r_2}\cdot g^{m_2})$ to get $c = ( g^{r_1 + r_2}, h^{r_1 + r_2} \cdot g^{m_1 + m_2})$.

One glaring issue with this adaptation is that in order to decrypt and recover $m_1 + m_2$ we need to solve the discrete logarithm problem. However, for our purposes, we will only be encrypting small values and combining them a small number of times, so a brute-force, linear-time approach is perfectly fine. Note that this doesn't compromise the security of encryption as the secret key $sk$ and the random $r$'s are still expected to be very large, so a brute-force approach without knowledge of $sk$ is still computationally infeasible.

## Threshold Encryption

Let's say we will be using homomorphic encryption that allows anyone to add their vote to a publicly tracked value or set of values. As of now, a single party (arbiter) solely holds the decryption key and can check the value at any time they please. This isn't necessarily desirable; it would be nice if decryption keys could be split amongst multiple parties (arbiters) and a ciphertext can only be **jointly** decrypted by all the parties together. This is known as **threshold encryption** and is also achievable with ElGamal encryption.

In threshold ElGamal encryption, $n$ parties will get together and each generate a keypair $(sk_i, pk_i)$ where $pk_i = g^{sk_i}$. Each party publishes $pk_i$ and keeps $sk_i$ private. They will then multiply their public values together and obtain $pk = \prod_i pk_i = g^{\sum_i sk_i}$. Encryption should use this combined public key $pk$.

In order to jointly decrypt a ciphertext $c=(c_1, c_2)$ that is encrypted using this public key, each party can partially decrypt the ciphertext, and then the parties can combine their partial decryptions to get a full decryption. To compute a partial decryption of $c$, each party computes $c_1^{sk_i}$. Then, multiplying all partial decryptions together retrieves $\prod c_1^{sk_i} = c_1^{\sum sk_i} = c_1^{sk}$, which can then be used to decrypt the second component of the ciphertext, namely $g^m = c_2/c_1^{sk}$.

## Zero-Knowledge Proofs

Let's say that our protocol allows voters to encrypt 1 or 0 and post it to the public message board as a vote for or against a particular policy. How will we know that the voters haven't cheated and posted an encryption of 100, without decrypting every ciphertext and checking that it is, in fact, 1 or 0? **Zero-knowledge proofs** allow us to prove this fact, among many others, without revealing any other information. They are a powerful cryptographic primitive that allows us to build trust without unnecessarily revealing information. We'll explore three zero-knowledge proof protocols to get the hang of things.

### Proving Correct Encryption 

The first ZKP we'll explore is a protocol to prove that a ciphertext $c=(c_1,c_2)$ is an ElGamal encryption of 0 under a public key $pk$, where the witness is the randomness $r$ used in the encryption, in particular $c_1 = g^r$ and $c_2 = pk^r$. The protocol is as follows. 
- First, the prover samples a random $r'$ from $(0, q-1)$ and sends $(A = g^{r'}, B = pk^{r'})$ to the verifier.
- Then the verifier chooses a random value $\sigma$ from $(0, q-1)$. 
- The prover then sends back $r'' = r' + \sigma \cdot r \mod q$ to the verifier. 
- Finally, the verifier checks if $g^{r''} = A \cdot c_1^{\sigma}$ and $pk^{r''} = B \cdot c_2^{\sigma}$. 

Similarly, we can prove a ciphertext $c=(c_1,c_2)$ is an encryption of 1 under a public key $pk$, where the witness is the randomness $r$ used in the encryption, in particular $c_1 = g^r$ and $c_2 = pk^r \cdot g$. We can re-write it as $c_1 = g^r$ and $c_2/g = pk^r$, and then use the above ZKP to prove that $(c_1, c_2/g)$ is an encryption of 0 (using the randomness $r$).

As we discussed in class, this **sigma protocol** satisfies completeness, is a proof of knowledge of $r$, and is honest-verifier zero-knowledge. A more detailed explanation can be found in the lecture notes and the readings.

### Proving OR Statement

The ZKP we need in the project is a proof that a ciphertext is an encryption of either 0 or 1. Proving AND statements is straightforward; simply prove both statements. However, proving OR statements is significantly more difficult since one of the statements could be false. We'll approach this ZKP in steps and build up to a protocol that works.

Consider the aforementioned ZKP that $c=(c_1,c_2)$ is an encryption of 0. Notice that the prover can actually cheat in the ZKP if she knows $\sigma$ before sending the first-round message. In particular, she can first randomly sample the third-round reponse $r''$ from $(0, q-1)$, and then compute the first-round message by $A=g^{r''}/c_1^{\sigma}$ and $B = pk^{r''} / c_2^{\sigma}$, which will end up verifying correctly. We can use this observation to generate a ZKP for an OR statement.

The protocol is as follows. Suppose $c=(c_1,c_2)$ is an encryption of 1 and the prover knows the randomness $r$. (If $c$ is an encryption of 0, the protocol follows similarly.)
- The prover first randomly samples $\sigma_0$ from $(0, q-1)$ and uses the above trick to "simulate" a valid sigma protocol for the (false) statement that $c=(c_1,c_2)$ is an encryption of 0, using $\sigma_0$ as the challenge.
- The prover performs two ZKPs simultaneously, one proving $c$ is an encryption of 0 and one proving $c$ is an encryption of 1. For the one proving $c$ is an encryption of 0, the prover sends the first-round message from the simulated protocol.
- The verifier then sends a randomly sampled challenge $\sigma$ from $(0, q-1)$.
- The prover computes $\sigma_1 = \sigma - \sigma_0 \mod q$. For the proof that $c$ is an encryption of 1, the prover generates a ZKP response based on the challenge $\sigma_1$ and her witness $r$, and sends it back to the verifier along with $\sigma_1$.
For the proof that $c$ is an encryption of 0, the prover sends back to the verifier $\sigma_0$ along with the third-round message from the simulated proof.
- The verifier finally checks both ZKPs and that $\sigma_0 + \sigma_1 = \sigma \mod q$.

This protocol is known as **Disjunctive Chaum-Pedersen** (DCP), or the **Sigma-OR** protocol. A more detailed explanation can be found in the lecture notes and the readings.

### ZKP for Partial Decryption

We explore another ZKP that proves that a partial decryption of a ciphertext $c=(c_1,c_2)$ is correct. The partial decryption with regard to a partial public key $pk_i$ is $d$, where the witness is the partial secret key $sk_i$, in particular, $pk_i = g^{sk_i}$ and $d = c_1^{sk_i}$. The protocol is as follows. 
- First, the prover samples a random $r$ from $(0, q-1)$ and sends $(A = g^{r}, B = c_1^{r})$ to the verifier.
- Then the verifier chooses a random value $\sigma$ from $(0, q-1)$. 
- The prover then sends back $s = r + \sigma \cdot sk_i \mod q$ to the verifier. 
- Finally, the verifier checks if $g^{s} = A \cdot pk_i^{\sigma}$ and $c_1^{s} = B \cdot d^{\sigma}$. 

### Non-Interactive Zero-Knowledge (NIZK)

All the zero-knowledge proofs (sigma protocols) we have discussed above are only honest-verifier zero-knowledge (HVZK), namely it is zero-knowledge against an honest verifier who samples the challenge $\sigma$ uniformly at random. The Fiat-Shamir heuristic allows us to transform these sigma protocols into non-interactive zero-knowledge (NIZK) proofs in the random oracle model. Instead of asking the verifier to sample $\sigma$, we compute $\sigma$ from a hash function computed on the ZK statement along with the first-round message. For example, in ZKP for partial decryption, we compute $\sigma = H(pk_i, c, d, A, B)$, where $H$ is a hash function modeled as a random oracle.

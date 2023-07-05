---
title: Multiparty Computation
name: mpc
---

# Background Knowledge

In this assignment, you'll implement a simple version of Yao's garbled circuits using a simple version of oblivious transfer. This assignment leaves a lot of room for optimizations, which we leave to the reader to explore. However, in your final submission, **do not** submit with any optimizations as it will not be compatible with the autograder.

We highly recommend reading the first few pages of [The Simplest Protocol for Oblivious Transfer](https://eprint.iacr.org/2015/267.pdf) for OT, and [A Gentle Introduction to Yao's Garbled Circuits](https://web.mit.edu/sonka89/www/papers/2017ygc.pdf) and [Faster Secure Two-Party Computation Using Garbled Circuits](https://www.usenix.org/legacy/event/sec11/tech/full_papers/Huang.pdf) for garbled circuits. It will help immensely in understanding the mathematics of this assignment.

## Oblivious Transfer

A foundational building block in secure multi-party computation is oblivious transfer (OT), which allows a receiver to select a message from a sender without the sender learning which message they selected, nor the receiver learning more than the message they selected. At first glance, OT seems impossible to achieve, but as we will see, we can use basic primitives that we've seen already to build a simple yet secure protocol for 1-out-of-2 OT.

We present an implementation of OT based on the Diffie-Hellman key exchange. The protocol proceeds as follows:
- The sender prepares two messages, $m_0, m_1$ to send to the receiver.
- The sender generates a Diffie-Hellman keypair, $(a, g^a)$, and sends $A = g^a$ to the receiver.
- The receiver generates a Diffie-Hellman keypair, $(b, g^b)$.
	- If the receiver wishes to receive $m_0$, they send back $B = g^b$.
	- If they wish to receive $m_1$, they send back $B = A g^b$.
- The receiver generates shared key $k_c = \mathsf{HKDF}(A^b)$.
- The sender generates $k_0 = \mathsf{HKDF}(B^a)$ and $k_1 = \mathsf{HKDF}((B/A)^a)$ and encrypts $e_0 \gets \mathsf{Enc}_{k_0}(m_0)$ and $e_1 \gets \mathsf{Enc}_{k_1}(m_1)$, sending both to the receiver.
	- Notice that depending on the sender's choice bit, the key for the message they selected will be equal to $k_c$.
- The receiver decrypts the ciphertext they selected, retreiving $m_c = \mathsf{Dec}_{k_c}(e_c)$.

![Architecture simple OT](/static/img/handout/yaos/simple-OT.png)

## Yao's Garbled Circuits

Yao's garbled circuits allow two parties to jointly compute over a boolean circuit without learning any intermediate values or the other party's inputs. This is an immensely useful primitive as it allows two parties to jointly compute any function securely. We'll describe a secure two-party computation protocol that uses garbled circuits (without any optimization) and our OT implementation above, along with some other primitives we've been interacting with.

All of our circuits are specified using [Bristol Format](https://homes.esat.kuleuven.be/~nsmart/MPC/old-circuits.html), which consists of three types of gates: AND, XOR, and NOT. We provide parsers to ease development. We highly recommend reading up on the format, should you need to debug a particular circuit or wish to write your own.

We present a simple construction of garbled circuits. The **garbler** and **evaluator** are the two parties. The protocol proceeds as follows:
- The garbler parses circuit $C$ and obtains a set of gates and wires to process.
- For each wire, the garbler samples a random 0-label and a random 1-label, each being a $\lambda$-bit string.
- For each gate, the garbler will produce a garbled gate consisting of 4 distinct ciphertexts per AND/XOR gate and 2 distinct ciphertexts per NOT gate, where each ciphertext is a double encryption of the corresponding output label (tagged with $\lambda$ trailing 0's so we can identify when decryption was successful) using the two input labels as keys. We instantiate the double encryption by a hash function. Concretely, let's say we're garbling an AND gate with input wires $w_x, w_y$ and output wire $w_z$. Then, the ciphertexts will be as follows:

![Yaos Encryption](/static/img/handout/yaos/yaos-encrypt.png)

- The garbler randomly permutes all 4 (or 2) ciphertexts for each garbled gate and sends all of them to the evaluator.
- The garbler also sends labels corresponding to the garbler's input.
- The evaluator runs OT with the garbler to retrieve the labels corresponding to its input.
- The evaluator evaluates the garbled circuit gate by gate, from the input labels all the way to output labels.
- The evaluator sends the labels correponding to the output wires to the garbler, which then reveals the final output to the evaluator.

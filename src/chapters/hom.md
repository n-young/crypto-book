---
title: Homomorphic Encryption
name: hom
---

## Homomorphic Encryption

We have already encountered additively homomorphic encryption in the Vote project. In this project, we will utilize an encryption scheme that supports both homomorphic addition and homomorphic multiplication. In particular, consider an encryption scheme with additional homomorphic evaluation algorithms $\mathsf{HomAdd}$ and $\mathsf{HomMul}$ such that for any two messages $m_0, m_1$, we have both that $\mathsf{HomAdd}(\mathsf{Enc}(m_0), \mathsf{Enc}(m_1)) = \mathsf{Enc}(m_0 + m_1)$ and that $\mathsf{HomMul}(\mathsf{Enc}(m_0), \mathsf{Enc}(m_1)) = \mathsf{Enc}(m_0 \cdot m_1)$.  In other words, we can construct a ciphertext for $m_0 + m_1$ or $m_0 \cdot m_1$ using ciphertexts for $m_0$ and $m_1$ individually.

Achieving fully homomorphic encryption which supports homomorphic evaluation for all polynomial-sized circuits is fairly inefficient due to the expensive bootstrapping step. Nevertheless, for most practical applications such as private information retrieval (PIR), it suffices to have a slightly weaker primitive known as somewhat homomorphic encryption (SWHE). SWHE also supports both homomorphic addition and homomorphic multiplication, but allows for only a bounded number of homomorphic operations.

## Private Information Retrieval

We turn our attention to the problem of private information retrieval (PIR). In PIR, we have a server and a client, where the server holds a database of $n$ plaintexts $D=\{p_1,\dots, p_n\}$ and the client wants to retrieve $p_k$ for some $k\in[n]$. We would like to allow the client to retrieve $p_k$ without the server learning $k$. This is distinct from OT as the client is allowed to learn as many values as she wants. There is a trivial solution in which the server simply sends the entire database to the client; however, this requires communication complexity of $O(n)$, and our goal in PIR is to achieve lower (sublinear) communication complexity.

We can use somewhat homomorphic encryption to achieve a PIR scheme. If the client wants to retrieve $p_k$, we can send a selection vector $s = (s_1, \ldots, s_n)$ where $s_k$ is an encryption of 1 and every other $s_i$ is an encryption of 0. Now, the server can homomorphically compute $p'_i = p_i \cdot s_i$ and then sum up all values of $p'_i$ to retrieve an encryption of just the selected value. The communication complexity from the server to the client is constant (namely a single ciphertext), which is a huge improvement compared to sending the entire database. However, the communication cost from the client to the server is still on the order of the size of the database. 

By organizing the database into a square of side length $\sqrt{n}$ and sending a selection vector for each dimension $(x, y)$, the client only needs to send $2\cdot \sqrt{n}$ ciphertexts. However, we end up incurring more ciphertext multiplications on the server side. This tradeoff between communication and computation is hard to strike perfectly. We can generalize this approach to organize our database into a hypercube of side length $\sqrt[d]{n}$, and then sending $d\cdot \sqrt[d]{n}$ ciphertexts from the client to the server. In this project we will implement the generic protocol to work with any dimension.

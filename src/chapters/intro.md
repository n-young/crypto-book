---
title: Introduction
name: intro
---

# Welcome!

Welcome to this short webbook on modern cryptographic techniques! My goal in writing this book is primarily to provide an accessible, understandable, centralized place to explain as much about cryptography as I can. I've split this book up into relatively isolated chapters - that being said, reading in order is recommended.

Throughout the chapters, we will go over the classical, everyday cryptography that many of us are used to, and then cover more advanced, modern topics with more specialized applications. Indeed, we are currently experiencing a renaissance of sorts in the cryptographic community with blockchain technology and consumer privacy fuelling much of the excitement. While these facets are indeed interesting and we will draw from them to motivate some of our applications, they are not the focus of this book. We will stick to the cryptography itself.

Specifically, we will first cover private key and public key primitives, and some of the everyday applications of both. Next, we will explore zero knowledge proofs, a technique for proving that a particular statement is true without revealing any other unnecessary information. Following that, we will dive into multiparty computation, a set of protocols in which multiple parties can jointly compute a result over some inputs without revealing the inputs to each other. Lastly (for now), we will explore homomorphic encryption, a set of techniques that allow computation over encrypted data. However before anything else, we must go over some preliminary mathematics, which this article will cover.

## Elementary Number Theory

The following is an overview of the algebra and number theory necessary to understand the cryptographic schemes in this book. You can safely skip this section if you're already familiar with elementary number theory. We try to use notation that is common in the literature.

### Divisibility and GCDs

Consider two integers $a,b\in\mathbb{Z}$, where $\mathbb{Z}$ denotes the set of all integers. We say that $a$ **divides** $b$ if there exists an integer $c \in \mathbb{Z}$ such that $a \cdot c = b$. We denote this by $a \mid b$.

Given integers $a, b, m \in \mathbb{Z}$. We say that $a$ and $b$ are **congruent mod $m$** if there exists an integer $k\in\mathbb{Z}$ such that $a + km = b$. In other words, it means that $a$ and $b$ differ by a multiple of $m$, or that when divided by $m$, they yield the same remainder. We denote this by $a \equiv b \mod m$.

Recall **greatest common divisors (GCDs)**. Given two integers $a, b \in\mathbb{Z}$, the GCD of $a$ and $b$ is the largest integer $d\in\mathbb{Z}$ such that $d \mid a$ and $d \mid b$. We say that two integers are **coprime** if their GCD is 1. Calculating the GCD of two integers can be done efficiently using the [Euclidean Algorithm](https://en.wikipedia.org/wiki/Euclidean_algorithm), and calculating integers $s, t$ such that $s \cdot a + t \cdot b = \gcd(a, b)$ can be done efficiently using the [Extended Euclidean Algorithm](https://en.wikipedia.org/wiki/Extended_Euclidean_algorithm). We eschew a detailed explanation of either algorithm in favor of the Wikipedia articles - the important thing to know is that calculating GCDs and inverses is efficient.

### Groups

To allow us to generalize later on, we introduce the notion of a group. A **group** is defined as a set $\mathbb{G}$ along with a binary operation $\otimes: \mathbb{G} \times \mathbb{G} \rightarrow \mathbb{G}$, such that the following three properties hold:
1. **Identity**: there exists an identity element $e\in\mathbb{G}$ such that for any element $a \in \mathbb{G}$ we have that $e \otimes a = a \otimes e = a$.
2. **Associativity**: for any $a, b, c \in \mathbb{G}$, we have that $(a \otimes b) \otimes c = a \otimes (b \otimes c)$.
3. **Inverses**: for any $a \in \mathbb{G}$, there exists an inverse element $b \in \mathbb{G}$ such that $a \otimes b = b\otimes a = e$. We denote the inverse of $a$ by $a^{-1}$.

For example, $\mathbb{Z}$ under addition (where our operation $\otimes$ is $+$) is a group (the integers are actually much, much richer than a group, but for now consider them as a group). It has identity $0$, addition is associative, and every integer $n$ has $-n$ as its inverse. An example of a *non-group* could be $\mathbb{Z}$ under multiplication. While it has an identity element $1$ and is associative, not every element has a multiplicative inverse. However, if we change our set slightly to be the non-zero rational numbers $\mathbb{Q}^{\neq 0}$ with multiplication, this again is a group.

The set of integers modulo a prime $p$ (excluding $0$) under *multiplication* is also a group, denoted as $\mathbb{Z}_p^* = \{1,2,\dots,p-1\}$. More generally, if you consider the integers from $[1,m-1]$ that are coprime to $m$, then we can construct a special group called the **multiplicative group of units**, denoted as $\mathbb{Z}_m^*=\{a \mid a \in [1,m-1], \gcd(a,m)=1\}$.

Let's verify that $\mathbb{Z}_m^*$ is indeed a group. Notice that for all $a \in \mathbb{Z}_m^*$, we have that $\gcd(a, m) = 1$ by definition. Finding an inverse is as simple as running the Extended Euclidean Algorithm. Taking the relation $s \cdot a + t\cdot m = 1 \mod  m$, we get $s \cdot a \equiv 1 \mod  m$ where $s$ is the inverse of $a$. It's worth noting that the $s$ must necessarily also be coprime to $m$; otherwise, we wouldn't be able to write 1 as a linear combination using it and $m$ (see: Bezout's Identity). Identity and associativity clearly hold. Thus, $\mathbb{Z}_m^*$ containing the set of integers coprime to $m$ is a group, and will be the group we use for the rest of this section.

Lastly, a **cyclic group** is a group $\mathbb{G}$ in which there is some element, $g$ that **generates** the whole group; that is, all elements of $\mathbb{G}$ are some power of $g$ (for any $a\in\mathbb{G}$, $a= g^e$ for some $e$). For the group $\mathbb{Z}$ under addition, and $\mathbb{Z}_p^*$ (for prime $p$) under multiplication, we can assume that some generator exists (this is clear in the case of $\mathbb{Z}$, but not quite for groups of units modulo prime $p$) and use it accordingly.

### Fast Powering

We take an aside and consider group elements of the form $g^e \mod  m$. Unfortunately, computing such an element naively by multiplying $g$ by itself $e$ times is very inefficient (linear in $e$); thankfully, by employing [Exponentiation by squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring) we can speed this process up exponentially. In short, we notice that $g^e$ is equivalent to the product of $g^{b_i}$ where $b_i$ are the powers of 2 that add up to $e$. By repeatedly squaring $g$ and only multiplying our result by the powers of 2 that are included in $e$, we can compute $g^e$ in logarithmic time. We eschew a detailed explanation of either algorithm in favor of the Wikipedia article.

### Fermat's Little Theorem and Euler's Theorem

We end this section with two more useful results, which we state without proof. [Fermat's Little Theorem](https://en.wikipedia.org/wiki/Fermat%27s_little_theorem) (FLT) states that given a prime $p$, it is the case that for any $a$,  $a^p \equiv a \mod p$. Equivalently, $a^{p-1} \equiv 1 \mod p$.

[Euler's Theorem](https://en.wikipedia.org/wiki/Euler%27s_theorem), which generalizes FLT, states that given any integer $m$, it is the case that for any $a$, $a^{\phi(m)} \equiv 1 \mod m$. Note that $\phi(m)$ is [Euler's totient function](https://en.wikipedia.org/wiki/Euler%27s_totient_function), defined as the number of positive integers that are coprime to $m$ and less than $m$. In particular, for prime $p$, $\phi(p) = p-1$. If $m = p\cdot q$ for primes $p, q$, $\phi(m) = (p-1)(q-1)$.

## Proofs of Security

TODO:

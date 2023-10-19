---
title: "The Cypherpunk's Notes"
date: 2023-10-16T20:45:33-07:00
draft: false
image: cover.jpg
description: "A work-in-progress compilation (and living document) of notes taken while studying various cryptography resources"
math: true
toc: true
categories: 
    - Cypherpunk
tags:
    - Mathematics
comments: false
---

> From Dan Boneh's [Cryptography I](https://crypto.stanford.edu/~dabo/courses/OnlineCrypto/) and the [MoonMath Manual](https://leastauthority.com/community-matters/moonmath-manual/), to [ZK Hack's Whiteboard Sessions](https://zkhack.dev/whiteboard/); and soon, the [Proofs, Arguments, and Zero-Knowledge](https://people.cs.georgetown.edu/jthaler/ProofsArgsAndZK.html) book by Justin Thaler. Many topics I have taken notes for already are not yet included here, as I am working on sectioning it in such a way that the material flows more intuitively; In other words, expect many changes to this document in the future.

## Introduction

Cryptography is _everywhere_. From securing communications ([HTTPS](https://en.wikipedia.org/wiki/HTTPS), [SSL/TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security#SSL_1.0,_2.0,_and_3.0), [WPA](https://en.wikipedia.org/wiki/Wi-Fi_Protected_Access), etc.) to encrypting sensitive, persisted files, cryptography is a tremendous tool, and forms the basis for many security mechanisms. Cryptography _is not_ the solution to _all_ security problems, it is _not_ reliable if used incorrectly, and it is _not_ something you should try to invent yourself (there are many examples of broken adhoc designs, and this is why we use the phrase "_don't roll your own crypto!_").

## Some Notes on Notation

### Discrete Notation

There are a few notational concepts that show up a lot in the following material. Some people may be familiar with these already, and others may not be. I figured I'd write some notes on them here so the following material is more easily accessible for those who may not have studied Discrete Math yet.

We use the symbol $\forall$ to denote the concept of "_for all_", and the symbol $\in$ denotes the concept "_in_". We often combine these symbols to make statements such as, _for all_ $x$ _in_ the set $X$, which we could explicitly write as
$$\forall x\in X$$

It is also useful to discuss the _existence_ of arbitrary values. For this, we use the $\exists$ symbol.

$$\exists x\in X\implies\text{there exists an }x\text{ in }X$$
$$\nexists x\in X\implies\text{there does not exist an }x\text{ in }X$$

Another concept we like to represent through notation is the phrase "_such that_", this allows us to restrict certain sets of values to abiding by an arbitrary property. Consider the case in which we'd like to discuss the set of all ordered-pairs in $X$ such that no pair contains two of the same values, we may write this as
$$\lbrace(x,y)\in X\space|\space x\ne y\rbrace$$

Combining the above concepts, you should feel comfortable reading expressions like

$$\forall x\in X,\exists y\in Y\space|\space f(x)=y$$

### Functional Notation

When discussing functions, it is useful to be able to relate them to their domain and codomain. This allows us to clearly state the set of all inputs the function can accept, and the set of all outputs the function can generate. We use the notation
$$f:X\rightarrow Y$$
to show that $f$ is a function that accepts inputs from the set $X$ and maps them to outputs in the set $Y$.

## Ciphers

A _cipher_ is a pair of efficient* algorithms $(E,D)$ that are defined over a _keyspace_ $\mathcal K$, a _message space_ $\mathcal M$, and a _cipher space_ $\mathcal C$. For convenience, we say that a cipher is defined over $(\mathcal K,\mathcal M,\mathcal C)$, where
$$E:\mathcal K\times\mathcal M\rightarrow\mathcal C$$
$$D:\mathcal K\times\mathcal C\rightarrow\mathcal M$$
such that for all messages $m$ in the message space ($\forall m\in\mathcal M$), and for all keys $k$ in the keyspace,
$$D(k,E(k,m))=m$$

## Discrete Probability

### Universe

Discrete probability is defined over a _universe_, denoted \\(U\\). The universe is a _finite set_, and very commonly, simply the set of all \\(n\\)-bit strings,
$$U=\lbrace 0,1 \rbrace^n.$$
It then follows that \\(U\\) has \\(2^n\\) elements, or, we could say that the cardinality of the universe \\(U\\) as defined above, is \\(2^n\\), and denote it as $|U|$. For example, the universe \\(U=\lbrace 0,1\rbrace^2\\) is the set of all \\(2\\)-bit strings, e.g., \\(\lbrace 00,01,10,11 \rbrace\\).

### Probability Distributions

A probability distribution \\(Pr\\) over a universe \\(U\\) is a function \\(Pr:U\rightarrow[0,1]\\) such that
$$\sum_{x\in U}Pr(x)=1.$$
To say this less formally, we could say that \\(Pr\\) is a function that maps an input \\(x\in U\\) to a value ranging from \\(0\\) to \\(1\\), in such a way that the summation of all the images of \\(Pr\\) equals \\(1\\).

#### Uniform Distribution

A uniform distribution is a type of probability distribution in which all outcomes are _equally likely_.
$$\forall x\in U,Pr(x)=\frac{1}{|U|}$$

#### Events

An event is a subset of a universe and is to be understood as something that might or might not happen. Intuitively, when dealing with probabilities, we are considering the likelihood of certain outcomes. Each of these potential outcomes, or collections of outcomes, is called an _event_. The probability of an event $A\subseteq U$ occuring is
$$Pr(A)=\sum_{x\in A}Pr(x)\quad\in[0,1]$$

To say this less formally, the probability of an event $A$ occuring, $Pr(A)$, is the sum of all the probabilities of the individual outcomes that make up $A$. To illustrate this, let's discuss an example. Suppose $U$ is the set of all outcomes when rolling a fair six-sided die, i.e., $U=\lbrace1,2,3,4,5,6\rbrace$. If you consider the event $A$ as rolling an _even_ number, then $A=\lbrace2,4,6\rbrace$. In a _uniform distribution_ (like a fair die), the probability of each outcome is $\frac{1}{|U|}=\frac{1}{6}$. To find $Pr(A)$, you would sum up the probabilities of the individual outcomes in $A$:
$$Pr(A)=Pr(2)+Pr(4)+Pr(6)=\frac{1}{6}+\frac{1}{6}+\frac{1}{6}=\frac{1}{2}$$

#### Distribution Vector

Let \\(|U|=n\\) be the cardinality of a universe \\(U\\). Because \\(U\\) is a finite set, we can represent the distribution of every element in \\(U\\) by storing the weights that the distribution assigns to every element in \\(U\\) as an \\(n\\)-vector:
$$\begin{bmatrix}
Pr(x_0) \\\\
Pr(x_1) \\\\
\vdots \\\\
Pr(x_n)
\end{bmatrix}$$

### Random Variables
A random variable is a function that maps outcomes of random processes (like a coin flip, or the roll of a die) to numerical values. In other words, it assigns a value to each possible outcome of a random experiment or sample.

Formally, if $U$ is the universe of all possible outcomes of a certain random experiment, then a random variable $x$ is defined as

$$x:U\rightarrow V$$
where $V$ is the set in which $x$ takes its values.

### Uniform Random Variable
A random variable is said to be uniformly distributed if all the values in its range have an equal likelihood of occuring. Explicitly, let \\(U=\lbrace 0,1 \rbrace^n\\) be some universe. We write \\(r\xleftarrow{R}U\\) to denote a ***uniform random variable*** over \\(U\\), where
$$\forall a\in U,\quad Pr[r=a]=\frac{1}{||U||}.$$

## XOR
It is said that all cryptographers do is compute XORs in various ways, and there is some truth to that, so what is XOR?

XOR, or _exclusive or_, is a **bitwise** operator, i.e., it operates on sets of bits $\lbrace0,1\rbrace^n$. XOR can be thought of as a modification of the OR operator, so to better understand it, let's review the behavior of OR.

The way OR works is intuitive. What it does is essentially, for every two bits $a_i,b_i$, it asks is one OR the other equal to $1$. Taking the OR of $1$ and $0$ outputs $1$, taking the OR of $1$ and $1$ outputs $1$, but the OR of $0$ and $0$ outputs $0$, since neither of the bits are $1$. So,
$$\lbrace1,1,1,0\rbrace$$
$$\text{ OR }$$
$$\lbrace0,1,1,1\rbrace$$
$$\text{yields }\lbrace1,1,1,1\rbrace$$

The difference with XOR is it's _exclusive_, meaning it _must be one or the other, but not both_.
$$\lbrace1,1,1,0\rbrace$$
$$\text{ XOR }$$
$$\lbrace0,1,1,1\rbrace$$
$$\text{yields }\lbrace1,0,0,1\rbrace$$
Also, note that, in the following material, we use the symbol $\oplus$ to denote the bitwise XOR operator.

## Information Theoretics

### Perfect Secrecy

Let \\(\mathcal{K}\\) be the keyspace, \\(\mathcal M\\) be the message space, and \\(\mathcal C\\) be the cipher space. A cipher \\((E,D)\\) over \\((\mathcal K,\mathcal M,\mathcal C)\\) has **perfect secrecy** if
$$\forall m_0,m_1\in\mathcal M,\quad ||m_0||=||m_1||,$$
where \\(||m_0||\\) denotes the_length of \\(m_0\\), and
$$\forall c\in\mathcal C,\quad\text{Pr}[(E(k,m_0)=c)]=\text{Pr}[(E(k,m_1)=c)],$$
where \\(k\\) is uniform and random in \\(\mathcal K\\), i.e., \\(k\xleftarrow{R}\mathcal K\\). In other words, \\((E,D)\\) has perfect secrecy if for every message \\(m_1,m_2\\) in the message space \\(\mathcal M\\), the length of \\(m_1\\) and \\(m_2\\) are the same, and additionality, the probability that the ciphertext \\(c\\) came from \\(m_0\\) is equal to the probability that it came from \\(m_1\\).

## Symmetric Encryption Systems
Symmetric encryption systems form the building blocks for many modern cryptographic protocols. In a symmetric encryption system, there are two algorithms,
* $E$: encryption algorithm, and
* $D$: the decryption algorithm
  * important! $D$ is _publicly known_: never use a proprietary cipher!

It is called _symmetric_ encryption because the two parties, Alice and Bob, _share a secret key_ $k$.

{{< figure src="symm.png" >}}

### One Time Pad
The **One Time Pad** is a simple symmetric encryption scheme that is a good starting point in our studies, due to its simplicity. Let $\mathcal M=\mathcal K=\mathcal C=\lbrace0,1\rbrace^n$ and $\forall m\in\mathcal M$ and $k\in\mathcal K$, $|m|=|k|$. We define the encryption and decryption algorithms as

$$E(k,m)=k\oplus m$$
$$D(k,c)=k\oplus c$$

## Polynomials

Polynomials are algebraic expressions composed of variables, commonly referred to as indeterminates, and coefficients. These coefficients function to scale the indeterminates and must be uniformly categorized, meaning they should all belong to a specific number set, such as integers or rational numbers.

### Terminology

> _**Univariate**_ polynomials are polynomials _of one variable_. More precisely,
>
> $$P(x)=\sum_{j=0}^m a_jx^j=a_mx^m+a_{m-1}x^{m-1}+...+a_1x+a_0,$$
>
> where \\(a_j,...,a_1\\) are the coefficients, and \\(x\\) is the variable. If we let \\(R\\) denote the _type_ of the coefficients (e.g., rational, real, integer), then the set of all _univariate polynomials with coefficients in_ \\(R\\) is denoted \\(R[x]\\).

---

> ***Zero Polynomial***
>
> The _zero polynomial_ is a polynomial in which **all the coefficients are \\(0\\)**.

---

> ***One Polynomial***
>
> A polynomial is called the **one polynomial** if it is a _zero polynomial_ with the constant term (\\(a_0\\) above) of \\(1\\).

---

> ***Leading Coefficient***
>
> The **leading coefficient** is the the coefficient **of the term with the highest degree**. We denote it as follows,
> $$Lc(P)=a_m$$

### Notation

#### Degree-bounded Sets of Polynomials

It is useful to look at sets of polynomial w.r.t. their _degree_. If we, for instance, say \\(m\\) is the maximum degree allowed, we'd denote this set as \\(R^{\le m}[x]\\).
> Some other notations put the bound in the subscript, but I think it's cleaner as a superscript; helps me remember that it's a degree bound and not some sort of indexing.

### Polynomial Arithmetic
>
> "Polynomials behave like integers in many ways. In particular, they can be added, subtracted and
multiplied. In addition, they have their own notion of Euclidean Division. Informally speaking,
we can add two polynomials by simply adding the coefficients of the same index, and we can
multiply them by applying the distributive property, that is, by multiplying every term of the
left factor with every term of the right factor and adding the results together."

More formally, let \\(P(x),R(x)\in R[x]\\) be polynomials with \\(P(x)=\sum_{n=0}^{m_1}a_nx^n\\) and \\(R(x)=\sum_{n=0}^{m_2}b_nx^n\\). Then the **sum** of these polynomials is

$$\sum_{n=0}^{m_1}a_nx^n+\sum_{n=0}^{m_2}b_nx^n\implies\sum_{n=0}^{\textbf{max}(m_1,m_2)}(a_n+b_n)x^n,$$

while the **product** is defined as

$$\left(\sum_{n=0}^{m_1}a_nx^n\right)\cdot\left(\sum_{n=0}^{m_2}b_nx^n\right)\implies\sum_{n=0}^{m_1+m_2}\sum_{i=0}^n a_ib_{n-i}x^n.$$

> **Example 22: [MoonMath Manual](https://leastauthority.com/community-matters/moonmath-manual/)**
>
> To give an example of how polynomial arithmetic works, consider the following two integer polynomials \\(P,Q\in\mathbb Z[x]\\) with \\(P(x)=5x^2-4x+2\\) and \\(Q(x)=x^3-2x^2+5\\). The sum of these two polynomials is computed by adding the coefficients of each term with equal exponent in \\(x\\) (**Eq. 2**). This yields the following:
> $$\begin{align*}
> (P+Q)(x) &= (0+1)x^3+(5-2)x^2+(-4+0)x+(2+5) \\\\
> &= x^3+3x^2-4x+7.
> \end{align*}$$
> The product \\(P\cdot Q\\) is computed by multiplying each term in the first factor with each term in the second factor (**Eq. 3**). We get the following:
> \begin{align}
(P\cdot Q)(x) &= (5x^2-4x+2)\cdot(x^3-2x^2+5) \\\\
              &= (5x^5-10x^4+25x^2)+(-4x^4+8x^3-20x)+(2x^3-4x^2+10) \\\\
              &= 5x^5-14x^4+10x^3+21x^2-20x+10.
\end{align}

### Lagrange Interpolation

A neat thing about polynomials is that, for a polynomial \\(P\\) of degree \\(m\\), \\(P\\) is completely determined by its evaluation on \\(m+1\\) points. An important observation from this is that we can uniquely derive a polynomial of degree \\(m\\) from a set \\(S\\), where
$$S=\lbrace(x_0,y_0),(x_1,y_1),...,(x_m,y_m)\space|\space x_i\ne x_j\text{ for all indices }i\text{ and }j\rbrace.$$

In other words, polynomials have the property that given \\(m+1\\) pairs of points \\(x_i,y_i\\) for \\(x_i\ne x_j\\) are enough to determine the set of pairs \\((x,P(x)),\space\forall x\in R\\).

> #### *Important
>
> If you want to find a polynomial that fits a given set of points--and if its coefficients have a multiplicative inverse--you can use the **Lagrange Interpolation** method. This technique allows you to derive a polynomial \\(P\\) of degree \\(m\\) where \\(P(x_i)=y_i,\space\forall (x_i,y_i)\in S\\).
>
#### Algorithm: Lagrange Interpolation

{{< figure src="lagrange.png" >}}

## Finite (Galois) Fields

Finite fields are used extensively in modern cryptography. This is due to them possessing characteristics that make performing certain operations within them easy. But what even is a field? Why do we restrict them to being _finite_? What's useful about them?

### On Terminology

The term _Finite Field_ is sometimes referred to as a _Galois Field_, after [Évariste Galois](https://en.wikipedia.org/wiki/%C3%89variste_Galois); a french mathematician who made enormous contributions to mathematics in his short 20 year life.

> These terms are interchangable, though we will opt to use the term _Galois Field_ (GF) in what follows to give credit where credit is due.

### Fields

The term _field_ is just a fancy way of denoting a **set** on which the ordinary operations of _addition_, _subtraction_, _multiplication_, and _division_ are defined. In fact, you are already familiar with (at least) a couple fields! For instance, the set of all real numbers \\(\mathbb R\\) forms a field; and likewise, so does the set of all rational numbers, \\(\mathbb Q\\). These operations are undoubtably defined on these sets, so we call them _fields_.

So why doesn't the set of integers \\(\mathbb Z\\) constitue a field? Well, more formally--to be classified as a field--the set must satisfy the following properties:

1. ***Associativity of addition and multiplication***

* \\(\forall a,b,c\in\mathbb F\\):
  $$a+(b+c)=(a+b)+c$$
  $$a\cdot(b\cdot c)=(a\cdot b)\cdot c$$

2. ***Commutativity of addition and multiplication***

* \\(\forall a,b,c\in\mathbb F\\):
  $$a+b=b+a$$
  $$a\cdot b=b\cdot a$$

3. ***Left-distributivity***

* \\(\forall a,b,c\in\mathbb F\\)
  $$a\cdot(b+c)=a\cdot b+a\cdot c$$

4. ***Existence of additive identity***

* \\(\exists i\in\mathbb F\\) such that
  $$\forall a\in\mathbb F,\quad a+i=a$$

5. ***Existence of a multiplicative identity***

* \\(\exists e\in\mathbb F\\) such that \\(e\ne0\\) and
  $$\forall a\in\mathbb F,\quad ae=a$$

6. _**Existence of additive inverse**_ (often denoted as \\(-a\\))

* \\(\forall a\in\mathbb F,\exists b\in\mathbb F\\) such that
  $$a+b=0$$

7. _**Existence of multiplicative inverse**_ (often denoted as \\(a^{-1}\\))

* \\(\forall a\in\mathbb F\\) where \\(a\ne0\\), \\(\exists b\in\mathbb F\\) such that
  $$a\cdot b=1$$

We can pretty clearly see that \\(\mathbb Z\\) satisfies _most_ of these properties. But it is missing one important one: _existence of a multiplicative inverse_. Because \\(\mathbb Z\\) is restricted to only _whole_ numbers, it's trivial to see that \\(\mathbb Z\\) does not satisfy this property, and therfore, does not constitue a field.

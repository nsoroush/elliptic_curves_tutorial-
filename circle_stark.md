:::warning
In this series of blog posts, I'll be diving into the world of Circle STARKs, breaking down its mechanisms for those who are technically inclined and have a foundational understanding of programming and basic math. Although I will touch on the principles of STARKs to compare them with Circle STARKs, this series won't be a deep dive into STARKs themselves. For that, I recommend checking out these [posts](https://aszepieniec.github.io/stark-anatomy/).

So, let's kick off by exploring the Circle STARK-FFT in our first post of the series. Let's get started!


:::

Circle STARK: Simple and Elegant!
---

Since you are reading this blog post, it is likely that you are familiar with Zero-Knowledge Proof (ZKP) systems. If not, my survey on ZKP is available [here](https://nsoroush.github.io/assets/docs/SurveyOnZK.pdf)! So from now on, I'll presume  that you know what a ZKP is.

In the framework of cryptographic proof systems, the structures we use to build these proofs can often be complex and computationally intensive. Much research has been done proposing new ZKP systems, and one of the recent advancements is STARK. STARKs, which stand for Scalable, Transparent ARguments of Knowledge, are cryptographic proof systems that have rapidly developed and found deployment in numerous applications. They are popular because they do not require any trusted setup, and the arithmetization field for STARKs is not reliant on cryptographic hard problem, they use Reed-Solomon IOPs instead. This flexibility enables the selection of fields that optimize performance. 

⁤However, traditional STARKs has some limitations since they require a cyclic group of [smooth order](https://hackmd.io/KnqE9BwWS2yoDDsQHTPVoQ#Cyclic-group-of-a-smooth-order-in-the-field) in the field to efficiently interpolate points using the FFT algorithm and to write constraints involviing neighboring rows. ⁤⁤This restriction can limit the types of fields and structures that can be used. ⁤⁤The Elliptic Curve Fourier Transform (ECFFT) addresses this by enabling the use of elliptic curves as a domain for the Fourier transform.

Elliptic curves provides smooth subgroups for its transformation making it more applicable to broader algebraic settings including those with non-smooth fields thus eliminating the requirement that fields having specific characteristics required for classical FFTs. ⁤


While ECFFT-STARKs are quite powerful, they come with complexity. Circle STARK, however, leverages the simplicity of the Circle Curve—a straightforward and elegant mathematical structure. By using the Circle Curve, Circle STARK simplifies the construction and computation of cryptographic proofs, making them more accessible and efficient without sacrificing security.

**Table Of Contents:**
[toc]

---

# Overview of STARKs and FFT

First, let's recall STARKs to understand why FFT (Fast Fourier Transform) plays a crucial role in them.

STARKs are cryptographic proof systems designed to enable the prover to convince another party, the verifier, that a computation has been performed correctly, without revealing the actual data.

In the context of univariate STARKs, the witness data (the information needed to verify the proof) is represented as polynomials with one variable (hence, univariate). A univariate domain is a set of values over which these polynomials are evaluated. This encoding is essential for performing operations like polynomial evaluation, interpolation, and the fast Fourier transform (FFT), which are used in the STARK protocol.

One of the key steps in STARKs is the low-degree test. This test ensures that the polynomial used in the proof is of a sufficiently low degree, which is necessary for the security  proof. The test involves evaluating the polynomial at several points and checking that the results are consistent with a low-degree polynomial. For more information, I recommend this blogpost.

In STARKs, polynomial interpolation is a fundamental operation. Polynomial interpolation involves finding a polynomial that passes through a given set of points. This is essential for STARKs because it allows for efficient encoding and manipulation of data. FFT is used to perform these interpolations quickly and efficiently. The Fast Fourier Transform (FFT) is a cornerstone in the field of cryptographic proofs. It allows for efficient polynomial interpolation and evaluation, which are essential for constructing and verifying proofs. The Circle FFT is a specialized version of FFT that leverages the simplicity of the Circle Curve to achieve more efficient and scalable computations.

Now that we have a basic understanding of STARKs, let's explore how Circle STARKs build on these principles. Circle STARKs follow the same steps as traditional STARKs but use the Circle Curve for polynomial interpolation and FFT, offering a simpler and more efficient approach.

## Benefits of the Circle Curve Approach

The Circle Curve has several  advantages that make it a compelling choice for cryptographic applications like Circle STARK.
1. **Quadratic Group Squaring Map:** One of the key features of the Circle Curve is its quadratic group squaring map. This map, which takes an element and maps it to its square, has a quadratic degree. Notably, each point on the Circle Curve has exactly two preimages when the squaring map is applied, creating a 2-to-1 map. This property simplifies the mathematical structure significantly compared to elliptic curves, which often require a series of interconnected curves to achieve similar results.

2. **Rotation and Polynomial Basis:** Another  benefit is related to the rotation and polynomial basis of the Circle Curve. When points on the Circle Curve are rotated, the resulting points can still be described by polynomials. This rotation-preserving property allows for the use of a polynomial basis for the Fast Fourier Transform (FFT) instead of rational functions. While the polynomial basis used here is not the standard monomial basis, it provides a foundation for efficient transformations, even though converting between these bases efficiently is a challenge that has yet to be fully addressed.
3. **Simplicity:** The simplicity of the Circle Curve is one of its most attractive features. Unlike traditional methods that require advanced algebraic geometry concepts like divisor calculus and the Riemann-Roch theorem, the Circle Curve approach avoids these complexities. Instead, it uses the projective line view, treating the Circle Curve as a projective line, which is easier to understand and work with. Additionally, the Circle Curve is a genus zero curve in algebraic geometry, meaning it has no "holes" (similar to how a sphere has no holes compared to a donut). This simplicity translates to more accessible and straightforward implementations for developers and researchers.

By leveraging these advantages, the Circle Curve approach provides a simpler, yet powerful, foundation for cryptographic proofs, making the construction and computation of proofs more efficient and scalable.


# ECFFT-Friendly Primes

A **CFFT-friendly prime** $p$ is a prime number such that $p + 1$ is divisible by $2^{n+1}$ for sufficiently large $n \geq 1$. In the context of Cirle STARK , $n$ is called a supported order, and $N = 2^n$ is a supported domain size.

The specific value of $n$ that is considered sufficiently large depends on the application. In the context of STARKs, we typically look for a small prime $p$ where $p + 1$ is as smooth as possible. One compelling choice is the ***Mersenne prime***

$$
p = 2^{31} - 1,
$$

known as **M31** , which is favored for its exceptionally efficient arithmetic. This prime is teh main target in Circle STAK, but other ECFFT-friendly primes can also be useful in practice. For more details on Mersenne primes in Circle STARKs, you can find a great post [here](https://www.zksecurity.xyz/blog/posts/circle-starks-1/).

ECFFT-friendly primes is $p = 2^{n+1} \cdot t - 1$ for integers $n$ and $t \geq 1$. Specifically, this means $p - 1 = 2 \cdot t'$ where $t' = 2^n \cdot t - 1$ is odd, or equivalently $p \equiv 3 \mod 4$, indicating that $-1$ does not have a square root modulo $p$. For any such prime, the polynomial $x^2 + 1$ has no roots in the prime field $\mathbb{F}_p$.


# Transition from Traditional STARKs to Circle STARKs

In traditional STARKs, computations are performed within a cyclic group in the field $\mathbb{F}_p$. This approach works well but comes with some complexities, especially when dealing with larger field sizes and specific algebraic structures. Circle STARKs simplify this process by operating on the Circle Curve $C(\mathbb{F}_p)$, a more straightforward and elegant structure.

### Working with the Circle Curve $C(\mathbb{F}_p)$

The Circle Curve $C(\mathbb{F}_p)$ is defined by the equation, $x^2 + y^2 = 1$, over the finite field $\mathbb{F}_p$. This set of points forms a group under a specific group law, which we will now explain.

$$
C(\mathbb{F}_p) = \{ (x,y) : x, y\in \mathbb{F}_p, x^2 + y^2 = 1 \mod p\}
$$

### Group Structure of $C(\mathbb{F}_p)$

To show that $C(\mathbb{F}_p)$ is a group, we need to define a group operation and verify that it satisfies the group axioms: closure, associativity, identity, and inverses.

**Group action:** The group action on $C(\mathbb{F}_p)$ is defined as follows:
$$
P \cdot Q = (x_1, y_1) \cdot(x_2, y_2) =  = (x_1 x_2 - y_1 y_2, x_1 y_2 + x_2 y_1).
$$
**NOTICE:** The group action is in fact similar to the complex number multiplications $(x_1 + i\cdot y_1) \cdot (x_2 + i\cdot y_2)$. 

:::spoiler
***Additional Note:***

1. **Closure:**
$$
(x_1 x_2 - y_1 y_2)^2 + (x_1 y_2 + x_2 y_1)^2 = 1 \mod p.
$$

2. **Identity Element** is $(1, 0)$:
$$
(x, y) \cdot(1,0) = (x,y)
$$

3. **Inverse Element** for  point $P = (x, y)$  is given by $(x, -y)$. This is because $P + (-P) = (1, 0)$.

4. **Associativity:** For any three points $P, Q,$ and $R$ on $C(\mathbb{F}_p)$, we have $(P \cdot Q) \cdot R = P \cdot (Q \cdot R)$.
:::

### Advantages of Using $C(\mathbb{F}_p)$

We have the following advantages for using the Unit circle in $\mathbb{F}_p$ such as:

1. **Simplicity**: The algebraic operations on the Circle Curve are simpler compared to those in elliptic curves, leading to faster computations and easier implementation.
2. **Efficiency**: The Circle Curve allows for efficient polynomial interpolation and transformation, which are crucial for the Circle STARK-FFT algorithm.
3. **Accessibility**: The straightforward nature of the Circle Curve makes it more accessible to a broader range of developers and researchers, reducing the barrier to entry for implementing advanced cryptographic protocols.

By leveraging the Circle Curve $C(\mathbb{F}_p)$, Circle STARKs simplify the construction and computation of cryptographic proofs, making them more efficient and scalable.


 [Definition 2:](https://eprint.iacr.org/2024/278.pdf) Let $G_{n-1}$ be a cyclic subgroup of the circle group $C(\mathbb{F}_p)$ with size $|G_{n-1}| = 2^{n-1}$ for $n \geq 1$. 
 
 A twin-coset of size $N = 2^n$ is defined as a union of two disjoint sets, $Q \cdot G_{n-1}$ and $Q^{-1} \cdot G_{n-1}$, where $Q$ is an element in $C(\mathbb{F}_p)$ and:
 - $Q \cdot G_{n-1}$ and $Q^{-1} \cdot G_{n-1}$ do not overlap.
 - If these sets overlap, the union $D$ is simply a coset of the larger subgroup $G_n$, which is called a **standard position coset** of size $N$.

:::spoiler
Add more explanation of Twin-Coset here, also need to put the figure 1 in paper with explanation. The figure  shows  the concept of standard position cosets and twin-cosets on the circle curve. Each circle represents a cyclic subgroup, with blue points as elements of the subgroup and red points as their inverses.
:::

![image](https://hackmd.io/_uploads/SJd9SdAwC.png)
:::danger
TBC...
:::

# FFT and Circle FFT

The FFT is an efficient way of converting between the coefficient and evaluation representations of a polynomial. The key idea behind the FFT is to recursively divide the DFT into smaller FFTs. Each round of the FFT reduces the evaluation into a problem only half the size. Most commonly, we use polynomials of length some power of two, $n = 2^k$, and apply the halving reduction recursively.

The most common FFT algorithm is the Cooley-Tukey algorithm, which has three main steps:

1. **Divide the Sequence**: Split the sequence into two halves, the even-indexed and odd-indexed elements. For example, for $f(x) = p_0 + p_1 x + p_2 x^2 + \cdots + p_7 x^7$, the even and odd polynomials would be:

   $$
   \begin{aligned}
   f_{\text{even}}(x) &= p_0 + p_2 x^2 + p_4 x^4 + p_6 x^6, \\
   f_{\text{odd}}(x) &= p_1 x + p_3 x^3 + p_5 x^5 + p_7 x^7.
   \end{aligned}
   $$

2. **Recursive Computation**: Compute the DFT of the even-indexed and odd-indexed elements separately:

   $$
   f_{\text{even}} = \text{FFT}(x_{\text{even}}), \quad f_{\text{odd}} = \text{FFT}(x_{\text{odd}}).
   $$

3. **Combine Results**: Combine the results of the smaller DFTs to get the final DFT:

   $$
   \begin{aligned}
   X_k &= X_{\text{even}, k} + e^{-i 2\pi k / N} \cdot X_{\text{odd}, k}, \\
   X_{k + N/2} &= X_{\text{even}, k} - e^{-i 2\pi k / N} \cdot X_{\text{odd}, k},
   \end{aligned}
   $$

   for $k = 0, 1, \ldots, N/2 - 1$.
 
 
 :::info
 
 
### Number Theoretic Transform (NTT)

While the traditional FFT operates over complex numbers, the Number Theoretic Transform (NTT) is used for polynomials over finite fields, which is particularly relevant in cryptographic applications such as STARKs. The NTT is similar to the FFT but uses an $N$-th root of unity $\omega$ in a finite field $\mathbb{F}_q$.

In NTT, the combination step uses $\omega$ instead of complex exponentials:

$$
\begin{aligned}
X_k &= X_{\text{even}, k} + \omega^k \cdot X_{\text{odd}, k}, \\
X_{k + N/2} &= X_{\text{even}, k} - \omega^k \cdot X_{\text{odd}, k},
\end{aligned}
$$



for $k = 0, 1, \ldots, N/2 - 1$, where $\omega$ is a primitive $N$-th root of unity in the finite field $\mathbb{F}_q$.



The NTT is crucial for efficient polynomial multiplication in cryptographic protocols, as it allows operations to be performed in a finite field rather than over the complex numbers, which is more suitable for discrete computations required in cryptography.
:::

## Circle FFT

In both cases, FFT and EC-FFT, the idea is to halve down the problem into smaller parts, apply the transformation recursively, and then combine the results. In the traditional FFT, a polynomial $P(x)$ of degree $n-1$ is split into two smaller polynomials based on the parity of the coefficients:

- $P_{\text{even}}(x)$: Formed by the even-indexed coefficients.
- $P_{\text{odd}}(x)$: Formed by the odd-indexed coefficients.

The FFT then recursively computes the DFT of these smaller polynomials, leveraging the fact that this splitting reduces the problem size by half at each recursion level.

Similarly, in EC-FFT and Circle FFT, the mappings play a similar role to the splitting of polynomials into odd and even coefficients in the traditional FFT.

Similarly, in EC-FFT and Circle FFT, the mappings (such as the 2-to-1 maps in EC-FFT and the twin-coset structure in Circle FFT) reduce the problem size in a structured way.


 Decomposition in Circle STARK: REcal domain and cosets. The first step in the process involves decomposing a function $f(x, y)$ into even and odd parts:

1. **Even Part $f_{\text{even}}(x)$**:
   $$
   f_{\text{even}}(x) = \frac{f(x, y) + f(x, -y)}{2}
   $$
   This formula takes the average of $f(x, y)$ and $f(x, -y)$. Since this average is invariant under the involution (flipping $y$ to $-y$), it captures the even part of the function.

2. **Odd Part $f_{\text{odd}}(x)$**:
   $$
   f_{\text{odd}}(x) = \frac{f(x, y) - f(x, -y)}{2y}
   $$
   This formula computes the difference between $f(x, y)$ and $f(x, -y)$, scaled by $\frac{1}{2y}$. This captures the odd part of the function, which changes sign under the involution.

3. **Reconstruction**:
   $$
   f_{\text{even}}(x) + y \cdot f_{\text{odd}}(x) = f(x, y)
   $$
   The original function $f(x, y)$ can be reconstructed by combining the even and odd parts.
:::info
** 2-to-1 Maps:** In Circle STARKs, the Circle Curve $C(\mathbb{F}_p)$ replaces the elliptic curves used in ECFFT. The group squaring map on the Circle Curve is of quadratic degree and is 2-to-1, meaning each point on the curve has exactly two preimages. This property simplifies the structure and avoids the need for a chain of interconnected curves.
:::


To efficiently perform the Circle FFT, we utilize the twin-coset structure. The domain $D$ for the FFT is defined as:

$$
D = Q \cdot G_n \cup Q^{-1} \cdot G_n,
$$

where $G_n$ is a proper subgroup of the Circle Curve $C(\mathbb{F}_p)$ of size $N = 2^n$, and $Q$ is an element of $C(\mathbb{F}_p) \setminus G_n$.

The twin-coset structure ensures that the domain $D$ is invariant under the involution $J$, which maps any point $(x, y)$ on the Circle Curve to $(x, -y)$. This invariance is crucial for the efficiency and correctness of the FFT.

Polynomial Basis : In Circle FFT, we use a specific polynomial basis for interpolation. The basis polynomials $b_j^{(n)}(x, y)$ are defined as:

$$
b_j^{(n)}(x, y) := y^{j_0} \cdot v_1(x)^{j_1} \cdots v_{n-1}(x)^{j_{n-1}},
$$

where $v_k(x)$ are vanishing polynomials. These polynomials evaluate to zero over specific subsets of the Circle Curve, ensuring that the interpolation and transformation are efficient.

## Example of Circle FFT
:::warning
The credit of the example goes to [lecture](https://www.youtube.com/watch?v=NAhLYMysSdk&t=839s) althoug, more computation has been added.
:::
Let's consider an example over $\mathbb{F}_7 = \{0, 1, 2, 3, 4, 5, 6\}$  to illustrate these steps.
$$C(\mathbb{F}_7) = \{(0,1),(1,0) , ( 0,6) , (6,0) , (2,2) ,(5,5), (2,5) , (5,2)\}$$ 

:::spoiler
Add the computation
:::

To demonstrate, let’s start with the decomposition of eight values:

1. **Step 1**: Split the polynomial into even and odd parts.
  - The average of symmetric points (e.g., points 2 and 8) gives one part of the polynomial, while their difference gives another part.
  
2. **Step 2**: Further decompose each part.
  - Continue this process, decomposing each polynomial until reaching polynomials of single values (constants).

3. **Field Operations**: 
  - Throughout the process, only basic field operations are used (e.g., taking averages, differences, and dividing by field elements), with no need for complex group operations.

Let's walk through the computations step by step using the decomposition formulas to find $f_0(x)$ and $f_1(x)$ based on the points in the original circle.

Given curve pints $\{(A,B), \dots, (-A,-B)\}$ with the following evaluations:
| $(A, B)$ | $(B, A)$ | $(-A, B)$ | $(-B, A)$ | $(A, -B)$ | $(B, -A)$ | $(-A, -B)$ | $(-B, -A)$ |
|------------|--------|--------|---------|---------|---------|---------|----------|
| $f_1$ | $f_2$ | $f_3$  | $f_4$  | $f_5$  | $f_6$  | $f_7$   | $f_8$   |
|1|1|3|2|21|13|5|8|





### Compute $f_0(x)$

$f_0(x)$ is computed using the formula:

$$
f_0(x) = \frac{f(x, y) + f(x, -y)}{2}
$$

Let's compute $f_0(x)$ for each point:

1. $f_0(A) = \frac{f(A, B) + f(A, -B)}{2} = \frac{f_1 + f_5}{2}$
2. $f_0(B) = \frac{f(B, A) + f(B, -A)}{2} = \frac{f_2 + f_6}{2}$
3. $f_0(-A) = \frac{f(-A, B) + f(-A, -B)}{2} = \frac{f_3 + f_7}{2}$
4. $f_0(-B) = \frac{f(-B, A) + f(-B, -A)}{2} = \frac{f_4 + f_8}{2}$

### Compute $f_1(x)$



$f_1(x)$ is computed using the formula:

$$
f_1(x) = \frac{f(x, y) - f(x, -y)}{2y}
$$

Let's compute $f_1(x)$ for each point:

1. $f_1(A) = \frac{f(A, B) - f(A, -B)}{2B} = \frac{f_1 - f_5}{2B}$
2. $f_1(B) = \frac{f(B, A) - f(B, -A)}{2A} = \frac{f_2 - f_6}{2A}$
3. $f_1(-A) = \frac{f(-A, B) - f(-A, -B)}{2B} = \frac{f_3 - f_7}{2B}$
4. $f_1(-B) = \frac{f(-B, A) - f(-B, -A)}{2A} = \frac{f_4 - f_8}{2A}$

Using the values from the figure:

1. For point $(A, B)$ where $A = 1$ and $B = 2$:
   - $f_0(A) = \frac{3 + (-10)}{2} = -3.5$
   - $f_1(A) = \frac{3 - (-10)}{2 \cdot 2} = 3.25$

2. For point $(B, A)$ where $A = 2$ and $B = 1$:
   - $f_0(B) = \frac{-3 + 11}{2} = 4$
   - $f_1(B) = \frac{-3 - 11}{2 \cdot 1} = -7$

### Resulting Points on the Right Circle

Let's summarize the calculations for each resulting point:

- $f_0(A) = -3.5$
- $f_0(B) = 4$
- $f_0(-A) = -0.5$
- $f_0(-B) = 1.5$

- $f_1(A) = 3.25$
- $f_1(B) = -7$
- $f_1(-A) = 2.75$
- $f_1(-B) = 2.5$

So, we now have the points for $f_0(x)$ and $f_1(x)$ as follows:

#### Even Part $f_0(x)$

- $f_0(A) = -3.5$
- $f_0(B) = 4$
- $f_0(-A) = -0.5$
- $f_0(-B) = 1.5$

#### Odd Part $f_1(x)$

- $f_1(A) = 3.25$
- $f_1(B) = -7$
- $f_1(-A) = 2.75$
- $f_1(-B) = 2.5$

This completes the decomposition and mapping for the Circle STARK example. The points on the right circles are the even and odd parts obtained from the original points on the left circle.

:::warning
The following comoputation is going to be simplified based on the above indexing
:::

$$
\begin{align*}
&f(x, y) = \\
&\text{// First decomposition} \\
&f_0(x) + y f_1(x) = \\
&\text{// Second decomposition} \\
&[f_{00}(2x^2 - 1) + x f_{01}(2x^2 - 1)] + y [f_{10}(2x^2 - 1) + x f_{11}(2x^2 - 1)] = \\
&\text{// Third decomposition} \\
&[(f_{000} + (2x^2 - 1) f_{001}) + x (f_{010} + (2x^2 - 1) f_{011}) +
y [(f_{100} + (2x^2 - 1) f_{101}) + x (f_{110} + (2x^2 - 1) f_{111})]] = \\
&\text{// Expand the terms} \\
&f_{000} + f_{001} (2x^2 - 1) + f_{010} x + f_{011} (2x^2 - 1) x + \\
&f_{100} y + f_{101} y (2x^2 - 1) + f_{110} yx + f_{111} y (2x^2 - 1) x = \\
&\text{// Substitute our values} \\
&\frac{27}{4} + \frac{3}{4C} (2x^2 - 1) + \frac{-11A + 9C}{4AB} x + \frac{-11A + 9C}{4ABC} x(x^2 - 1) + \cdots
\end{align*}
$$


:::spoiler
Additional Note: 
Notice that the Circle FFT simplifies the encoding and polynomial arithmetic and despite its simplicity, it remains efficient and avoids complex operations typically seen in elliptic curve computations. also unlike ECFFT, which uses rational functions, Circle FFT only involves polynomials, which agin simplify computations.
:::



# Inverse Circle FFT
will be added


# Circle FFT Algorithm

The Circle FFT algorithm has a structure similar to the standard Cooley-Tukey FFT algorithm, but with a touch of the Circle Curve's algebraic framework. In this section we go through the algorithm step by step.


```python
1: procedure CFFT(a)
2:     # First layer
3:     step ← N/2
4:     for k in range(step) do
5:         twiddle ← (QG^k).y
6:         butterfly(a[k], a[k + step], twiddle)
7:     
8:     # Rest of the layers
9:     for l in range(log(N) - 2) do
10:         step ← 2^(log(N)-2-l)
11:         for i in range(0, N, 2 * step) do
12:             for k in range(step) do
13:                 twiddle ← ((QG^k)^(2^l)).x
14:                 butterfly(a[i + k], a[i + k + step], twiddle)
15: 
16: procedure BUTTERFLY(a, b, t)
17:     a, b ← a + b, (a - b)/t
```

#### Phase 1. Initialization

No significant operations are performed in this step apart from setting up parameters.

#### Phase 2. First Layer Transformation

In the first layer, the algorithm processes the data in steps of size $N/2$. For each pair of data points, we apply the butterfly operation. It has $N/2$ of pairs including 2 additions (one for the sum, one for the difference) and 1 multiplication for applying the twiddle factor. So in total  $N/2 \times 2 = N$ additions and $N/2$ multiplications.


#### Phase 3. Recursive Transformation

The recursive transformation is applied for $\log(N) - 2$ layers. In each layer, the problem size is halved. For each layer $l$ the step size is  $2^{(\log(N) - 2 - l)}$ and  the number of pairs per Sub-FFT is $N / 2^{l+1}$. This makes the number  of Sub-FFTs $2^l$ and the number of pairs $(N / 2^{l+1}) \times 2^l = N / 2$ in total.

Now notice that in each sub-FFT, there are 2 additions and 1 multiplication per pair.

```python
# Rest of the layers
for l in range(log(N) - 2) do
  step ← 2^(log(N) - 2 - l)
  for i in range(0, N, 2 * step) do
    for k in range(step) do
      twiddle ← ((QG^k)^(2^l)).x
      butterfly(a[i + k], a[i + k + step], twiddle)
```
Therefore, we have $(N / 2) \times 2 = N$ additions per layer and  $N / 2$ multiplications per layer.


Summing the operations across all layers, including the first layer:



| Total Layers | Total Additions | Total Multiplications |
| -------- | -------- | -------- |
| $\log(N) - 1$      | $N \times (\log(N) - 1)$    | $(N / 2) \times (\log(N) - 1)$     |

Note that in the `butterfly` algorithm there are 2 additions and 1 division. Here we assume 2 additions and 1 multiplication , assuming we can find the inverse of the field element. 

Based on the above analysis, the Circle FFT algorithm, like the traditional FFT, achieves a logarithmic complexity in terms of the number of additions and multiplications required. Specifically, it requires $N \times (\log(N) - 1)$ additions and $(N / 2) \times (\log(N) - 1)$ multiplications. This efficiency makes it suitable for applications in cryptographic proofs where large-scale polynomial transformations are needed.


# 7. Optimization of Circle FFT Implementation

There are some ways to optimize  the efficiency of the FFT algorithm. Here are some suggestions:

1. **Montgomery Reduction**: Add link and some example.
2. **Optimizing Power Computation**: Efficiently compute successive powers (e.g., $x, x^2, x^4, \ldots, x^n$) rather than repeatedly multiplying $x$ raised to the power $n$. This approach reduces the number of multiplications required. (link it to the code)

3. **Precompute Twiddle Factors**: Calculate and store all necessary twiddle factors before initiating the main FFT loop. This precomputation minimizes redundant calculations, thereby expediting the butterfly operations. :1234: link it to the block code

4. **Vectorized Operations** (specifically in Rust programming): Utilize vectorized operations to perform multiple additions and multiplications concurrently, leveraging parallelism to boost performance. (put a concret example here)

5. **Multithreading**: Implement multithreading to parallelize independent butterfly operations. Since each layer's operations are independent, they can be executed in parallel, significantly speeding up the process. ( implement the bench mark and add a table with concret N)





# Implemention Circle FFT in Rust

Here,  we show a simple Rust code example to demonstrate how to implement the CircleFFT interface using the `num-bigint` and `num-traits` crates for handling large integers and numeric traits respectively. The `CircleFFT` trait encompasses several methods crucial for performing FFT operations in a field $\mathbb{F}_p$, including the FFT and inverse FFT processes, precomputing twiddle factors, computing powers, and executing butterfly operations—all essential for efficient FFT algorithms. The provided code outlines the structure and placeholder methods of the `MyFFT` struct, which serves as a starting point for further customization and actual implementation of FFT algorithms tailored to specific needs.


Here's a Rust code example incorporating some of these optimizations:

We need to define the circle group $C(\mathbb{F}_p)$
```rust
pub trait C<F> {
    // Define the addition operation
    fn add(self, other: Self) -> Self;

    // Define the subtraction operation
    fn sub(self, other: Self) -> Self;

    // Define the multiplication operation
    fn mul(self, other: Self) -> Self;

    // Define the division operation
    fn div(self, other: Self) -> Self;

    // Define the multiplicative inverse operation
    fn inv(self) -> Self;

    // Define the exponentiation operation
    fn pow(self, exp: u64) -> Self;

    // Define a method to check if the element is zero
    fn is_zero(self) -> bool;

    // Define a method to return the zero element of the field
    fn zero() -> Self;

    // Define a method to return the one element of the field
    fn one() -> Self;
}
```
and then define the trait `circleFFT`

```rust
use num_bigint::BigUint;

trait CircleFFT {
    
    /// Perform the FFT on a given set of data in the field F_p.
    fn fft(&self, data: &mut [BigUint], p: &BigUint);

    /// Perform the inverse FFT on a given set of data in the field F_p.
    fn ifft(&self, data: &mut [BigUint], p: &BigUint);
    
    /// Precompute the twiddle factors for the FFT in the field F_p.
    fn precompute_twiddle_factors(&self, size: usize, p: &BigUint) -> Vec<BigUint>;

    /// Optimize the computation of powers in the field F_p.
    fn compute_powers(&self, base: &BigUint, exponent: usize, p: &BigUint) -> BigUint;

    /// Perform the butterfly operations in the FFT algorithm.
    fn butterfly(&self, a: &BigUint, b: &BigUint, twiddle: &BigUint, p: &BigUint) -> (BigUint, BigUint);
}

struct StarkCircleFFT;

impl CircleFFT for StarkCircleFFT {
    fn fft(&self, data: &mut [BigUint], p: &Bi
        todo!();
    }

    fn ifft(&self, data: &mut [BigUint], p: &BigUint) {
        // Implementation of inverse FFT algorithm
        todo!();
    }

    fn precompute_twiddle_factors(&self, size: usize, p: &BigUint) -> Vec<BigUint> {
        // Implementation of twiddle factors precomputation
        todo!();
        Vec::new()
    }

    fn compute_powers(&self, base: &BigUint, exponent: usize, p: &BigUint) -> BigUint {
        let mut result = BigUint::one();
        let mut base = base.clone();
        let mut exp = exponent;

        while exp > 0 {
            if exp % 2 == 1 {
                result = (result * &base) % p;
            }
            base = (&base * &base) % p;
            exp /= 2;
        }

        result
    }

    fn butterfly(&self, a: &BigUint, b: &BigUint, twiddle: &BigUint, p: &BigUint) -> (BigUint, BigUint) {
       todo!();
        (a_new, b_new)
    }
}
```

# Conclusion 
In the next blog post, we will delve into the Circle FRI and the entire Circle STARK protocol. Additionally, we will provide some implementations of the Circle FFT algorithm in Rust. Stay tuned !
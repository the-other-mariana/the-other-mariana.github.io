---
layout: post
title:  "The Color Signal Processing"
date:   2023-02-02 10:51:13 -0600
categories: signal processing
---

## Signal Spectrum

The **signal spectrum** describes a **signal's magnitude and phase** characteristics as a **function of frequency**. The system spectrum describes how the system changes signal magnitude and phase as a function of frequency. For example, At the lower frequencies, below around 80 Hz, the magnitude spectrum is 1.0.

**Frequency refers to the number of times an event occurs in a given period of time**. In the context of functions, it refers to the number of times the graph of a function repeats itself in a given amount of time.

The Frequency Domain refers to the analytic space in which mathematical functions or signals are conveyed in terms of frequency, rather than time. For example, where a time-domain graph may display changes over time, a frequency-domain graph displays how much of the signal is present among each given frequency band.

It is possible, however, to convert the information from a time-domain to a frequency-domain. An example of such transformation is a Fourier transform.

- The Fourier transform converts the time function into a set of sine waves that represent different frequencies.
    
- The frequency-domain representation of a signal is known as the "spectrum" of frequency components.

![img]({{site.url}}/img/2/1.png)

- In the image above, we see the green function in the time domain is the result of the sum of the pink, purple and blue functions, which have 3 different frequencies. The green function has the biggest frequency with the value of the axis point where the pink function is.

The **spectrum of a signal** refers to the distribution of its energy across different frequency components. It provides information about the frequency content of the signal and can be represented as a **plot of the magnitude or power of the frequencies** present in the signal.

> The spectrum can be obtained using techniques such as Fourier transform, short-time Fourier transform, or wavelet transform.

Given that a frequency-domain graph displays **how much of the signal is present among each given frequency** band, then we can say that the **spectrum of a signal** can be represented as a **frequency domain graph of a signal**. We can generally say that **power density spectra = signal spectra**.

## The Set P Of Physical Light Stimuli

Any physical light (signal) power density spectrum can be written as the sum of two terms:

1. One whose integrated spectrum is non-decreasing, continuous and bounded.

2. One whose integrated spectrum is a non-decreasing step function with a finite number of steps.

If $F(\lambda)$ is the integrated spectrum, then 

$$
F(\lambda) = \int_{\lambda_{min}}^{\lambda} f(\mu) d\mu
$$

If we say f(\mu) is non-negative, then $F(\mu)$ is non-decreasing. 

A **signal space** is a set of all functions $f$ such that $F(\mu)$ can be written as $F(\mu) = F_c(\mu) + F_d(\mu)$ where $F_c(\mu)$ is *absolutely continuous* and $F_d(\mu)$ is a step function with steps at a finite set of wavelengths $\mu_1, \mu_2, ..., \mu_k$.

Two lights, with power spectral densities $f_1(\mu)$ and $f_2(\mu)$, that are superimposed, generate a resulting light ray whose power spectral density is $f_1(\mu) + f_2(\mu)$. Given this we can start to impose some algebraic structure on the set $P$ (set of power density spectra).

----

### Theorem 2.1

- ($P$, +) is a commutative semigroup with a neutral element (Abelian Monoid) in which every element is cancellable.

	- A semigroup is a set equipped with a binary, associative operation. The associative operation is the superposition of light rays, equivalent to addition of power density spectra.

- Thus we conclude that $f_1 + f_2 \in P$ so that $P$ is closed under the binary operation. This operation is also associative and commutative. The function $f(\lambda) = 0$ is the nuetral element. An element $f$ is cancellable if for all $f_1, f_2 \in P$:

$$
f + f_1 = f + f_2
$$ 

----

It is not possible to express the difference between two spectra as elements of this space $P$. Thus, we extend $P$ by embedding it in a larger algebraic structure, space $A$, that contains it. $A$ consists of all the sums and differences of elements of $P$. Therefore, every element in $P$ has an additive inverse in $A$. A group is a semigroup with a neutral element such taht every element is invertible. The group $A$ is called the inverse completion of ($P$, +).

----

### Theorem 2.2

- The semigroup $P$ of physical light rays can be embedded in a unique, minimal commutative group $A$ such that every elementof $P$ has an inverse in $A$. Then, $A = P + P*$.

----

## The Color Vector Space

Different spectral densities can give rise to identical color sensations, based on the spectral sensitivity of the cone receptors in the human retina. If two spectral densities, $C_1(\lambda)$ and $C_2(\lambda)$, produce an **identical color sensation** for a human viewer under the same viewing conditions, they form a **metameric pair**. We denote this by

$$
C_1(\lambda) \triangle C_2(\lambda)
$$

This equation applies to the two spectral densities as a whole, and not to a particular wavelength: $C_1(\lambda) \triangle C_2(\lambda), C_1, C_2 \in P$. This condition of metamerism applies to a *specific human observer* and it may be very different for other seeing entities, such as robots, cameras, etc.

The mathematical structure for color measurement is often referred to as **Grassmann's Laws**, labelled as G1, G2, etc.

### Properties of Metamerism

Metamerism is a relation on the set $P$. The first property is **Transivity (G1)**:

> If $C_1(\lambda) \triangle C_2(\lambda)$ and $C_2(\lambda) \triangle C_3(\lambda)$, then $C_1(\lambda) \triangle C_3(\lambda)$.

For any $C(\lambda) \in P$, we denote

$$
[C] = {C_1(\lambda) \in P | C_1(\lambda) \triangle C_1(\lambda)},
$$

the set of all spectral densities metamerically equivalent to $C(\lambda)$; this is an equivalence class. Metameric classes are a partition of $P$. Some of these classes may contain a single element of $P$, such as 0, while most will contain an infinite number of elements. We refer to $[C]$ as a **color**.

## References

Dubois, E. (2010). The Structure and Properties of Color Spaces and the Representation of Color Images. Springer.
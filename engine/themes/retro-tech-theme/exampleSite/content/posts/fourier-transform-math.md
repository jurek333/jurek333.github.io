---
title: "Understanding Fourier Transforms in Digital Signal Processing"
date: 2026-01-20
author: "jurek"
tags: ["math", "dsp", "signal-processing", "fourier"]
---

When working with audio circuits or analyzing waveforms from retro computers, understanding the Fourier Transform is essential. Let's dive into the math!

## The Discrete Fourier Transform

The Discrete Fourier Transform (DFT) converts a finite sequence of samples into a sequence of frequencies. It's defined as:

$$
X_k = \sum_{n=0}^{N-1} x_n \cdot e^{-i 2\pi k n / N}
$$

where $N$ is the number of samples, $x_n$ are the input samples, and $X_k$ are the frequency components.

## Euler's Formula

The magic behind the DFT lies in Euler's formula, which connects complex exponentials to trigonometric functions:

$$
e^{i\theta} = \cos(\theta) + i\sin(\theta)
$$

This means we can rewrite the DFT as:

$$
X_k = \sum_{n=0}^{N-1} x_n \left[\cos\left(\frac{2\pi kn}{N}\right) - i\sin\left(\frac{2\pi kn}{N}\right)\right]
$$

## Frequency Resolution

The frequency resolution of the DFT depends on the sampling rate $f_s$ and the number of samples:

$$
\Delta f = \frac{f_s}{N}
$$

For example, with a sampling rate of 44.1 kHz and 1024 samples, we get a resolution of approximately $\Delta f = 43$ Hz.

## The Fast Fourier Transform

Computing the DFT directly has a complexity of $O(N^2)$. The Fast Fourier Transform (FFT) algorithm reduces this to $O(N \log N)$ by exploiting symmetries in the calculation.

The key insight is the **Cooley-Tukey algorithm**, which recursively breaks down a DFT of size $N$ into two DFTs of size $N/2$:

$$
X_k = \sum_{m=0}^{N/2-1} x_{2m} \cdot e^{-i 2\pi k (2m) / N} + \sum_{m=0}^{N/2-1} x_{2m+1} \cdot e^{-i 2\pi k (2m+1) / N}
$$

## Practical Example

In a Z80-based audio analyzer, you might implement a simple 8-point FFT. The butterfly diagram shows how the algorithm reuses calculations:

```
Input:  x[0] x[1] x[2] x[3] x[4] x[5] x[6] x[7]
         |    |    |    |    |    |    |    |
Stage 1: +----+    +----+    +----+    +----+
         |    |    |    |    |    |    |    |
Stage 2: +----+----+----+    +----+----+----+
         |    |    |    |    |    |    |    |
Stage 3: +----+----+----+----+----+----+----+
         |    |    |    |    |    |    |    |
Output: X[0] X[1] X[2] X[3] X[4] X[5] X[6] X[7]
```

## Power Spectrum

To analyze signal strength at different frequencies, we calculate the power spectrum:

$$
P_k = |X_k|^2 = \text{Re}(X_k)^2 + \text{Im}(X_k)^2
$$

This gives us the energy at each frequency bin, perfect for visualizing on LED displays or oscilloscopes!

## Conclusion

The Fourier Transform bridges time-domain signals and frequency-domain analysis. Whether you're building a spectrum analyzer for your vintage computer or analyzing cassette tape signals, understanding these equations is crucial.

Next time: Implementing an FFT in Z80 assembly! 🎵

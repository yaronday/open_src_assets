# Argandly — Complex Roots Visualizer

A high-performance, cross-platform Flutter application tailored for mathematicians, engineers, physicists, and researchers to compute, analyze, and visualize the $n$-th roots of complex numbers and the roots of univariate complex polynomials within the complex plane.

---

## Overview

Argandly bridges pure numerical computation with interactive geometric analysis through two primary operational engines:

1. **Number Mode**: Evaluates an arbitrary complex number $z \in \mathbb{C}$ to find its $n$-th roots, mapping their symmetric distribution along an origin-centered circumscribed circle.
2. **Polynomial Mode**: Solves for the complex roots of univariate polynomials $P(z) = \sum_{k=0}^{n} c_k z^{n-k}$ where $c_k \in \mathbb{C}$, featuring real-time algebraic manipulation (differentiation, integration, and exponentiation).

---

## Key Features

### Computational Engines

- **Arbitrary Root Extraction**: Extracts up to $N = 20$ roots per complex value using automated input-aware memoization via an LRU cache to isolate execution from the UI thread.
- **High-Degree Polynomial Solver**: Locates roots for complex polynomials up to degree $N$ using an accelerated, sequential Aberth-Ehrlich numerical engine.
- **Calculus & Algebraic Operators**:
  - **Differentiate**: Computes the symbolic first derivative $\frac{dP}{dz}$.
  - **Integrate**: Evaluates the indefinite integral $\int P(z)\,dz$ (assuming $C = 0$).
  - **Power**: Computes $P(z)^k$ via algebraic expansion up to the maximum degree boundary.
- **Bi-Directional Synthesis**: Synthesizes polynomial coefficients dynamically from an arbitrary set of user-defined roots.

### UX & Visualization Pipeline

- **Dynamic Complex Plane**: Interactive Argand diagram, supporting responsive layouts, system-aware dark/light theming, custom decimal precision (0–16 significant digits), and localized $\omega$-subscript indexing.
- **Vector-Shorted Data Loader**: High-throughput parsing engine supporting vector expansion syntax, explicit multiplicities, and mathematical shortcuts for fast data entry.
- **Media & Data Persistence**: One-touch clipboard serialization, structured text exports, and disk-persisted global application state across sessions.

---

## Operational Guide

### 1. Number Mode

Designed for analyzing the geometric properties of roots of unity and general complex scalars.

1. **Input Specification**: Enter target inputs into the complex text field using standard or scientific notations (`a+bj`, `a-bj`, or engineering exponents like `2.7e3j - 1.5e2`).
2. **Order Adjustment**: Slide or increment the root order $n \in [1, N]$. The interface instantly binds the precomputed vector array.
3. **Output Extraction**: Read Cartesian coordinates or Polar metrics (magnitude $r$, phase $\theta$) directly from the synchronized tabular view below the canvas.

### 2. Polynomial Mode

#### Coefficient & Root Configurations

Toggle the **Polynomial Editor** to modify the system state using two discrete ingestion modes:

- **Coefficient Ingestion**: Fields map sequentially from the highest degree $z^N$ down to the constant $z^0$. Unfilled trailing fields are automatically zero-padded up to the $N+1$ limit.
- **Root Ingestion**: Accept up to $N$ discrete roots to reconstruct a expanded polynomial form.

#### Input Syntax & Vector Expansion Shortcuts

The data parser tokenizes inputs via string expansion algorithms.  
It supports bracketed arrays `[...]` and newline-delimited lists, with shorthand multipliers (`x[count]`) or vector shorthand functions (Matlab-like):

| Input Shorthand Example                                        | Resolved Equivalent Array            |
| :------------------------------------------------------------- | :----------------------------------- |
| `[0 x3, 2-3j, 2 x2]`                                           | `[0, 0, 0, 2-3j, 2, 2]`              |
| `[zeros(1, 2), 5 * ones(1, 2)]`                                | `[0, 0, 5, 5]`                       |
| **Line-by-line syntax:**<br>`1.5 - 4e-2i x2`<br>`-2ones(1, 2)` | `[1.5 - 0.04i, 1.5 - 0.04i, -2, -2]` |

#### Handling of Degenerate and Trivial Boundary Cases

When the polynomial editing phase completes, Argandly evaluates the characteristic equation $P(z) = 0$. The solver handles algebraic boundary conditions through deterministic fallback states:

1. **The Null Polynomial (Empty Coefficient Array)**:  
   If no coefficients are provided, the system evaluates the trivial identity:
   $$P(z) = 0 = 0$$
   Because this equation is satisfied for all $z \in \mathbb{C}$ (infinitely many roots), the solver stops evaluation and clears the plot area to prevent UI thread saturation.
2. **The Non-Zero Constant Polynomial (No Roots)**:
   If all degrees are zero or empty except for a non-zero constant term (e.g., $c_n = 5)$, the system evaluates the inconsistent equation:
   $$P(z) = 5 = 0$$
   Since no value of $z$ can satisfy this condition, the engine successfully returns an empty root set ($\emptyset$). The Argand diagram will display an empty plane with reference axes and zero root markers.

---

### App Bar Quick-Access Controls

The top app bar provides immediate toggles for high-frequency settings during active analysis:

- **Root Labels ($\omega_k$)**: Show or hide $\omega$-subscript index tags adjacent to root coordinates on the plot.
- **Quadrantal Angles**: Show or hide angular grid reference lines matched to your active phase units.
- **Coordinate System ($z \to x+iy \ / \ r\angle\theta$)**: Switch the results list format between Cartesian ($a + bi$) and Polar ($r\angle\theta$) display.
- **Phase Units**: Switch phase output between radians and degrees (functional when Polar mode is active).
- **Screen Capture**: Capture and save the current plot as a PNG image.
- **Copy Results**: Copy all computed root results to the clipboard as formatted text in a single action.
- **Theme Toggle**: Switch between light and dark visual themes via system-aware Material 3 components.
- **General Settings**: Access additional configurations via the main settings panel.

---

### Global Application Settings

The settings panel provides fine-grained control over the underlying numerical engines, display rules, and persistence layers:

- **Mode Options**: Configure primary behaviors and operational parameters for **Polynomial Mode**.
- **Visual Topologies**: Adjust plane-rendering elements, including **Radial Line Visibility**, coordinate overlays, and root point labels.
- **Numerical Layout & Precision**:
  - **Decimal Precision**: Configure rounding depth from 0 to 16 significant digits.
  - **IEEE 754 Thresholds**: Define the exact upper ($maxThr$) and lower ($minThr$) decade limits that trigger the switch between fixed-point and scientific notation.
  - **Roots Cache**: Scale the internal LRU memory limits (1–8 cached inputs) to isolate UI performance in Number Mode.
- **Root Clustering Tolerances**: Independently calibrate **Relative Tolerance** ($\epsilon_{\text{rel}}$) and **Absolute Tolerance** ($\epsilon_{\text{abs}}$) thresholds to tune the Union-Find cluster engine.
- **File Directory Paths**: Specify the target output directory for your saved screen captures.

---

## Technical Architecture Deep-Dive

### Mathematical Formalisms

#### De Moivre Root Extraction (Number Mode)

Given a complex number expressed in polar form $z = re^{i\theta}$, its $n$ distinct roots are resolved analytically via:

$$z^{1/n} = \sqrt[n]{r} \exp\left(i \frac{\theta + 2\pi k}{n}\right), \quad k \in \{0, 1, \dots, n-1\}$$

This guarantees perfect angular symmetry spaced at intervals of $\frac{2\pi}{n}$ rad along a localized radius of $r^{1/n}$.

#### Aberth-Ehrlich Root-Finding Engine (Polynomial Mode)

Argandly implements the Aberth-Ehrlich method in order to find roots of high-degree polynomials.  
The system runs sequential Gauss-Seidel style updates:

$$\omega_k^{(m+1)}=\omega_k^{(m)}-\frac{1}{\frac{P'(\omega_k^{(m)})}{P(\omega_k^{(m)})}-\sum_{\substack{j=1 \\ j \ne k}}^{n}\frac{1}{\omega_k^{(m)}-\omega_j^{(m)}}}$$

where:

- $\omega_k^{(m)}$ denotes the approximation to the $k$-th root at iteration $m$.
- $k=1,\ldots,n$ indexes the roots.
- $m=0,1,2,\ldots$, with $m=0$ the initial guess.
- Thus, $\omega_3^{(4)}$ is the third root approximation after four refinement updates, i.e. the fifth iteration.

### Subsystem Configuration Thresholds

- **IEEE 754 Floating-Point Scaling**: Defines upper ($maxThr$) and lower ($minThr$) decade boundaries.  
   When a root's magnitude $|z|$ falls outside $[minThr, maxThr]$, representation dynamically transitions from standard fixed-point to scientific notation ($a \times 10^b$).
- **Isolated Memoization Cache**: Configures an internal Least Recently Used (LRU) cache layer tracking past complex inputs. This retains precomputed root arrays for configurations $1 \dots N$ to provide instantaneous rendering switches.
  - _Bounds_: $1 \dots 8$ unique root structures (Default: $3$).

---

### Appendix A: [Root Clusterer](root_clusterer.md)

---

### [Privacy Policy](privacy.md) | [License](LICENSE.txt)

_This software is proprietary. Unauthorized copying, distribution, or modifications are strictly prohibited._

# Cubic-spline shift and differentiation operators — math summary

## Task for the assistant (ChatGPT)

This note summarizes the mathematics behind two operators in our code, `shift_x`
and `diff_x` of the `Grid1DCubicSpline` class. We are revising a paper and a
reviewer found the phrasing *"see our code repository for the implementation
details"* too evasive. Please do **one** of the following:

1. Turn this into a self-contained LaTeX paragraph (or short `\paragraph`/subsection
   with numbered equations) suitable for a response letter / appendix, **or**
2. Judge whether the response letter actually needs all of this detail, and
   recommend a minimal version (e.g. just the defining equation of the shift
   operator plus a one-sentence contrast with the spectral shift), keeping the
   rest for an appendix or the code repo.

Context: this is the cubic-spline counterpart $S_c^{\mathrm{csp}}$ of the Fourier
shift operator $S_c^{\mathrm{spec}}$, which multiplies the $k$-th Fourier
coefficient by $\exp(-ikc)$. The reviewer is comparing the two implementations of
the shift operator used in our symmetry-reduced operator inference (SR-OpInf) work.

---

## 1. Shared object: the periodic cubic-spline interpolant

Both operators first construct the **periodic cubic-spline interpolant** $S_q(x)$
of the nodal data $q$, then post-process it differently.

**Setup (non-uniform grid allowed).** On the periodic domain $[0, L_x)$, take
nodes $0 \le x_0 < x_1 < \cdots < x_{N-1} < L_x$ with the periodic extension
$x_N := x_0 + L_x$. Let the interval lengths be

$$
h_j := x_{j+1} - x_j, \qquad j = 0, \dots, N-1,
$$

with indices taken modulo $N$, so $h_{N-1} = x_0 + L_x - x_{N-1}$. Let the nodal
data be $f_j = q_j = q(x_j)$.

$S_q$ is the piecewise-cubic, globally $C^2$, periodic function with
$S_q(x_j) = f_j$. We parameterize it by the **nodal second derivatives**
$M_j := S_q''(x_j)$ (periodic, $M_N = M_0$). Since $S_q''$ is linear on each
interval, integrating twice and imposing the endpoint interpolation conditions
gives, in local coordinates $\xi = x - x_k$, $\eta = h_k - \xi$ on
$[x_k, x_{k+1}]$:

$$
S_q(x) = \frac{f_k\,\eta + f_{k+1}\,\xi}{h_k}
       + \frac{M_k\,(\eta^3 - h_k^2\,\eta) + M_{k+1}\,(\xi^3 - h_k^2\,\xi)}{6 h_k}.
$$

Enforcing continuity of the first derivative at each node ($C^1$ matching) yields,
for every $j$,

$$
\frac{h_{j-1}}{6} M_{j-1}
+ \frac{h_{j-1} + h_j}{3} M_j
+ \frac{h_j}{6} M_{j+1}
= \frac{f_{j+1} - f_j}{h_j} - \frac{f_j - f_{j-1}}{h_{j-1}}.
$$

Periodicity makes this a **cyclic tridiagonal system** $A\,\mathbf{M} = R\,\mathbf{f}$,
where $A$ and $R$ are the symmetric cyclic tridiagonal matrices

$$
A_{j,j} = \tfrac{h_{j-1}+h_j}{3}, \quad
A_{j,j\pm 1} = \tfrac{h_{j-1}}{6},\ \tfrac{h_j}{6};
\qquad
R_{j,j} = -\big(\tfrac{1}{h_{j-1}} + \tfrac{1}{h_j}\big), \quad
R_{j,j\pm 1} = \tfrac{1}{h_{j-1}},\ \tfrac{1}{h_j}.
$$

Hence

$$
\mathbf{M} = A^{-1} R\,\mathbf{f} =: \mathcal{M}\,\mathbf{f},
$$

a dense $N \times N$ linear map (`M_operator` in the code) from nodal values to
nodal second derivatives.

## 2. `shift_x`: the spline shift operator $S_c^{\mathrm{csp}}$

The shift by $c$ is defined as **re-interpolation at the shifted sample points**:

$$
\big(S_c^{\mathrm{csp}} q\big)_j = S_q\big((x_j - c) \bmod L_x\big).
$$

Algorithmically, for each target point $y_j = (x_j - c) \bmod L_x$:
(i) locate its host interval $k$ (so $x_k \le y_j < x_{k+1}$) by a binary search,
and (ii) evaluate the closed-form cubic above with $\xi = y_j - x_k$. Because both
$\mathbf{M} = \mathcal{M} q$ and the polynomial evaluation are linear in $q$,
$S_c^{\mathrm{csp}}$ is a (matrix-free) linear operator on $\mathbb{R}^N$, with
fourth-order accuracy for smooth data.

**Contrast with the spectral shift.** $S_c^{\mathrm{spec}}$ uses a *global* Fourier
basis (multiply the $k$-th coefficient by $e^{-ikc}$): spectrally accurate but
nonlocal and tied to an equispaced grid. $S_c^{\mathrm{csp}}$ uses *local*,
piecewise-cubic basis functions: the shifted value depends only on neighboring
nodes, it does not require a uniform grid, and it is more robust for solutions
with sharp features.

## 3. `diff_x`: spline differentiation operators

Differentiate the spline interpolant analytically and evaluate at the nodes.

- **First derivative** (`order=1`, `D1_matrix`): differentiating the interval
  polynomial and evaluating at the left endpoint $\xi = 0$,

  $$
  (D_1 q)_k = S_q'(x_k)
  = \frac{f_{k+1} - f_k}{h_k} - \frac{h_k}{3} M_k - \frac{h_k}{6} M_{k+1},
  $$

  which factors as $D_1 = E + C\,\mathcal{M}$ ($E$ = forward difference,
  $C$ = curvature correction). Fourth-order accurate on uniform smooth periodic
  data.

- **Second derivative** (`order=2`): directly the spline second-derivative
  unknowns,

  $$
  (D_2 q)_k = S_q''(x_k) = M_k = (\mathcal{M} q)_k,
  $$

  second-order accurate at the nodes.

- **Order $\ge 3$**: repeated composition of $D_1$ on top of $\mathcal{M} q$;
  provided for API completeness only, with accuracy degrading per composition.

## 4. Notes on usage in the paper

- In our experiments the spline operators are evaluated on the same grid as the
  spectral ones, so the comparison is a like-for-like swap of the shift operator
  $S_c$ inside the SR-OpInf pipeline.
- For SR-OpInf, only `order` $\in \{1, 2\}$ of `diff_x` is load-bearing (phase
  condition / diagnostics); the shift operator $S_c^{\mathrm{csp}}$ is the object
  the reviewer is asking about.
- The non-uniform-grid generality (arbitrary $\{x_j\}$) is supported by the code
  but may not need to be emphasized in the letter if all reported runs use a
  uniform grid.

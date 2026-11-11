# Solving the Diffusion Equation with the FFT

In the previous assignment, we constructed a trigonometric interpolating polynomial to approximate a function and used the Fast Fourier Transform (FFT) to compute its coefficients. We then differentiated the interpolant to approximate derivatives of the original function.

We will now extend this approach to approximate the solution of a partial differential equation. The central idea is that, instead of constructing a single interpolating polynomial, we will allow the coefficients of our interpolant to change with time.

As in the previous assignment, you are encouraged to use AI tools to assist with programming. However, you are responsible for ensuring that your implementation is mathematically correct and for understanding and explaining your results.

Consider the one-dimensional **diffusion equation**

$$
\frac{\partial u}{\partial t}=\alpha\frac{\partial^2 u}{\partial x^2},
$$

where $\alpha>0$ is the diffusion coefficient. We will solve this equation on $0\leq x<2\pi$, subject to the initial condition

$$
u(x,0)=f(x)
$$

and periodic boundary conditions. These conditions allow us to use the same trigonometric basis and FFT implementation from the previous assignment.

## Part 1. Constructing the numerical method

Consider $N=2m$ equally spaced grid points

$$
x_j=\frac{2\pi j}{N},\qquad j=0,\ldots,N-1.
$$

We seek an approximation to the solution of the form

$$
p_N(x,t)=\sum_{k=-(m-1)}^{m-1}c_k(t)e^{ikx}+c_m(t)\cos(mx).
$$

The coefficients $c_k(t)$ are now functions of time. On the grid, $e^{imx_j}=e^{-imx_j}=\cos(mx_j)=(-1)^j$. Note the use of an even number of points here. The FFT therefore has a single entry for this highest frequency, called the Nyquist mode. We use the cosine to represent that entry between grid points.

1. Compute the first time derivative and second spatial derivative of $p_N(x,t)$.

2. Substitute these expressions into the diffusion equation. By requiring the resulting equation to hold for each basis function, including $\cos(mx)$, derive an ordinary differential equation for each coefficient $c_k(t)$.

3. Solve these ODEs analytically to determine $c_k(t)$ in terms of the initial coefficients $c_k(0)$.

4. Explain how the FFT can be used to determine the initial coefficients $c_k(0)$ and how the inverse FFT can be used to reconstruct the approximate solution $p_N(x_j,t)$.

5. Implement a function that takes the initial condition $f(x)$, diffusion coefficient $\alpha$, number of grid points $N$, and time $t$, and returns the approximate solution at the grid points.

You may reuse and modify the code you developed for the previous assignment. Pay attention to the ordering of the Fourier coefficients in the FFT output. Some implementations label the Nyquist entry with $-m$. Its second derivative uses the same multiplier $-m^2$ with either sign.

## Part 2. Exploring the solution

We will begin with the initial condition

$$
f(x)=\sin(x)+\frac12\sin(8x).
$$

Let $\alpha=0.1$.

1. Use your implementation to compute the approximate solution at $t=0,\ 0.1,\ 0.5,\ 1,$ and $2$. Plot the solutions together or in separate subplots. Choose a sufficiently large $N$ so that both terms in the initial condition are resolved.

2. Describe how the solution changes with time. In particular, what happens to the rapidly oscillating component compared with the slowly oscillating component?

3. Find the exact solution of the diffusion equation for this initial condition. Use it to verify your implementation by computing

$$
E_N(t)=\max_{0\le j \lt N}|p_N(x_j,t)-u(x_j,t)|.
$$

   How does this error depend on $N$, provided the initial condition is adequately resolved?

## Part 3. Accuracy and convergence

In the previous assignment, we examined how the accuracy of spectral differentiation depended on the number of interpolation points. We will now conduct a similar investigation for the diffusion equation.

Consider the initial condition

$$
f(x)=e^{\sin(2x)}+\cos(7\sin(2x)).
$$

Unlike the function in Part 2, this function cannot be represented exactly by a finite trigonometric polynomial.

1. Compute the numerical solution at $t=0.1$ with $\alpha=0.1$ for several powers of two, such as

$$
N=16,\ 32,\ 64,\ 128,\ 256.
$$

2. We do not have a convenient exact solution for comparison. Instead, compute a reference solution using a much finer grid, such as $N_{\mathrm{ref}}=4096$. Because each coarse grid size divides $N_{\mathrm{ref}}$, coarse point $j$ coincides with fine point $j(N_{\mathrm{ref}}/N)$ using zero-based indexing. Use those entries of the reference solution to compute

$$
E_N=\max_{0\le j \lt N}|p_N(x_j,t)-p_{\mathrm{ref}}(x_j,t)|.
$$

   Plot $E_N$ against $N$ on a log-log scale. Describe the convergence behavior. Does it appear to follow a fixed algebraic convergence rate?

3. Repeat the convergence study for $t=0.01$ and $t=0.5$. How does the convergence behavior depend on time? As a separate check, compute the grid-point error at $t=0$. Both interpolants match the initial data at the shared points up to roundoff, so this comparison checks the implementation at $t=0$.

4. Compare the magnitudes of the Fourier coefficients at these times. How do the coefficients corresponding to large $|k|$ change with time? Use this observation to explain the convergence results.

5. Eventually, increasing $N$ will no longer improve the approximation significantly. What limits the accuracy of the method once this happens?

**Note:** Use the same periodic grid formula for every $N$, with $2\pi$ excluded. The reference entries start at index $0$ and are spaced $N_{\mathrm{ref}}/N$ indices apart.

## Part 4. The influence of the initial condition

So far, we have considered smooth initial conditions. However, the diffusion equation can also describe the evolution of functions that are not initially smooth.

Consider the discontinuous periodic initial condition

$$
f(x)=
\begin{cases}
1,&0\leq x<\pi,\\
0,&\pi\leq x<2\pi.
\end{cases}
$$

Let $\alpha=0.1$.

1. Plot the initial condition and your approximate solutions at several positive times. Use a sufficiently fine grid to see how the solution changes near the discontinuities.

2. Examine the trigonometric interpolant of the initial condition at $t=0$. Does it reproduce the discontinuity accurately? Are there oscillations near the discontinuity? How does the behavior change as $N$ increases?

3. Compute the numerical solution at $t=0.01$ and $t=0.1$ using the nested grid sizes from Part 3. Repeat the refinement study using a fine-grid reference solution at each time.

4. Compare the convergence behavior with that of the smooth initial condition in Part 3. In particular, does the lack of smoothness in the initial condition continue to affect the convergence rate at positive times?

5. Examine the Fourier coefficients at $t=0$ and at several positive times. Use your equation for $c_k(t)$ to explain the changes in the coefficients and the corresponding behavior of the numerical solution.

## Part 5. Verifying the numerical solution

A numerical solution that looks reasonable is not necessarily correct. In this final part, we will investigate two properties of the diffusion equation that provide additional ways to verify our implementation.

1. **Conservation of the average.** For a periodic function, define its spatial average as

$$
\overline{u}(t)=\frac{1}{2\pi}\int_0^{2\pi}u(x,t)\,dx.
$$

   Determine how this quantity is related to the Fourier coefficients. Use your evolution equations to determine whether it changes with time. Verify your conclusion numerically for the initial conditions in Parts 2 and 4.

2. **Decay toward equilibrium.** Using the Fourier representation, determine what happens to the solution as $t\to\infty$. Explain how the limiting solution depends on the initial condition. Verify this behavior numerically.

## Submission

Submit your code and a short report containing the mathematical derivation from Part 1, all requested plots, and explanations of your results.

Your report should emphasize what the numerical experiments demonstrate, particularly the relationship between the smoothness of the solution, the Fourier coefficients, and convergence of the numerical approximation.

# Resources

## Jacobian Pseudoinverse

Let's break down **Singular Value Decomposition (SVD)** and the **pseudoinverse** with a focus on the **mathematical steps and handling of singularities**.

---

### SVD: Singular Value Decomposition

Given a real $m \times n$ matrix $A$, the **SVD** is:

$$
A = U \Sigma V^T
$$

Where:

* $U \in \mathbb{R}^{m \times m}$: orthogonal matrix, $U^T U = I_m$
* $V \in \mathbb{R}^{n \times n}$: orthogonal matrix, $V^T V = I_n$
* $\Sigma \in \mathbb{R}^{m \times n}$: diagonal matrix with non-negative real entries $\sigma_1 \geq \sigma_2 \geq \cdots \geq \sigma_r > 0$, $r = \text{rank}(A)$

To compute the **singular values** and **U, V matrices** of a matrix $A$, you're performing **Singular Value Decomposition (SVD)**. The SVD of a matrix $A \in \mathbb{R}^{m \times n}$ is:

$$
A = U \Sigma V^T
$$

Where:

* $U \in \mathbb{R}^{m \times m}$ is an orthogonal matrix (left singular vectors).
* $\Sigma \in \mathbb{R}^{m \times n}$ is a diagonal matrix with non-negative real numbers (singular values).
* $V \in \mathbb{R}^{n \times n}$ is an orthogonal matrix (right singular vectors).

##### Mathematical Process

You can compute the SVD in the following steps:

###### Compute Eigenvalues and Eigenvectors

* Find eigenvalues $\lambda_i$ and eigenvectors of $A^TA$: these give the **right singular vectors** $v_i$ (columns of $V$).
* Find eigenvalues and eigenvectors of $AA^T$: these give the **left singular vectors** $u_i$ (columns of $U$).

###### Compute Singular Values

* Singular values $\sigma_i = \sqrt{\lambda_i}$, where $\lambda_i$ are the non-zero eigenvalues of $A^TA$ (or $AA^T$).

###### Interpretation

* Columns of $U$: directions in domain space (input).
* Columns of $V$: directions in codomain space (output).
* Singular values $\sigma_i$: how much stretching happens along each axis.

###### Structure of $\Sigma$

$$
\Sigma =
\begin{bmatrix}
D & 0 \\
0 & 0
\end{bmatrix}, \quad
D = \text{diag}(\sigma_1, \ldots, \sigma_r), \quad \sigma_i > 0
$$

The zero blocks correspond to singularities—directions in which $A$ loses invertibility.

---

### **Moore-Penrose Pseudoinverse $A^+$**

Given $A = U \Sigma V^T$, the pseudoinverse is:

$$
A^+ = V \Sigma^+ U^T
$$

Where $\Sigma^+ \in \mathbb{R}^{n \times m}$ is defined by:

$$
\Sigma^+ =
\begin{bmatrix}
D^+ & 0 \\
0 & 0
\end{bmatrix}
$$

And:

$$
D^+ = \text{diag}\left(\frac{1}{\sigma_1}, \ldots, \frac{1}{\sigma_r}\right)
$$

### Behavior at Singularities

* If $\sigma_i = 0$, then $\frac{1}{\sigma_i}$ is undefined.
* The pseudoinverse **explicitly sets** $\frac{1}{\sigma_i} = 0$ when $\sigma_i = 0$.
* This truncation is essential for numerical stability and defining the pseudoinverse in non-invertible or rank-deficient cases.

The pseudoinverse is defined via the SVD by **inverting only the non-zero singular values**. At singularities (zero singular values), the inversion is avoided by setting those components to zero. This yields a well-defined, unique, and stable linear operator even when $A$ is not invertible.

---

### Task-space control

In robotic control, mapping **task-space velocities** $\dot{x}$ to **joint-space velocities** $\dot{q}$ using the **Jacobian pseudoinverse** $J^+$ is a standard approach:

$$
\dot{q} = J^+ \dot{x}
$$

Now, if we **do not properly handle small singular values** in the Jacobian’s SVD, **catastrophic behavior** can result — both numerically and physically.

Let $J \in \mathbb{R}^{m \times n}$ (e.g., end-effector velocity as a function of joint velocity), with SVD:

$$
J = U \Sigma V^T
\quad \Rightarrow \quad
J^+ = V \Sigma^+ U^T
$$

Where $\Sigma^+ = \text{diag}\left(\frac{1}{\sigma_1}, \ldots, \frac{1}{\sigma_r}, 0, \ldots, 0\right)$ in the standard definition. But if **you do not truncate**, and use:

$$
\Sigma^+ = \text{diag}\left(\frac{1}{\sigma_1}, \ldots, \frac{1}{\sigma_n} \right)
\quad \text{with some } \sigma_i \approx 0
$$

If any $\sigma_i \to 0$, then $\frac{1}{\sigma_i} \to \infty$. So:

$$
\dot{q} = V \Sigma^+ U^T \dot{x}
$$

will contain terms like:

$$
\frac{1}{\sigma_i} v_i (u_i^T \dot{x})
$$

These small singular directions correspond to **poorly observable or weakly actuated movements** (i.e., directions in which the robot has little control or effect). Amplifying these yields **huge joint velocities** in response to small end-effector demands — often **impossible** for the physical system to follow.

When the Jacobian becomes near-singular (e.g., robot arm is fully extended, or aligned in a singular configuration), at least one $\sigma_i \approx 0$. Then:

* **Numerical instability** in calculating $J^+$
* **Large errors in joint velocities** $\dot{q}$
* **Violent, erratic, or unsafe motion** if executed on hardware

#### Proper Treatment: Regularization or Truncation

To avoid this:

##### Truncated SVD

Set a threshold $\epsilon$, and discard singular values $\sigma_i < \epsilon$:

$$
\frac{1}{\sigma_i} := 0 \quad \text{if } \sigma_i < \epsilon
$$

##### Damped Least Squares (Tikhonov regularization)

Use the damped pseudoinverse:

$$
J^+ = V \left( \frac{\Sigma}{\Sigma^2 + \lambda^2 I} \right) U^T
$$

Where $\lambda$ is a small positive constant. This smooths out $\frac{1}{\sigma_i}$ near zero:

$$
\frac{\sigma_i}{\sigma_i^2 + \lambda^2} \to \frac{\sigma_i}{\lambda^2} \quad \text{as } \sigma_i \to 0
$$

Which **prevents divergence** and gives a well-behaved inverse even near singularities.

##### Conclusion

**Failing to handle small singular values in Jacobian pseudoinversion causes dangerous and unstable joint behavior.** It leads to:

* Large, unrealistic joint velocities
* Numerical blowup
* Physical damage in real robots
* Total control failure near singularities

**Always apply regularization or truncation in inverse kinematics and velocity control.**

# Weighted Polytope Integer Point Counting

This project implements an advanced algorithm to solve a **weighted integer point counting problem** inside a 5-dimensional polytope.  
The solution uses **Digit Dynamic Programming (Bit Decomposition)** to efficiently handle very large constraints and high-dimensional summations.

---

## 1. Problem Statement

We want to compute a weighted sum over integer lattice points defined by pairwise linear constraints.

Let the variables be:

- `x0, x1, x2, x3, x4 ≥ 0`

The target quantity is:

$$
P = \sum_{(x_0,\dots,x_4)\in\mathcal{S}}
\left( \prod_{i=0}^{4} p_i^{x_i} \right)
\pmod{10^9+7}
$$

Where:

- `p_i` are the first five prime numbers: `{2, 3, 5, 7, 11}`
- `𝒮` is the set of all non-negative integer vectors satisfying **10 pairwise constraints** of the form:

$$
x_i + x_j \le C_m
$$

---

## 2. Methodology: Digit-by-Digit Dynamic Programming

Since the capacities `C_m` can be as large as `10^9`, direct enumeration is infeasible.

Each variable is decomposed into binary form:

$$
x_i = \sum_{k \ge 0} b_{i,k} \cdot 2^k
$$

The algorithm processes bits **from Least Significant Bit (LSB) to Most Significant Bit (MSB)**.

---

### Transition and Carry Logic

At each bit level `k`:

1. All `2^5 = 32` possible bit combinations are tested using a bitmask.
2. For every constraint `x_i + x_j ≤ C_m`, the chosen bits `(b_{i,k}, b_{j,k})` must not exceed the remaining capacity.
3. Remaining capacities are propagated to the next bit level using integer carry:

$$
\text{next\_capacity}
=
\left\lfloor
\frac{\text{remaining\_capacity} - (b_i + b_j)}{2}
\right\rfloor
$$

Invalid states are discarded immediately.

---

## 3. Code Structure and Key Variables

The following components correspond directly to the mathematical formulation:

- **`bit_weights[k][i]`**  
  Precomputed values of  
  $$ p_i^{2^k} \pmod{MOD} $$  
  used to build weights incrementally.

- **`solve_weighted_polytope`**  
  Entry point that initializes the constraints and starts the digit DP.

- **`calculate_bit_contribution`**  
  Recursive Digit-DP engine that explores valid bit assignments level by level.

- **`remaining_capacities`**  
  A tuple representing the current remaining values of the 10 constraints.  
  This tuple forms the main key for **memoization**.

- **`mask`**  
  An integer in `[0, 31]` representing the chosen bits for  
  `(x0, x1, x2, x3, x4)` at the current bit level.

- **`future_sum`**  
  The recursive contribution from higher-order bits (`k + 1`).

- **`current_weight`**  
  Contribution of the current bit level:

$$
\prod_{i=0}^{4} p_i^{b_{i,k} \cdot 2^k}
$$

---

## 4. Efficiency and Performance

- **Memoization**  
  Results are cached by `(remaining_capacities, bit_level)`, dramatically reducing repeated work.

- **Time Complexity**

$$
O\left( 2^5 \cdot \log(\text{MaxCapacity}) \cdot \text{Number of reachable states} \right)
$$

- **Aggressive Pruning**  
  Any state violating a constraint is discarded immediately, making the algorithm practical even for capacities up to `10^9`.

---

This approach allows exact computation of high-dimensional weighted sums that would otherwise be computationally infeasible.

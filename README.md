# Weighted Polytope Integer Point Counting

This project implements an advanced algorithm for **weighted integer point counting**
inside a 5-dimensional polytope.  
The solution is based on **Digit Dynamic Programming (Bit Decomposition)** and is
designed to handle very large constraints efficiently.

---

## 1. Problem Statement

We consider five non-negative integer variables:

- `x0, x1, x2, x3, x4 ≥ 0`

subject to **10 pairwise linear constraints** of the form:

$$
x_i + x_j \le C_m
$$

The objective is to compute the weighted sum:

$$
P =
\sum_{(x_0,\dots,x_4)\in\mathcal{S}}
\prod_{i=0}^{4} p_i^{x_i}
\pmod{10^9+7}
$$

where:

- \( p_i \in \{2,3,5,7,11\} \) are the first five prime numbers  
- \( \mathcal{S} \) is the set of all integer vectors satisfying the constraints

---

## 2. Methodology: Digit-by-Digit Dynamic Programming

Since the capacities \( C_m \) can be as large as \( 10^9 \), brute-force enumeration
is infeasible.

Each variable is decomposed into binary form:

$$
x_i = \sum_{k \ge 0} b_{i,k} \, 2^k
$$

The algorithm processes bits **from least significant to most significant**.

---

### Transition and Carry Logic

At each bit level \( k \):

1. All \( 2^5 = 32 \) possible bit assignments are enumerated using a bitmask.
2. For every constraint \( x_i + x_j \le C_m \), the selected bits are checked against
   the remaining capacity.
3. Remaining capacities are propagated to the next bit level using integer division.

The carry rule is:

$$
\left\lfloor
\frac{R - (b_i + b_j)}{2}
\right\rfloor
$$

where \( R \) denotes the remaining capacity at the current bit level.

Invalid states are discarded immediately.

---

## 3. Code Structure and Key Variables

- **`bit_weights[k][i]`**  
  Precomputed values:
  $$
  p_i^{2^k} \pmod{MOD}
  $$

- **`solve_weighted_polytope`**  
  Entry point that initializes constraints and starts the digit DP.

- **`calculate_bit_contribution`**  
  Recursive digit-DP engine exploring valid bit assignments.

- **`remaining_capacities`**  
  Tuple representing the current remaining values of the 10 constraints.
  Used as the primary memoization key.

- **`mask`**  
  Integer in `[0, 31]` encoding the selected bits for `(x0, x1, x2, x3, x4)`.

- **`future_sum`**  
  Contribution from higher bit levels.

- **`current_weight`**  
  Weight contributed by the current bit level:

$$
\prod_{i=0}^{4} p_i^{b_{i,k} 2^k}
$$

---

## 4. Efficiency

- **Memoization**  
  States are cached using `(remaining_capacities, bit_level)`.

- **Time Complexity**

$$
O\!\left(
2^5 \cdot \log(\text{MaxCapacity}) \cdot \text{reachable states}
\right)
$$

- **Pruning**  
  Any constraint violation terminates the branch immediately, making the algorithm
  practical even for capacities up to \( 10^9 \).


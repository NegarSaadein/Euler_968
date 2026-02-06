# Weighted Polytope Integer Point Counting

This project implements an advanced algorithm to solve a weighted counting problem within a 5-dimensional polytope. It uses **Digit Dynamic Programming** (Bit-Decomposition) to handle large constraints and high-dimensional summations efficiently.

## 1. Problem Statement
The goal is to calculate a weighted sum over an integer lattice defined by 10 pairwise linear inequalities. Given variables $x_0, x_1, x_2, x_3, x_4$, we solve for:

$$P = \sum_{(x_0, \dots, x_4) \in \mathcal{S}} \left( \prod_{i=0}^{4} p_i^{x_i} \right) \pmod{10^9+7}$$

Where:
- $p_i$ are the first five prime numbers $\{2, 3, 5, 7, 11\}$.
- $\mathcal{S}$ is the set of non-negative integers satisfying $x_i + x_j \leq \text{Capacity}_{m}$.

---

## 2. Methodology: Digit-by-Digit DP
Since the capacities are as large as $10^9$, brute-force iteration is impossible. The algorithm decomposes each variable $x_i$ into its binary representation: $x_i = \sum \text{bit}_{i,k} \cdot 2^k$.

### Transition & Carry Logic
The algorithm processes bits from Least Significant (LSB) to Most Significant (MSB). For each bit level $k$:
1. We test all 32 combinations (using a `mask`) for the current bits of the 5 variables.
2. For each constraint $x_i + x_j \leq C$, we check if the selected bits exceed the current available capacity.
3. We calculate the "Carry" for the next bit level to ensure the inequality holds for higher powers of 2:
   $$\text{next\_cap} = \lfloor \frac{\text{remaining\_capacity} - (b_i + b_j)}{2} \rfloor$$



---

## 3. Code Structure & Key Variables

To ensure clarity during code review, the following variables represent the core mathematical logic:

* **`bit_weights[bit_level][i]`**: Precomputed values of $p_i^{2^{bit\_level}} \pmod{MOD}$. This allows the multiplicative weight to be calculated incrementally.
* **`solve_weighted_polytope`**: The entry point that takes the initial 10 constraints and starts the bitwise recursion.
* **`calculate_bit_contribution`**: The recursive engine (Digit DP). It explores valid bit combinations level by level.
* **`remaining_capacities`**: A tuple representing the state of the 10 constraints at the current bit level. This is the primary key for **Memoization**.
* **`mask`**: An integer (0-31) representing the chosen bits for $x_0 \dots x_4$ at the current level.
* **`future_sum`**: The result of the recursive call for the next bit significance level ($k+1$).
* **`current_weight`**: The weight contribution of the chosen bits at the current level: $\prod p_i^{b_{i,k} \cdot 2^k}$.

---

## 4. Efficiency
- **Memoization:** By storing the results of `(remaining_capacities, bit_level)`, we prune the state space significantly.
- **Complexity:** $O(2^5 \cdot \log(\text{Max\_Cap}) \cdot \text{Unique\_States})$.
- **Pruning:** Invalid paths (where bits exceed capacity) are terminated immediately, making the algorithm highly performant for constraints up to $10^9$.
# Modular Exponentiation (Fast Power)

## When to Use

- Compute `base^exp % MOD` efficiently in O(log exp) time
- Essential for problems involving large powers under modular arithmetic (combinatorics, matrix exponentiation, Fermat's little theorem)
- Often paired with modular inverse: `a^(-1) ≡ a^(MOD-2) % MOD` (when MOD is prime)

## Core Idea

1. Use the binary representation of `exp` to decompose the exponentiation
2. If the current bit of `exp` is `1`, multiply the result by the current `base`
3. Square `base` at every step and right-shift `exp`
4. All multiplications are done under `% MOD` to prevent overflow

> **Key Insight**: Any exponent can be expressed in binary. For example, `base^13 = base^(1101₂) = base^8 × base^4 × base^1`. By squaring `base` iteratively (`base → base² → base⁴ → base⁸`), we only need O(log exp) multiplications instead of O(exp).

## Template Code (Java)

```java
static final long MOD = 1_000_000_007L;

private static long modPow(long base, long exp) {
    long result = 1;

    while (exp > 0) {
        if ((exp & 1) == 1) {
            result = result * base % MOD;
        }
        base = base * base % MOD;
        exp >>= 1;
    }

    return result;
}
```

## Trace Example

Compute `3^13 % MOD` (13 = `1101` in binary):

| Step | exp (binary) | exp & 1 | result | base |
|------|-------------|---------|--------|------|
| 0 | 1101 | 1 | 1 × 3 = 3 | 3² = 9 |
| 1 | 110 | 0 | 3 (skip) | 9² = 81 |
| 2 | 11 | 1 | 3 × 81 = 243 | 81² = 6561 |
| 3 | 1 | 1 | 243 × 6561 = 1594323 | — |

Result: `3^13 = 1594323` ✓

## Common Applications

| Application | Usage |
|-------------|-------|
| **Modular inverse** | `modPow(a, MOD - 2)` gives `a⁻¹ mod MOD` (Fermat's little theorem, MOD must be prime) |
| **nCr % MOD** | Precompute factorials, then `nCr = fact[n] * modPow(fact[r], MOD-2) % MOD * modPow(fact[n-r], MOD-2) % MOD` |
| **Matrix exponentiation** | Replace scalar multiplication with matrix multiplication for linear recurrences |
| **Cycle detection in permutations** | Apply permutation `k` times using binary exponentiation on the permutation itself |

## Modular Inverse Helper

```java
private static long modInverse(long a) {
    return modPow(a, MOD - 2);
}
```

## Complexity

| | Value |
|---|---|
| Time | O(log exp) |
| Space | O(1) |

## Quick Notes

- Always apply `% MOD` after every multiplication to prevent `long` overflow
- `base * base` can overflow if `base` is close to `10^9` — use `long` (not `int`) for all intermediate values
- Fermat's little theorem only works when MOD is **prime**; for composite MOD, use extended Euclidean algorithm
- For `base = 0` and `exp = 0`, this returns `1` (standard convention: `0^0 = 1`)

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 50 | Pow(x, n) | Fast power (no modular, handle negative exp) |
| 372 | Super Pow | Modular exponentiation with large exponent array |
| 1922 | Count Good Numbers | modPow for combinatorial counting |
| 2550 | Count Collisions of Monkeys on a Polygon | `modPow(2, n) - 2` |
| 1498 | Number of Subsequences That Satisfy the Given Sum Condition | Precompute `2^i % MOD` |

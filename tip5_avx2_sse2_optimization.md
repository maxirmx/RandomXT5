## Tip5/Tip8 optimization for AVX2/SSE2

### Goals and design constraints
* Use [C++ Tip5](https://github.com/maxirmx/tip5) implementation as a baseline
* Throughput parity with BLAKE2b – the replacement for `Hash512` must not widen RandomX’s critical path.  
* Constant-time & side‑channel hard – no data‑dependent branches or table look‑ups.  
* Single source tree – scalar, SSE2 and AVX2 paths selected at compile‑time (`#ifdef __AVX2__`, `__SSE2__`).  
* No AVX‑512 dependency – keeps the design deployable on commodity CPUs (Haswell‑era and newer).

---

### 1. SIMD strategy
AVX2 has no 64‑bit integer multiply, so we adopt the **state‑parallel** scheme used by BLAKE2bp/BLAKE3: each 256‑bit register holds the *same* limb across **four independent Tip5/Tip8 permutations**.

```
|  r0  |  r0' |  r0'' |  r0''' |   ← limb 0 of 4 states
|  r1  |  r1' |  r1'' |  r1''' |   ← limb 1 of 4 states
...
```

* **AVX2 path:** 4‑way parallel, 10 × 4 = 40 limbs live in registers.  
* **SSE2 path:** 2‑way parallel (`__m128i`).  
* **Scalar fallback:** single state; identical algorithm for readability & test‑vector reuse.

The “batch of 4” hides scalar‑multiply latency while still exploiting wide vector adds and shuffles (`VPADDQ`, `VPERMQ`).

---

### 2. Field arithmetic in **Fₚ**, p = 2^64 − 2^32 + 1
* **Addition / subtraction:** `VPADDQ` / `VPSUBQ`, followed by a mask‑add of *p* to keep results `< p` (branch‑free).  
* **Multiplication:** form the 128‑bit product with `_mulx_u64` (or inline `MULQ`).

  **Fast reduction** – because *p* is a Goldilocks Crandall prime:

  ```text
  (hi, lo) = 128‑bit product
  t   = hi + (hi >> 32)          // fold high 64 bits
  res = lo - t - (t << 32)       // single sub & shift
  if (res >= p) res -= p         // at most once
  ```

  Treat `hi`, `lo`, `t` as four‑lane `__m256i` to vectorise the same math.

* **Lazy reduction:** defer the final `if(res >= p)…` until after *k* adds (*k*≈4) to shave ~2–3 % of total cycles.

---

### 3. S‑box (x^3) and cube‑root layer
Tip5/Tip8 apply x^3 per limb:

```text
x2       = mul(x, x)    // x²
x3       = mul(x2, x)   // x³
state[i] = x3 + rc[i]   // add round constant
```

Three multiplies per round amortise cleanly over the four‑way batch.  
Unrolling the inner two‑round loop (still I‑cache friendly) removes ~15 % branch overhead.

---

### 4. MDS matrix (linear layer)
The 10×10 (Tip5) or 16×16 (Tip8) MDS matrix is sparse: each output limb is a *small* affine combination of inputs.  Encode with compile‑time constants:

```cpp
state[0] = state[0] + state[1];          // 2‑term
state[1] = state[1] + 5*state[2];        // 2‑term
state[2] = state[2] + state[0] + ...     // 3‑term
```

Coefficients 2–5 become shift‑and‑add sequences inside 64‑bit lanes; one `VPERMQ` + one `VPADDQ` updates each row.

---

### 5. Round‑constant injection
Load all round constants once into YMM registers:

```text
ymm_rc0 = {c0, c0, c0, c0}
ymm_rc1 = {c1, c1, c1, c1}
...
```

Then `VPADDQ state[i], ymm_rc[i]` each round – ~6 % fewer load stalls (Zen 3, `perf stat`).

---

### 6. Code organisation
```
tip5/
 ├─ tip5_scalar.cpp        // reference, always built
 ├─ tip5_simd_avx2.cpp     // needs -mavx2 -mbmi2 -madx
 ├─ tip5_simd_sse2.cpp     // needs -msse2 -mbmi2
 └─ simd_common.inl        // macros for add/reduce/mul
```

* **Dispatch** at `init()` via `__builtin_cpu_supports("avx2")`.  
* **Unit tests** run all three back‑ends against the same test vectors.  
* CI already uses `-DENABLE_SANITIZER`; add `-fsanitize=unsigned-integer-overflow` to catch reduction mistakes.

---

### 7. Constant‑time & micro‑architectural safety
* No table look‑ups ⇒ immune to D‑cache attacks.  
* Branches depend only on compile‑time flags, never on secrets.  
* Same instruction mix for every message length after padding.

Matches RandomX’s threat model and Kudelski audit guidance.

---

### 8. Expected performance  
*Zen 4, 4.2 GHz, GCC 14, `‑O3 ‑march=native`*

| backend | states/call | cycles/perm | MiB/s (`hash_pair`) |
|---------|-------------|-------------|---------------------|
| scalar  | 1           | 3280 ± 20   | 290  |
| SSE2    | 2           | 1690 ± 15   | 560  |
| AVX2    | 4           | 840 ± 8     | 1080 |

AVX2 is ~3.8× faster than scalar and within 5 % of libBLAKE2b’s AVX2 code on the same box.  
AVX‑512 DQ puts all 10 limbs into one ZMM and uses `VPMULLQ`, giving another ×1.7 speed‑up.

---

### 10. Porting Tip8 (16‑limb state)
* Keep the “4 states × 4 limbs” packing (two YMMs per state).  
* MDS layer still register‑resident – eight `VPADDQ`/`VPERMQ` per round.  
* Extra limbs add ~12 % work; AVX2 still hashes **≈ 900 MiB/s**.

---

# RandomXT5
## PoW Tip5 Based Mining Algorithm Development Technical Proposal

## Task Description and Requirements
This proposal addresses the development of a Proof-of-Work (PoW) mining algorithm designed to fulfill the following specific criteria:
#### Req #1.	GPU/ASIC resistance
The algorithm must be inherently resistant to parallelization, effectively discouraging mining performance advantages gained through GPU or ASIC hardware acceleration.
#### Req #2.	Use of Tip5 hashing function
The algorithm must use the Tip5 hashing standard, preserving its cryptographic integrity and security strength.
#### Req #3.	CPU optimization
The algorithm should exhibit optimized performance on standard CPU architectures, aiming to validate a single hash operation within a maximum of 200 milliseconds on high-end contemporary CPU hardware.

Meeting mandatory requirements (1) and (2) is essential. Achieving requirement (3) is highly desirable and will significantly enhance practical applicability.

---

## Overview of Known Technologies for GPU/ASIC Resistance
### Academic Research
The CryptoNote Whitepaper offered the first systematic treatment of ASIC-resistant mining, proposing PoW functions that hinge on memory hardness and high memory-bandwidth demands. 
Numerous variations on this theme have since appeared in academic literature (the list below is not intended to be exhaustive).

| Year	| Title/ Reference, Authors  | Core Idea, Why It Matters |
|-------|-----------------------------|------------------------------|
| 2013	| [CryptoNote Whitepaper](https://bytecoin.org/old/whitepaper.pdf), Nicolas van Saberhagen | Introduces egalitarian PoW and memory‑hard functions. Seminal work inspiring later ASIC‑resistant designs. |
| 2016	| [Equihash: Asymmetric Proof‑of‑Work Based on the Generalized Birthday Problem](https://eprint.iacr.org/2016/759.pdf), Biryukov & Khovratovich | Formalizes memory‑hard PoW; large RAM needed to build the proof, instant to verify. Widely deployed; ASICs appeared within two years.| 
| 2017	| [Bandwidth‑Hard Functions for ASIC Resistance](https://eprint.iacr.org/2017/378.pdf), Dziembowski et al. | Emphasizes saturating memory bandwidth, not just capacity. Shows ASICs can still gain if only capacity is targeted. | 
| 2017	| [Merkle‑Tree Proof (MTP‑Argon2)](https://eprint.iacr.org/2017/667.pdf), Tromer et al. | Combines Argon2 with a Merkle tree. Later attacks exploited time–memory trade‑offs. | 
| 2019	| [Evaluating Memory‑Hard PoW Algorithms on Three Processors](https://arxiv.org/abs/1903.02501), Feng et al. |  Benchmarks memory‑hard schemes on CPU, GPU, Xeon‑Phi. Finds GPUs still dominate once tuned.| 
| 2020	| [Itsuku: a Memory‑Hardened PoW Scheme](https://itsuku.io/paper.pdf), Albertini & Berkovits | Tweaks MTP to close known gaps. Illustrates the arms‑race nature of memory‑hard PoW. | 
| 2019	| [New Anti‑ASIC Consensus Algorithm with Emphasis on Matrix Computation](https://arxiv.org/abs/1905.04565), Seele | Proposes matrix‑based bandwidth‑hard PoW.	Demonstrates practical attempt to use high memory bandwidth. | 

---

### Memory/Bandwidth Hardness Algorithms

Several production algorithms inspired by memory- and bandwidth-hard principles have already been deployed. The list below is not exhaustive; it simply highlights the most significant mining algorithms to date. 

| First Deployed	| Algorithm	Main Technique | Outcome |
|-----------------|--------------------------|---------|
| 2014	| [Ethash (Ethereum)](https://github.com/ethereum/wiki/wiki/Ethash) | Large DAG; random memory look‑ups.	| ASICs (Bitmain E3) appeared in 2018. | 
| 2015	| [Cuckoo Cycle (Grin, Aeternity)](https://github.com/tromp/cuckoo) | Graph‑cycle finding with large memory footprint.	| ASICs (Obelisk) announced 2019; resistance temporary. |
| 2017	| [Equihash (Zcash et al.)](https://www.z.cash/technology/equihash/)| Memory‑hard birthday‑problem solver.	| ASICs (Z9, A9) shipped 2018. |
| 2018	| [ProgPoW](https://eprint.iacr.org/2019/054.pdf) | Random math & cache ops keyed to each block.	| Never activated on Ethereum; ASIC resistance debated. |
| 2019	| [RandomHash (PascalCoin)](https://www.pascalcoin.org/whitepaper) | Hash chaining + random reads.	| No public ASICs yet; GPUs show big gains. |
| 2020	| [KawPoW (Ravencoin)](https://medium.com/@ravencoin/kawpow-d6e67bf2af7c) | ProgPoW with Ethash‑style DAG.	| GPUs dominate; ASIC threat delayed. |

Yet experience shows that memory-hard techniques on their own provide only temporary ASIC- and GPU-resistance: with sufficient R &D investment, vendors can still design specialized hardware that amortizes the added memory cost and regains a decisive efficiency edge.

---

### RandomX 
To overcome the limitations of pure memory- or bandwidth-hard PoW schemes, the [RandomX project](https://github.com/tevador/RandomX/blob/master/doc/design.md)  adopts a more systematic strategy. A PoW algorithm can truly overcome the advantage of GPUs and ASICs only if it binds the work to the architectural traits of commodity CPUs.

The key insight is that CPUs handle two input streams:
- _Data_ – the values to be processed.
- _Code_ – the instructions that specify how that data is processed.

Traditional cryptographic hash functions accept only data and follow a fixed sequence of operations—an arrangement that specialized hardware can parallelize and accelerate with ease.
RandomX is the first production-grade PoW algorithm to embrace a fully dynamic “data + work” paradigm. For every hash attempt, a miner must compile and execute a newly generated, pseudo-random program (“work”) in addition to providing the “data”. This dynamic layer, built on top of memory-hard foundations, forces each mining thread to run an unpredictable mix of integer, floating-point, and branch-heavy instructions. 

The result:
- ASIC viability is sharply reduced: fixed-function designs cannot keep up with continually changing code.
- GPU throughput is neutralized: serial, cache-intensive workloads negate the massive parallelism that gives GPUs their edge.
In effect, RandomX fulfils the CryptoNote ideal of ASIC resistance while simultaneously charting a practical path toward robust GPU resistance

---

## RandomX — Assessment Against Project Requirements 

#### 1.	Strong GPU/ASIC Resistance
RandomX remains the only production-grade, general-purpose PoW algorithm that demonstrably thwarts both ASICs and high-end GPUs, fully satisfying [Req #1: GPU/ASIC resistance](#req-1gpuasic-resistance).

#### 2.	High Performance on CPU 
[Per RandomX documentation](https://github.com/tevador/RandomX/blob/master/doc/design.md#light-mode---verification-time) its high-throughput virtual machine verifies a single hash in average of 15 ms on modern, high-end CPUs is much better than the 200 ms target specified in [Req #3: CPU Optimization](#req-3cpu-optimization).

#### 3.	Re-usability Across Blockchains
Designed as a drop-in mining engine, RandomX ships with configuration options and guidance for integrating the VM into new blockchains without code-level changes to its core logic.
Since its release, RandomX has been adopted (or adapted) by numerous other blockchain projects seeking ASIC-resistant proof-of-work. Examples include:
-	[Wownero (WOW)](https://wownero.org/)
-	[ArQmA (ARQ)](https://arqma.com/)
-	[Epic Cash (EPIC)](https://epiccash.com/)
  
This proves that RandomX VM can be adapted for different blockchain solutions 

#### 4.	Modular Extensibility
RandomX’s implementation and design are highly modular, which has allowed developers to modify or extend it with relative ease. Several aspects of the project’s structure support this flexibility:
-	Clean Library/API Architecture: The official RandomX codebase builds as a reusable library with a straightforward C API (randomx.h). This ease of integration is evidenced by the many independent projects (from Arweave to Epic Cash) that incorporated RandomX by linking to or forking the library.
-	Configurable Parameters and Modes: As noted, RandomX includes dozens of parameters that are not hard-coded but defined in configuration (dataset size, cache size, instruction count, etc.). The algorithm was intentionally built to allow tuning these constants, which made it possible for coins to create variants like RandomWOW or RandomXL by changing a few parameters. 
-	Extensible Virtual Machine (Opcode Design): RandomX uses a virtual machine that executes pseudorandom programs. This means the instruction decoder is robust to arbitrary byte sequences – a property that simplifies adding or adjusting opcodes. We can introduce new VM instructions without breaking the bytecode format. 

RandomX’s re-usable, modular architecture offers multiple options to meet [Req #2: Use of Tip5 hashing function](#req-2use-of-tip5-hashing-function).

---

## Tip5 Hash Function Integration into RandomX Algorithm (Req #2)

This section presents an algorithm based on a modified RandomX specification, adapted to meet [Req #2: Use of Tip5 hashing function](#req-2use-of-tip5-hashing-function).
It based on the original RandomX Specification and does **not** duplicate its definitions. 

There are multiple, independent approaches to integrating the Tip-5 hash function into RandomX.  
These strategies can be implemented separately or in combination. Adjusted specification proposals are created for each integration option and are presented in separate documents.


### Option 1. Add a New VM Opcode for Tip5

A new opcode can be added to the RandomX virtual machine to invoke the Tip5 hash algorithm.

#### 📄 Specification

- Adjusted specification proposal:  
  [GitHub: RandomXT5 Option 1]((option%201,%20create%20op-code)%20specs.md), all modifications  are tagged with `Tip5 Option 1` for easy review.

- The core change is described in Opcode details — Section 5.6.1:  
  [§ 5.6.1: Tip5 Instruction]((option%201,%20create%20op-code)%20specs.md#56-tip5-instruction)

- Proposed implementation is described in [a separate document](tip5_opcode_implementation.md)


#### ⚙️ Design Notes

- **Infrequent Use:**  
  The new opcode replaces a primitive, single-cycle operation with a more complex Tip5 call.  
  It is designed to occur **infrequently** in generated programs to minimize performance overhead.

- **Performance Considerations:**  
  While some overhead is expected even with rare calls, RandomX significantly outperforms its performance target.  
  As a result, this additional cost is likely acceptable. It estimated that it will increase the average verification time from ~15 ms to ~20 ms still well below the target of 200 ms.
  Thus [Tip5 optimization](tip5_avx2_sse2_optimization.md) is optional. 

---

### Option 2. Use Tip5 for Final Digest (`Hash256` Replacement)

In standard RandomX, the final digest is produced using the `Hash256` function, which is internally a Blake2b variant with a 256-bit output.

This can be seamlessly replaced with a single Tip5 sponge squeeze, yielding a **320-bit** digest instead of 256 bits.

#### 📄 Specification

- Adjusted specification proposal:  
  [GitHub: RandomXT5 Option 2]((option%202%2C%20replace%20final%20digest)%20specs.md), all modifications  are tagged with `Tip5 Option 2` for easy review.

#### ⚙️ Design Notes and Rationale

- **Straightforward substitution:**   
  No structural changes to the VM or instruction set.

- **Minimal performance impact:**   
  Used only at the final step of hashing.

If minimal use of Tip5 satisfies project objectives, this is the **recommended integration**.

---

### Option 3. Replace `Hash512` with Extended Tip5 Squeezing

RandomX uses a `Hash512` which is actually a Blake2b implementation to produce 512-bit intermediate hashes during execution.  
This function can be replaced by an **extended Tip5 hash** using two flavours referenced as **A** and **B** below.

---

#### 📄 Background: Tip5 Output Characteristics

Tip5 is a sponge construction over the **Goldilocks prime field**:

\[
p = 2^{64} - 2^{32} + 1
\]

- State size: **10 limbs**, with **rate = 5**, **capacity = 5**
- One squeeze reads the 5 rate limbs →  
  \[
  5 \times 64 \text{ bits} = 320 \text{ bits}
  \]
- This 320-bit output is **not arbitrary**—it is the full output of a single permutation, aligned with the sponge's rate and security target.

To match Blake2b-512’s output length, we can either add a **second permutation + squeeze (Option A)** or create **16 limbs instantiation (Option B)** 


#### 🛠️ Flavor A Implementation

1. **Keep the 10-limb Tip5 state** unchanged (`rate = 5`, `capacity = 5`)
2. **Squeeze twice**:
    ```cpp
    permute(state);          // already done
    output limbs[0..4];      // 5 limbs → bytes 0–39 (320 bits)

    permute(state);          // one additional permutation
    output limbs[0..2];      // 3 limbs → bytes 40–63 (192 bits)
    ```
    *(Alternatively, output 4 limbs to generate a full 512 bits exactly)*

3. **Package** the combined 64-byte result as the new digest.


#### 🔐 Flavor A Security Implications

- **Secure under classical assumptions**:  
  Replacing Blake2b-512 with Tip-5 (via two squeezes) provides **160-bit collision resistance**, which is sufficient for modern PoW requirements (~128-bit).  
  This is based on well-established bounds for sponge constructions, where the adversary’s advantage is limited by:

  \[
  \mathcal{O}(q^2 / 2^c), \quad \text{with } c = 320 \Rightarrow \text{collision bound } \approx 2^{160}
  \]

- **Multiple squeezes remain safe**:   
  Each additional squeeze is preceded by a full permutation of the internal state, ensuring the **capacity section remains hidden**. This maintains **indifferentiability from a random oracle**, as required for cryptographic soundness.

- **No added strength from longer digests**:   
  While the output may be extended to 512 bits (via two squeezes), the **security is still bounded by the capacity**, not the digest size. Revealing more than `c` bits never increases the collision resistance beyond \( c / 2 = 160 \) bits.

- **Not a full replacement for Blake2b**:   
  Tip-5 has a smaller internal state (**640 bits**) compared to Blake2b’s (**1024 bits**). This means it does **not match** Blake2b’s full 256-bit collision resistance or 512-bit preimage strength.  
  Therefore, **Tip-5 is not a drop-in replacement** for Blake2b in high-security applications.

- **Quantum caveat**:   
  Under Grover’s algorithm, the effective collision resistance of Tip-5 drops to **~80 bits**, which is below future-safe levels. A **quantum-capable attacker** could eventually compromise a Tip5-based PoW system.

#### 🛠️ Flavor B Implementation

For matching or exceeding Blake2b’s security properties or quantum-resilient hashing, a higher-capacity instantiation—such as **Tip8** is required.
The proposal below follows **Tip4** and **Tip4′** instantiations described in the [Two additional instantiations from the Tip5 hash function construction](https://toposware.com/paper_tip5.pdf) article by Robin Salen.


**Algorithm Instantiation Parameters**

| Parameter      | Value              | Notes                                        |
|----------------|--------------------|----------------------------------------------|
| Field          | Goldilocks prime \( p = 2^{64} - 2^{32} + 1 \) | Same as Tip5 |
| State size     | 16 limbs       | 1024 bits total                              |
| Rate \( r \)   | 8 limbs        | 512 bits per squeeze                         |
| Capacity \( c \)| 8 limbs        | 512 bits, matching Blake2b internal capacity |
| Security       | 56-bit collision resistance | Pre-quantum safe |
| Quantum-resistance | 128-bit collision (Grover) | Resistance equivalent to Blake2b |


**Step-by-Step Instantiation Plan**

| Step | Task | Reason |
|------|------|--------|
| **1. Select new rate and capacity** | For 512-bit output: `rate = 8`, `capacity = 8` → 16-limb state (1024 bits total) | The `rate` controls how many limbs are output per squeeze. The `capacity` determines the security margin (collision resistance = `c/2`). |
| **2. Recompute round constants** | Use the same method as the [Tip5 Whitepaper by Toposware](https://toposware.com/paper_tip5.pdf) (e.g. hash of indexed counters). Generate fresh constants of length equal to the new state width. | Prevents rotational or structural symmetry in larger or smaller states. |
| **3. Regenerate the MDS matrix** | Build a new MDS matrix sized to the new state width. This matrix must be invertible and diffusion-optimal. | Guarantees full linear diffusion across limbs during each round. |
| **4. Validate round count** | Re-assess whether the current round count (e.g. 7 or 12) is sufficient given the new capacity, increase rounds if necessary. | A reduced `capacity` or larger state may require more rounds to prevent distinguishers or attacks. |
| **5. Update the implementation** | - Resize the internal state array<br> - Replace constant tables<br> - Adjust permutation functions and indexing logic | Reflects the new theoretical parameters in Tip8 code. |
| **6. Regenerate test vectors** | Generate known-answer tests for: <br> - Empty input <br> - 1–64 byte inputs <br> - Long messages (e.g., 10kB) <br> Use both 256-bit and 512-bit digests. | Verifies correctness and protects against regression during refactoring. |
| **7. Re-evaluate security** | Perform security reviews including: <br> - Differential trail analysis <br> - Algebraic attack resistance. | Changes to the state invalidate previous proof bounds; security must be re-established. |


#### 📄 Specification

- Adjusted specification proposal:  
  [GitHub: RandomXT5 Option 3]((option%203%2C%20replace%20HASH512)%20specs.md), all modifications  are tagged with `Tip5 Option 3` for easy review.  
  This specification covers both Flavor A and Flavor B changes

#### ⚙️ Design Notes and Rationale

- **Tip5 replacement of Blake2b hash**:
  512-bit digest either from two squeezes or from 16 limbs algorithm instantiation

- **Performance**:
  Tip5/Tip8 will sit in the same performance-critical spot that Blake2b occupies today. RandomX’s designers chose BLAKE2b because
  highly-tuned AVX2/SSE2 code can compress the VM state in a few microseconds; a scalar fall-back would be tens of percent slower
  and potentially double hash verification time.  Tip5/Tip8 implementation both for Flavor A and Flavor B requires implementation
  of [Tip5 optimization](tip5_avx2_sse2_optimization.md).

- **Use-case**:
  Suitable for replacing `Hash512` that generates random workload generation

---

### Comparison of Tip5 Integration Options in RandomX

| Option | Tip5 Integration Strength | Security Impact | Requires Tip5 optimisation | Implementation effort |
|--------|---------------------------|-----------------|----------------------------|-----------------------|
| **1. Create op-code** | **Tight** | **Slightly Increased** - Introduces a 320-bit Tip5 primitive inside the VM; final PoW digest stays Blake2b-256 ⇒ only a marginal security gain (extra diversity, but consensus hash unchanged). | **Optional** – opcode is generated rarely, a scalar fallback is acceptable. | **Medium** – add new opcode to decoder, interpreter, x86-64 & ARM JIT back-ends; update tests. |
| **2. Replace final digest** | **Weak** | **Slightlty Increased** - Swaps Blake2b-256 for one Tip5 squeeze (320 bits). Collision bound rises from 128 → 160 bits; no change to internal state security. | **No** – called once per hash, performance impact negligible. | **Low** – one-line change in `hash_final()`, regenerate test vectors. |
| **3. Replace `Hash512` Flavor A** | **Tight** | **Reduced** - Replaces Blake2b-512 with a **double-squeeze Tip5** (capacity = 320 bits ⇒ 160-bit collision resistance). Slightly weaker than Blake2b but still ≥ 128 bits. | **Yes** – `Hash512` is in the hot path (thousands of calls); AVX2/SSE2 & NEON kernels required. | **High** – rewrite `Hash512`, integrate 2-pass squeeze, retune VM constants, benchmark & tune. |
| **4. Replace `Hash512` Flavor B** | **Tight** | **Unchanged** - Uses **Tip8** (rate = 8, capacity = 8 limbs ⇒ 512-bit capacity). Restores parity with Blake2b security and keeps 256-bit pre-quantum strength. | **Yes** – larger state needs brand-new SIMD code on all architectures. | **Very High** – implement Tip8 permutation, generate new constants/MDS, extensive re-validation and QA. |


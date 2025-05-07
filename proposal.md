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

## Overview of Known Technologies
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

---

#### 2.	High Performance on CPU 
Its high-throughput virtual machine verifies a single hash in average of 15 ms on modern, high-end CPUs is much better than the target range specified in [Req #3: CPU Optimization](#req-3cpu-optimization).
The following figure shows the distribution of times to calculate 1 hash result using the light mode. 
![image](https://github.com/user-attachments/assets/0130b144-f503-48d3-ba61-126c4d082d96)

---

#### 3.	Re-usability Across Blockchains
Designed as a drop-in mining engine, RandomX ships with configuration options and guidance for integrating the VM into new blockchains without code-level changes to its core logic.
Since its release, RandomX has been adopted (or adapted) by numerous other blockchain projects seeking ASIC-resistant proof-of-work. Examples include:
-	[Wownero (WOW)](https://wownero.org/)
-	[ArQmA (ARQ)](https://arqma.com/)
-	[Epic Cash (EPIC)](https://epiccash.com/)
  
This proves that RandomX VM can be adapted for different blockchain solutions 

---

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

---

## Integration Strategies

There are multiple, independent approaches to integrating the Tip-5 hash function into RandomX.  
These strategies can be implemented separately or in combination. Adjusted specification proposals are created for each integration option and are presented in separate documents.

---

### 1. Add a New VM Opcode for Tip-5

A new opcode can be added to the RandomX virtual machine to invoke the Tip-5 hash algorithm.

#### 📄 Specification

- Adjusted specification proposal:  
  [GitHub: RandomXT5 Option 1]((option%201,%20create%20op-code)%20specs.md)
  All modifications  are tagged with `Tip5 Option 1` for easy review.

- The core change is described in Opcode details — Section 5.6.1:  
  [§ 5.6.1: Tip5 Instruction]((option%201,%20create%20op-code)%20specs.md#56-tip5-instruction)


#### ⚙️ Design Rationale

- **Infrequent Use:**  
  The new opcode replaces a primitive, single-cycle operation with a more complex Tip-5 call.  
  It is designed to occur **infrequently** in generated programs to minimize performance overhead.

- **Performance Considerations:**  
  While some overhead is expected even with rare calls, RandomX significantly outperforms its performance target.  
  As a result, this additional cost is likely acceptable. It estimated that it will increase the average verification time from ~15 ms to ~20 ms still well below the target of 200 ms.

- **Shared Optimizations:**  
  General optimization strategies applicable to all Tip-5 integration methods are discussed in a later section.

---

### 2. Use Tip-5 for Final Digest (`Hash256` Replacement)

In standard RandomX, the final digest is produced using the `Hash256` function, which is internally a Blake2b variant with a 256-bit output.

This can be seamlessly replaced with a single Tip-5 sponge squeeze, yielding a **320-bit** digest instead of 256 bits.

#### 📄 Specification

- Adjusted pecification proposal:  
  [GitHub: RandomXT5 Option 1]((option%202%2C%20replace%20final%20digest)%20specs.md))
  All modifications  are tagged with `Tip5 Option 2` for easy review.

#### ⚙️ Design Rationale

- **Straightforward substitution** — no structural changes to the VM or instruction set.
- **Minimal performance impact** — used only at the final step of hashing.

If minmal use of Tip5 satisfies project objectives, this is the **recommended integration**.

---

### Comparison of Tip-5 Integration Options in RandomX

| Option | Description | Output Size | Collision Resistance | Security Impact | Performance Impact | Notes |
|--------|-------------|-------------|-----------------------|------------------|--------------------|-------|
| **1. VM Opcode** | Add `tip5` as a new RandomX VM opcode | Variable (depends on use) | Up to 160-bit | ✅ Secure if opcode is rare and deterministic | ⚠️ Moderate — more costly than primitive ops | Controlled by opcode frequency; enables VM programmability |
| **2. Replace `Hash256`** | Swap RandomX's `Hash256` (Blake2b-256) with 1 Tip-5 squeeze | 320-bit | 160-bit | ✅ Secure — capacity-driven, ≥128-bit safe | ⚪️ Neutral to Slight — end-of-algorithm only | Increases output size by 64 bits; no runtime slowdowns |
| **3. Replace `Hash512`** | Replace Blake2b-512 with 2 Tip-5 squeezes (permute + squeeze) | 512-bit | 160-bit | ✅ Secure — same sponge rules apply | ⚠️ Slight — extra permutation adds ~0.1 μs | Still below collision threshold; safe for PoW use |

✅ = Cryptographically sound  
⚠️ = Requires tuning or hardened implementation  
⚪️ = Minimal or context-dependent effect

---

### Summary

- **Security**: All options preserve or improve the collision resistance compared to the original Blake2b-based design.
- **Performance**: Option 1 (VM opcode) introduces the highest runtime cost but provides the most flexibility. Options 2 and 3 offer cryptographic upgrades with minimal impact, particularly suitable for final hashing.
- **Use-case**:  
  - Use **Option 1** for enhanced entropy or novel VM logic.  
  - Use **Options 2/3** for drop-in cryptographic upgrades.


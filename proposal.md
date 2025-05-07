## Task Description and Requirements
This proposal addresses the development of a Proof-of-Work (PoW) mining algorithm designed to fulfill the following specific criteria:
#### Req 1.	GPU/ASIC RESISTANCE
The algorithm must be inherently resistant to parallelization, effectively discouraging mining performance advantages gained through GPU or ASIC hardware acceleration.
#### Req 2.	TIP5 CRYPTOGRAPHY
The algorithm must use the Tip5 hashing standard, preserving its cryptographic integrity and security strength.
#### Req 3.	CPU OPTIMIZATION
The algorithm should exhibit optimized performance on standard CPU architectures, aiming to validate a single hash operation within approximately 200 microseconds on high-end contemporary CPU hardware.

Meeting mandatory requirements (1) and (2) is essential. Achieving requirement (3) is highly desirable and will significantly enhance practical applicability.

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

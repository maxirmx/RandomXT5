## Task Description and Requirements
This proposal addresses the development of a Proof-of-Work (PoW) mining algorithm designed to fulfill the following specific criteria:
#### 1.	GPU/ASIC RESISTANCE
The algorithm must be inherently resistant to parallelization, effectively discouraging mining performance advantages gained through GPU or ASIC hardware acceleration.
#### 1.	TIP5 CRYPTOGRAPHY
The algorithm must use the Tip5 hashing standard, preserving its cryptographic integrity and security strength.
#### 1.	CPU OPTIMIZATION
The algorithm should exhibit optimized performance on standard CPU architectures, aiming to validate a single hash operation within approximately 200 microseconds on high-end contemporary CPU hardware.

Meeting mandatory requirements (1) and (2) is essential. Achieving requirement (3) is highly desirable and will significantly enhance practical applicability.

## Overview of Known Technologies
### Academic Research
The CryptoNote Whitepaper offered the first systematic treatment of ASIC-resistant mining, proposing PoW functions that hinge on memory hardness and high memory-bandwidth demands. 
Numerous variations on this theme have since appeared in academic literature (the list below is not intended to be exhaustive).
Year	Title / Authors	Core Idea	Why It Matters
2013	“CryptoNote Whitepaper,” Nicolas van Saberhagen
https://bytecoin.org/old/whitepaper.pdf 
Introduces egalitarian PoW and memory‑hard functions.	Seminal work inspiring later ASIC‑resistant designs.
2016	“Equihash: Asymmetric Proof‑of‑Work Based on the Generalized Birthday Problem,” Biryukov & Khovratovich
https://eprint.iacr.org/2016/759.pdf 
Formalizes memory‑hard PoW; large RAM needed to build the proof, instant to verify.	Widely deployed; ASICs appeared within two years.
2017	“Bandwidth‑Hard Functions for ASIC Resistance,” Dziembowski et al.
https://eprint.iacr.org/2017/378.pdf 
Emphasizes saturating memory bandwidth, not just capacity.	Shows ASICs can still gain if only capacity is targeted.
2017	Merkle‑Tree Proof (MTP‑Argon2), Tromer et al.
https://eprint.iacr.org/2017/667.pdf
Combines Argon2 with a Merkle tree.	Later attacks exploited time–memory trade‑offs.
2019	“Evaluating Memory‑Hard PoW Algorithms on Three Processors,” Feng et al.
https://arxiv.org/abs/1903.02501 
Benchmarks memory‑hard schemes on CPU, GPU, Xeon‑Phi.	Finds GPUs still dominate once tuned.
2020	“Itsuku: a Memory‑Hardened PoW Scheme,” Albertini & Berkovits
https://itsuku.io/paper.pdf
Tweaks MTP to close known gaps.	Illustrates the arms‑race nature of memory‑hard PoW.
2019	“New Anti‑ASIC Consensus Algorithm with Emphasis on Matrix Computation,” Seele
https://arxiv.org/abs/1905.04565 
Proposes matrix‑based bandwidth‑hard PoW.	Demonstrates practical attempt to use high memory bandwidth.


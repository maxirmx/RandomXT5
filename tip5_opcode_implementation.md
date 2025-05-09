## Tip5 op-code for RandomX Virtual Machine

This document specifies patches to RandomX Virtual Machine core that are required to support Tip5 op-code
The code snippets are tested against few trivial cases to prove the concept.

### 1  Decoder & interpreter layer (`interpreter.cpp`)

| step | patch |
|------|-------|
| **1.1** | Add `OP_TIP5` to `enum InstructionType` (next free slot after `OP_IMUL_RCP`). |
| **1.2** | Extend the decode switch (≈ line 190) to recognise opcode `0x3A`. Populate `inst->rd`, `inst->rs`, `inst->imm` (16‑bit *count*; 0 ≡ 65 536). |
| **1.3** | Interpreter execution (`executeInstruction`):<br>`tip5_permute_block(ptr, count)` performs the in‑place permutation loop.<br>Return value → `rd = state[0] ^ state[1]`. |
| **1.4** | Re‑run the existing “full‑program” interpreter tests; add a seed that exercises the new opcode in ≥ 10 % of generated byte‑code. |

---

### 2  JIT back‑end (x86‑64)

#### 2.1 Register allocation
| purpose | GPR | notes |
|---------|-----|-------|
| scratchpad base + offset | **RAX** | holds `scratchpad + rs + i*80` |
| loop counter (`count`)   | **RBX** | zero‑extend `imm16`, checked for zero |
| temporary load/store     | **RDX** | address calc |
| result fold              | **RCX** | `xor rcx, state[0]` just before loop‑end |

YMM/ZMM registers hold the permutation state (selected at runtime).

#### 2.2 Code emission skeleton
```asm
; prologue
mov     rax, [vm.scratchpad]
add     rax, r<rs>
movzx   ebx, imm16
jrcxz   .end

.loop:
; ---- load 80 B ----
vmovdqu ymm0,  [rax]         ; first 64 B
vmovdqu xmm4,  [rax+64]      ; last 16 B

; ---- Tip5 permutation ----
call    [tip5_dispatcher]    ; CPUID-gated stub (scalar/SSE2/AVX2)

; ---- store 80 B ----
vmovdqu [rax],    ymm0
vmovdqu [rax+64], xmm4

; fold into GP reg
xor     rcx, rax             ; cheap, data-dependent
add     rax, 80
dec     ebx
jnz     .loop
.end:
```
---

### 3  JIT back‑end (ARM64)

#### 3.1 Target baseline
* AArch64‑v8.2‑A (Apple M‑series).  
* 128‑bit NEON SIMD path plus scalar fallback.  
* Future SVE256/SVE2 behind `CPU_HAS_SVE2`.

#### 3.2 Register allocation
| purpose | GPR | notes |
|---------|-----|-------|
| scratchpad pointer | **X0** | `base + rs` |
| loop counter       | **W1** | `imm16` (0 ≡ 65 536) |
| temp loads/stores  | **X2** | address calc |
| result XOR         | **X3** | folded result |

| purpose | SIMD | width |
|---------|------|-------|
| state limbs 0–7 | **V0–V3** | 128‑bit |
| limbs 8–9       | **V4**    | lower 64 b |

Round constants pre‑loaded in **V16–V19** (non‑volatile).

#### 3.3 Code emission outline
```asm
; X0 = scratchpad + rs
; W1 = count
cbz     w1, end

loop:
; ---- 80 B load ----
ld1     {v0.16b-v3.16b}, [x0]     ; 64 B
ldr     q4, [x0,#64]              ; 16 B

; ---- permutation ----
bl      tip5_dispatch_neon        ; scalar/NEON/SVE2

; ---- 80 B store ----
st1     {v0.16b-v3.16b}, [x0]
str     q4, [x0,#64]

; fold into GP register
eor     x3, x3, v0.d[0]

add     x0, x0, #80
subs    w1, w1, #1
b.ne    loop
end:
```

* tip5_dispatch_neon mirrors x86: scalar, NEON, SVE2.  
* Constant‑time: no data‑dependent branches inside permutation.  
* Build stubs with `-fno-builtin-memcpy -fno-builtin-memmove`.

---

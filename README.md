# Performance Optimization of NTRU+ KeyGen via Montgomery Batch Inversion

## Overview
This project provides an optimized implementation aimed at maximizing the KeyGen performance of **NTRU+ (KEM 768, 864, 1152)**, a finalist in Round 2 of the Korean Post-Quantum Cryptography Competition (KpqC).

We addressed the performance bottleneck caused by repetitive calls to the costly modular inverse operation (`fqinv`) in the existing reference implementation (Clean version) by applying the **Montgomery Batch Inversion** technique. This approach significantly enhances computational efficiency and reduces KeyGen latency.

---

## Performance Benchmark

The following results were measured using the `speed_kem` benchmarking tool from **liboqs**.

**Test Environment:**
* **CPU:** [Your CPU Model, e.g., Apple M1 Pro / Intel Core i7-12700K]
* **OS:** [Your OS Version, e.g., macOS Sonoma / Ubuntu 22.04 LTS]
* **Compiler:** [Your Compiler Version, e.g., clang 14.0.0 / gcc 11.2]

### Summary

| Metric | Baseline (Clean) | Optimized (Batch Inv) | Improvement |
| :--- | :---: | :---: | :---: |
| **Time mean (Latency)** | 42.664 μs | 28.209 μs | **33.9% Faster** |
| **Iterations (3sec)** | ~ 70,317 | ~ 106,349 | **51.2% Increased** |

**Analysis:** We achieved approximately **34% speed improvement** compared to the baseline implementation. Furthermore, throughput per unit time increased by **more than 1.5x**, significantly reducing handshake overhead in high-performance network environments.

### Evidence
<details>
<summary><b>View Benchmark Screenshots (Click to expand)</b></summary>

**1. Baseline (Before Optimization)**
<img width="100%" alt="Baseline Result" src="https://github.com/user-attachments/assets/6823721f-5e7c-4cea-b0a1-dbc631715f48" />

**2. After Improvement**
<img width="100%" alt="Optimized Result" src="https://github.com/user-attachments/assets/6958f38d-3472-42a6-82ea-751b6ee81409" />

</details>

---

## Technical Details

### 1. Bottleneck Resolution
- **Problem:** The existing `poly_baseinv` function iteratively calls the costly `fqinv` (modular inverse) operation **144 times** inside a loop for polynomial coefficient processing.
- **Solution:** We introduced the Montgomery Batch Inversion algorithm to replace the 144 inverse operations with **a single** `fqinv` call and relatively low-cost multiplication operations.

### 2. Implementation Details
- **New Implementation Directory:**
  `src/kem/ntru_plus/KpqClean_ver2_NTRU_PLUS_KEM576_clean_montgomery-batch-normalization/`
  
- **Algorithm Optimization (`ntt.c`, `poly.c`):**
  - `fq_batchinv`: Implemented batch logic to calculate inverses for multiple elements simultaneously.
  - `baseinv_calc_t_values`, `baseinv_apply_inv`: Implemented auxiliary functions for batch processing.
  - `poly_baseinv`: Completely rewrote the loop-based individual inverse calculation logic to utilize batch processing logic.

- **Build System & Symbol Management:**
  - `CMakeLists.txt`: Added the `_opt` target library and link settings.
  - Changed function suffixes from `_clean` to `_opt` to avoid namespace collisions.
  - Created a wrapper file (`kem_ntru_plus_kem576_opt.c`) for `liboqs` integration and registered the algorithm identifier.

---

## Correctness Verification
This optimized implementation guarantees compatibility with the original algorithm.
- **KAT (Known Answer Test):** Verified that it passes `PQCgenKAT_kem` and generates test vectors identical to the Reference implementation.
- **Valgrind Test:** Confirmed no memory leaks or invalid access errors.

---

## Build & Test Instructions

Follow the steps below to verify the performance improvements yourself.

### 1. Build
```bash
# Create build directory
mkdir build && cd build

# Enable NTRU+ and set optimization options
# (Set ENABLE_KEM_ntru_plus_kem576_avx2=ON if AVX2 is supported)
cmake -DOQS_ENABLE_KEM_NTRU_PLUS=ON -DOQS_ENABLE_KEM_ntru_plus_kem576_avx2=OFF ..

# Compile benchmarking tool
make speed_kem

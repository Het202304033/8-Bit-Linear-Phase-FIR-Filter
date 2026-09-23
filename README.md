# 8-Bit Linear-Phase FIR Filter

## 1. Adder Architectures: First Principles to Trade-Off Analysis
* **Full Adder:** Built from basic logic gates, deriving worst-case delay equations (T_sum, T_cout) for generated versus propagated carry to confirm the specific input patterns that trigger the true critical path.
* **Ripple Carry Adder (RCA):** Derived the linear worst-case delay growth formula (n × T_carry) and identified the exact worst-case vector (11...1 + 00...01 + Cin=1). Connected these theoretical limits to real design consequences, such as the bit-width growth needed for overflow-free sums and the RCA's role as the final carry-propagate stage in multipliers.
* **Carry Look-Ahead Adder (CLA):** Implemented as two cascaded 4-bit blocks rather than a single flat 8-bit block to actively avoid high-fanout generate/propagate logic, demonstrating a practical area/speed trade-off. Achieved a ~2x speedup over the RCA on identical worst-case inputs.
* **Carry Save Adder (CSA):** Analyzed the conceptual split between carry-propagate and carry-save families. Extended CSA addition to three operands and derived the general k-operand CSA-tree delay formula: (k-2)T_CSA + T_CPA.
* **Architecture Trade-Offs:** Established a concrete progression matrix: **RCA** (simple, slow, small) -> **CLA** (fast, increased area) -> **CSA** (fast multi-operand accumulation requiring a final CPA resolution).

## 2. Multiplier Design & Comparison
* **Array Multiplier:** Constructed from a CSA chain and a final RCA, deriving total delay as a direct function of CSA stages and the final carry chain.
* **Wallace Tree Multiplier:** Implemented column-reduction stages (using FAs/HAs down to 2 rows) and a fast adder for final row reduction. Mathematically derived how reduction stages scale sub-linearly (log) with min(m,n), proving why Wallace trees scale better than array multipliers for wide operands.
* **Statistical Timing Analysis:** Ran exhaustive input simulations to quantify performance rather than relying on theoretical assertions. Computed mean, median, standard deviation, and max delay across multiplier types. Used the mean-vs-median gap and standard deviation to analyze "average-case vs. worst-case" timing predictability across balanced/unbalanced Wallace trees and RCA/CLA final adders.
* **Custom Mixed-Sign Multiplier:** Designed a specialized multiplier for the FIR datapath where one operand is always positive (post-pre-addition samples) and the other is signed (filter coefficients). Solved the non-trivial partial-product generation problem required when standard unsigned array/Wallace structures fail to handle single-signed operands without modified sign-extension handling.

## 3. Custom Timing Methodology
* **Transistor-Level Delay Extraction:** Eliminated standard uniform gate delay assumptions. Extracted transition-dependent (rise vs. fall) and input-dependent delays directly from custom Cadence Virtuoso transistor-level gate designs.
* **Accurate RTL Simulation:** Encoded these precise analog delays into structural Verilog using `specify` blocks, ensuring Vivado timing simulations accurately matched real-world transistor behavior. Used this to find true worst-case input vectors via exhaustive simulation.

## 4. Pipelining & Timing Closure
* **Constraints Analysis:** Derived setup and hold constraints for a two-register pipeline stage containing zero combinational logic between them.
* **Hold Violation Resolution:** Analyzed why pipelining a fast combinational block (like a multiplier) can inadvertently introduce hold violations while simultaneously improving throughput. Engineered and evaluated three concrete architectural fixes (delay elements, slower source flip-flops, and lockup latches) against their area and power costs.

## 5. FIR Filter Design & Fixed-Point DSP
* **Filter Architecture:** Designed a 15-tap linear-phase low-pass FIR filter from base specifications (cutoff frequency, stopband attenuation, and sampling frequency derivation).
* **Fixed-Point Quantization:** Selected Q1.7 fixed-point format for coefficients and samples based on actual coefficient ranges and 2's-complement arithmetic support. Quantified the exact degradation of magnitude/phase response and stopband attenuation caused by quantization.
* **SQNR Verification:** Analytically computed and verified the Signal-to-Quantization-Noise Ratio (SQNR) in MATLAB across three implementation tiers: ideal (floating-point), direct form (finite word length), and structurally optimized.
* **Hardware Optimization:** Exploited the filter's mathematical properties (alternating zero coefficients and coefficient symmetry) to reduce the required multiplier count from 15 to 5 via pre-addition. Identified the structure as a half-band filter and derived practical system benefits, such as resampling the output at half-rate to relax DAC requirements.
* **Structural Verilog RTL:** Translated the fixed-point/SQNR analysis into a working physical datapath using custom signed carry-save multipliers, carry-save adders, and overflow saturation logic.

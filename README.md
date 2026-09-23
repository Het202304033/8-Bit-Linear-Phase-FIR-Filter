# 8-Bit-Linear-Phase-FIR-Filter

1. Adder architectures — from first principles to trade-off analysis

Full Adder: built from basic gates, then derived worst-case delay equations by hand (T_sum, T_cout for generated vs. propagated carry) and confirmed which input pattern triggers the true critical path.
Ripple Carry Adder (RCA): derived that worst-case delay grows linearly with bit-width (n × T_carry), identified the exact worst-case vector (11...1 + 00...01 + Cin=1), and connected this to real design consequences — width growth needed for overflow-free sums, and RCA's use as the final carry-propagate stage inside multipliers.
Carry Look-Ahead Adder (CLA): implemented as two cascaded 4-bit blocks (rather than one flat 8-bit block) specifically to avoid high-fanout generate/propagate logic — a real area/speed trade-off decision, not just a textbook one. Measured ~2x speedup over RCA on the same worst-case inputs.
Carry Save Adder (CSA): understood the conceptual split between carry-propagate and carry-save adder families, extended CSA addition to 3 operands, and derived the general k-operand CSA-tree delay formula (k-2)T_CSA + T_CPA.
End-to-end trade-off table in your head: RCA (simple, slow, small) → CLA (fast, more area) → CSA (fast multi-operand accumulation, needs a final CPA to resolve).

2. Multiplier design and comparison

Array multiplier: built from a CSA chain + final RCA; derived total delay as a function of number of CSA stages and final carry chain.
Wallace Tree multiplier: implemented column-reduction stages with FAs/HAs down to 2 rows, then a fast adder for row reduction; derived how the number of reduction stages s grows sub-linearly (logarithmically) with min(m,n) — the actual mathematical reason Wallace trees scale better than array multipliers for wide operands.
Quantified this rather than just asserting it: ran exhaustive input simulations, computed mean/median/std-dev/max delay across multiplier types, and used mean-vs-median gap and standard deviation as evidence for "average-case vs. worst-case" and "timing predictability" trade-offs — with balanced vs. deliberately unbalanced Wallace trees, and RCA vs. CLA as the final adder.
Custom signed/mixed-sign multiplier (in the FIR project): designed a multiplier where one operand is always positive (samples after pre-addition) and the other can be positive or negative (filter coefficients) — a genuinely non-trivial partial-product generation problem, since standard unsigned array/Wallace structures don't handle a single signed operand correctly without modified partial-product generation or sign-extension handling.

3. Timing methodology (this is a strong, less-common skill to highlight)

Didn't assume uniform gate delay — extracted transition-dependent (rise vs. fall), input-dependent delays from your own Cadence Virtuoso transistor-level gate designs, and encoded them into structural Verilog using specify blocks so Vivado timing simulation matched real transistor behavior.
Used this to find true worst-case input vectors by exhaustive simulation rather than assumption.

4. Pipelining and timing closure

Derived setup and hold constraints for a two-register pipeline stage with zero combinational logic between them, and reasoned about why pipelining a fast combinational block (like a multiplier) can introduce hold violations even as it improves throughput — plus three concrete fixes (delay elements, slower source flip-flop, lockup latches) with their area/power costs.

5. FIR filter design and fixed-point DSP

Designed a 15-tap linear-phase low-pass FIR filter from spec (cutoff, stopband attenuation, sampling frequency derivation).
Chose Q1.7 fixed-point format for coefficients and samples based on actual coefficient range and existing 2's-complement adder/multiplier support; quantified the real effect of quantization on magnitude/phase response and stopband attenuation (not just "there's some error").
Computed SQNR analytically and verified it in MATLAB across three implementations: ideal (floating point), direct form (finite word length), and an optimized structure.
Recognized and exploited the filter's structure — alternating zero coefficients + coefficient symmetry — to cut multiplier count from 15 to 5 via pre-addition, identified it as a half-band filter, and derived the practical consequence (output can be resampled at half rate, relaxing DAC requirements).
Took this to hardware: structural Verilog implementation using your own signed carry-save multipliers and carry-save adders, with overflow saturation logic — connecting the fixed-point/SQNR analysis directly to a working RTL datapath.

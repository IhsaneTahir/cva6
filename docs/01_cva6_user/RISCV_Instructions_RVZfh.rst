..
   Copyright (c) 2026 OpenHW Foundation

   SPDX-License-Identifier: Apache-2.0 WITH SHL-2.1

.. Level 1
   =======

   Level 2
   -------

   Level 3
   ~~~~~~~

   Level 4
   ^^^^^^^

.. _cva6_riscv_instructions_RVZfh:

*Applicability of this chapter to configurations:*

This extension is supported by CVA6 (``RVZFH`` configuration parameter, or the deprecated ``XF16`` alias) but is
not enabled in any of the currently verified CVA6 configurations documented in this manual (CV32A60AX, CV32A60X,
CV64A6_MMU).

RVZfh: Half-Precision Floating-Point Instructions
---------------------------------------------------

Zfh adds 16-bit half-precision (``H``, IEEE 754 binary16) floating-point instructions compliant with the
IEEE 754-2008 arithmetic standard. Zfh depends on the single-precision floating-point extension ``F``. CVA6
implements the complete Zfh extension rather than the minimal Zfhmin subset.

Half-precision values are held in the low 16 bits of a floating-point register and NaN-boxed: the upper
``FLEN``-16 bits are set to 1 for a valid ``H`` value; a value that is not properly NaN-boxed is treated as the
canonical NaN when read. The RISC-V canonical NaN for ``H`` is ``0x7e00``.

.. csv-table::
   :widths: auto
   :align: left
   :header: "RV32", "RV64", "Mnemonic"

   "✔", "✔", "flh rd, imm(rs1)"
   "✔", "✔", "fsh rs2, imm(rs1)"
   "✔", "✔", "fadd.h rd, rs1, rs2"
   "✔", "✔", "fsub.h rd, rs1, rs2"
   "✔", "✔", "fmul.h rd, rs1, rs2"
   "✔", "✔", "fdiv.h rd, rs1, rs2"
   "✔", "✔", "fsqrt.h rd, rs1"
   "✔", "✔", "fmadd.h rd, rs1, rs2, rs3"
   "✔", "✔", "fmsub.h rd, rs1, rs2, rs3"
   "✔", "✔", "fnmsub.h rd, rs1, rs2, rs3"
   "✔", "✔", "fnmadd.h rd, rs1, rs2, rs3"
   "✔", "✔", "fsgnj.h rd, rs1, rs2"
   "✔", "✔", "fsgnjn.h rd, rs1, rs2"
   "✔", "✔", "fsgnjx.h rd, rs1, rs2"
   "✔", "✔", "fmin.h rd, rs1, rs2"
   "✔", "✔", "fmax.h rd, rs1, rs2"
   "✔", "✔", "fcvt.w.h rd, rs1"
   "✔", "✔", "fcvt.wu.h rd, rs1"
   "✔", "✔", "fcvt.h.w rd, rs1"
   "✔", "✔", "fcvt.h.wu rd, rs1"
   "", "✔", "fcvt.l.h rd, rs1"
   "", "✔", "fcvt.lu.h rd, rs1"
   "", "✔", "fcvt.h.l rd, rs1"
   "", "✔", "fcvt.h.lu rd, rs1"
   "✔", "✔", "fcvt.s.h rd, rs1"
   "✔", "✔", "fcvt.h.s rd, rs1"
   "✔ (with RVD)", "✔ (with RVD)", "fcvt.d.h rd, rs1"
   "✔ (with RVD)", "✔ (with RVD)", "fcvt.h.d rd, rs1"
   "✔", "✔", "fmv.x.h rd, rs1"
   "✔", "✔", "fmv.h.x rd, rs1"
   "✔", "✔", "feq.h rd, rs1, rs2"
   "✔", "✔", "flt.h rd, rs1, rs2"
   "✔", "✔", "fle.h rd, rs1, rs2"
   "✔", "✔", "fclass.h rd, rs1"

Half-Precision Load and Store Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **FLH**: Load Half-precision

    **Format**: flh rd, imm(rs1)

    **Description**: loads a 16-bit value from memory into the low 16 bits of ``rd``, and NaN-boxes the result.
    The effective address is obtained by adding register ``rs1`` to the sign-extended 12-bit offset. FLH does
    not modify the bits being transferred; the payload of a non-canonical NaN is preserved.

    **Pseudocode**: ``f[rd] = NaN-box(M[x[rs1] + sext(imm[11:0])][15:0])``

    **Invalid values**: NONE

    **Exception raised**: Load address misaligned

- **FSH**: Store Half-precision

    **Format**: fsh rs2, imm(rs1)

    **Description**: stores the low 16 bits of ``rs2`` to memory. The effective address is obtained by adding
    register ``rs1`` to the sign-extended 12-bit offset. FSH ignores all but the lower 16 bits of ``rs2`` and does
    not modify the bits being transferred.

    **Pseudocode**: ``M[x[rs1] + sext(imm[11:0])][15:0] = f[rs2][15:0]``

    **Invalid values**: NONE

    **Exception raised**: Store/AMO address misaligned

Half-Precision Computational Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **FADD.H**: Half-precision Add

    **Format**: fadd.h rd, rs1, rs2

    **Description**: adds the half-precision values in ``rs1`` and ``rs2``, and writes the rounded sum to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(f[rs1] + f[rs2]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FSUB.H**: Half-precision Subtract

    **Format**: fsub.h rd, rs1, rs2

    **Description**: subtracts the half-precision value in ``rs2`` from ``rs1``, and writes the rounded result to
    ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(f[rs1] - f[rs2]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FMUL.H**: Half-precision Multiply

    **Format**: fmul.h rd, rs1, rs2

    **Description**: multiplies the half-precision values in ``rs1`` and ``rs2``, and writes the rounded product
    to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(f[rs1] * f[rs2]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FDIV.H**: Half-precision Divide

    **Format**: fdiv.h rd, rs1, rs2

    **Description**: divides the half-precision value in ``rs1`` by ``rs2``, and writes the rounded quotient to
    ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(f[rs1] / f[rs2]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``DZ``, ``OF``, ``UF``, ``NX``

- **FSQRT.H**: Half-precision Square Root

    **Format**: fsqrt.h rd, rs1

    **Description**: computes the rounded square root of the half-precision value in ``rs1`` and writes it to
    ``rd``. ``rs2`` must be zero.

    **Pseudocode**: ``f[rd] = NaN-box(round(sqrt(f[rs1])))``

    **Invalid values**: a non-zero ``rs2`` field is an illegal instruction.

    **Exception raised**: ``NV``, ``NX``

- **FMADD.H**: Half-precision Fused Multiply-Add

    **Format**: fmadd.h rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, adds ``rs3``, and writes the rounded result to ``rd``, with
    only a single rounding step applied to the final sum.

    **Pseudocode**: ``f[rd] = NaN-box(round(f[rs1] * f[rs2] + f[rs3]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FMSUB.H**: Half-precision Fused Multiply-Subtract

    **Format**: fmsub.h rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, subtracts ``rs3``, and writes the rounded result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(f[rs1] * f[rs2] - f[rs3]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FNMSUB.H**: Half-precision Negate Fused Multiply-Subtract

    **Format**: fnmsub.h rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, negates the product, adds ``rs3``, and writes the rounded
    result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(-(f[rs1] * f[rs2]) + f[rs3]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FNMADD.H**: Half-precision Negate Fused Multiply-Add

    **Format**: fnmadd.h rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, negates the product, subtracts ``rs3``, and writes the
    rounded result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(-(f[rs1] * f[rs2]) - f[rs3]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FSGNJ.H**: Half-precision Sign-Inject

    **Format**: fsgnj.h rd, rs1, rs2

    **Description**: writes the value of ``rs1`` to ``rd``, but with the sign bit replaced by the sign bit of
    ``rs2``. Performs no arithmetic; NaN payloads are preserved.

    **Pseudocode**: ``f[rd] = NaN-box({f[rs2][15], f[rs1][14:0]})``

    **Invalid values**: NONE

    **Exception raised**: NONE

- **FSGNJN.H**: Half-precision Sign-Inject Negate

    **Format**: fsgnjn.h rd, rs1, rs2

    **Description**: writes the value of ``rs1`` to ``rd``, but with the sign bit replaced by the negation of the
    sign bit of ``rs2``.

    **Pseudocode**: ``f[rd] = NaN-box({~f[rs2][15], f[rs1][14:0]})``

    **Invalid values**: NONE

    **Exception raised**: NONE

- **FSGNJX.H**: Half-precision Sign-Inject XOR

    **Format**: fsgnjx.h rd, rs1, rs2

    **Description**: writes the value of ``rs1`` to ``rd``, but with the sign bit replaced by the XOR of the sign
    bits of ``rs1`` and ``rs2``.

    **Pseudocode**: ``f[rd] = NaN-box({f[rs1][15] ^ f[rs2][15], f[rs1][14:0]})``

    **Invalid values**: NONE

    **Exception raised**: NONE

- **FMIN.H**: Half-precision Minimum

    **Format**: fmin.h rd, rs1, rs2

    **Description**: writes the smaller of the half-precision values in ``rs1`` and ``rs2`` to ``rd``. If one
    operand is a quiet NaN and the other is not NaN, the non-NaN operand is returned without raising an exception.
    If both are NaN, the canonical NaN is returned.

    **Pseudocode**: ``f[rd] = NaN-box(min(f[rs1], f[rs2]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``

- **FMAX.H**: Half-precision Maximum

    **Format**: fmax.h rd, rs1, rs2

    **Description**: writes the larger of the half-precision values in ``rs1`` and ``rs2`` to ``rd``, with the
    same NaN-handling rules as FMIN.H.

    **Pseudocode**: ``f[rd] = NaN-box(max(f[rs1], f[rs2]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``

Half-Precision Conversion and Move Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **FCVT.W.H**: Convert Half-precision to Signed 32-bit Integer

    **Format**: fcvt.w.h rd, rs1

    **Description**: converts the half-precision value in ``rs1`` to a signed 32-bit integer in ``x[rd]``, rounded
    per the ``rm`` field. On RV64, the result is sign-extended to 64 bits.

    **Pseudocode**: ``x[rd] = sext(i32(round(f[rs1])))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``NX``

- **FCVT.WU.H**: Convert Half-precision to Unsigned 32-bit Integer

    **Format**: fcvt.wu.h rd, rs1

    **Description**: converts the half-precision value in ``rs1`` to an unsigned 32-bit integer in ``x[rd]``,
    rounded per the ``rm`` field. On RV64, the result is sign-extended to 64 bits.

    **Pseudocode**: ``x[rd] = sext(u32(round(f[rs1])))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``NX``

- **FCVT.H.W**: Convert Signed 32-bit Integer to Half-precision

    **Format**: fcvt.h.w rd, rs1

    **Description**: converts the signed 32-bit integer in ``x[rs1]`` to a half-precision value, rounded per the
    ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(h(i32(x[rs1]))))``

    **Invalid values**: NONE

    **Exception raised**: ``OF``, ``NX``

- **FCVT.H.WU**: Convert Unsigned 32-bit Integer to Half-precision

    **Format**: fcvt.h.wu rd, rs1

    **Description**: converts the unsigned 32-bit integer in ``x[rs1]`` to a half-precision value, rounded per the
    ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(h(u32(x[rs1]))))``

    **Invalid values**: NONE

    **Exception raised**: ``OF``, ``NX``

- **FCVT.L.H**: Convert Half-precision to Signed 64-bit Integer (RV64 only)

    **Format**: fcvt.l.h rd, rs1

    **Description**: converts the half-precision value in ``rs1`` to a signed 64-bit integer in ``x[rd]``, rounded
    per the ``rm`` field.

    **Pseudocode**: ``x[rd] = i64(round(f[rs1]))``

    **Invalid values**: illegal instruction when ``XLEN`` is 32.

    **Exception raised**: ``NV``, ``NX``

- **FCVT.LU.H**: Convert Half-precision to Unsigned 64-bit Integer (RV64 only)

    **Format**: fcvt.lu.h rd, rs1

    **Description**: converts the half-precision value in ``rs1`` to an unsigned 64-bit integer in ``x[rd]``,
    rounded per the ``rm`` field.

    **Pseudocode**: ``x[rd] = u64(round(f[rs1]))``

    **Invalid values**: illegal instruction when ``XLEN`` is 32.

    **Exception raised**: ``NV``, ``NX``

- **FCVT.H.L**: Convert Signed 64-bit Integer to Half-precision (RV64 only)

    **Format**: fcvt.h.l rd, rs1

    **Description**: converts the signed 64-bit integer in ``x[rs1]`` to a half-precision value, rounded per the
    ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(h(i64(x[rs1]))))``

    **Invalid values**: illegal instruction when ``XLEN`` is 32.

    **Exception raised**: ``OF``, ``NX``

- **FCVT.H.LU**: Convert Unsigned 64-bit Integer to Half-precision (RV64 only)

    **Format**: fcvt.h.lu rd, rs1

    **Description**: converts the unsigned 64-bit integer in ``x[rs1]`` to a half-precision value, rounded per the
    ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(h(u64(x[rs1]))))``

    **Invalid values**: illegal instruction when ``XLEN`` is 32.

    **Exception raised**: ``OF``, ``NX``

- **FCVT.S.H**: Convert Half-precision to Single-precision

    **Format**: fcvt.s.h rd, rs1

    **Description**: widening conversion of the half-precision value in ``rs1`` to a single-precision value. The
    conversion is exact.

    **Pseudocode**: ``f[rd] = NaN-box(s(f[rs1]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``

- **FCVT.H.S**: Convert Single-precision to Half-precision

    **Format**: fcvt.h.s rd, rs1

    **Description**: narrowing conversion of the single-precision value in ``rs1`` to a half-precision value,
    rounded per the ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(h(f[rs1])))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FCVT.D.H**: Convert Half-precision to Double-precision (requires RVD)

    **Format**: fcvt.d.h rd, rs1

    **Description**: widening conversion of the half-precision value in ``rs1`` to a double-precision value. The
    conversion is exact.

    **Pseudocode**: ``f[rd] = NaN-box(d(f[rs1]))``

    **Invalid values**: illegal instruction when ``RVD`` is not present.

    **Exception raised**: ``NV``

- **FCVT.H.D**: Convert Double-precision to Half-precision (requires RVD)

    **Format**: fcvt.h.d rd, rs1

    **Description**: narrowing conversion of the double-precision value in ``rs1`` to a half-precision value,
    rounded per the ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(h(f[rs1])))``

    **Invalid values**: illegal instruction when ``RVD`` is not present.

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FMV.X.H**: Move Half-precision to Integer Register

    **Format**: fmv.x.h rd, rs1

    **Description**: moves the half-precision bit pattern in ``rs1`` to the low 16 bits of ``x[rd]``, filling the
    upper ``XLEN``-16 bits with copies of bit 15 (the half-precision sign bit). Performs no arithmetic; NaN
    payloads are preserved.

    **Pseudocode**: ``x[rd] = sext(f[rs1][15:0])``

    **Invalid values**: NONE

    **Exception raised**: NONE

- **FMV.H.X**: Move Integer Register to Half-precision

    **Format**: fmv.h.x rd, rs1

    **Description**: moves the low 16 bits of ``x[rs1]`` to ``rd``, NaN-boxing the result. Performs no arithmetic;
    NaN payloads are preserved.

    **Pseudocode**: ``f[rd] = NaN-box(x[rs1][15:0])``

    **Invalid values**: NONE

    **Exception raised**: NONE

Half-Precision Floating-Point Compare Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **FEQ.H**: Half-precision Quiet Equal

    **Format**: feq.h rd, rs1, rs2

    **Description**: writes 1 to ``x[rd]`` if the half-precision values in ``rs1`` and ``rs2`` are equal, else 0.
    This is a quiet comparison: it does not raise an exception for a quiet NaN operand.

    **Pseudocode**: ``x[rd] = (f[rs1] == f[rs2]) ? 1 : 0``

    **Invalid values**: NONE

    **Exception raised**: ``NV``

- **FLT.H**: Half-precision Less Than

    **Format**: flt.h rd, rs1, rs2

    **Description**: writes 1 to ``x[rd]`` if ``rs1`` is less than ``rs2``, else 0. This is a signaling
    comparison.

    **Pseudocode**: ``x[rd] = (f[rs1] < f[rs2]) ? 1 : 0``

    **Invalid values**: NONE

    **Exception raised**: ``NV``

- **FLE.H**: Half-precision Less Than or Equal

    **Format**: fle.h rd, rs1, rs2

    **Description**: writes 1 to ``x[rd]`` if ``rs1`` is less than or equal to ``rs2``, else 0. This is a
    signaling comparison.

    **Pseudocode**: ``x[rd] = (f[rs1] <= f[rs2]) ? 1 : 0``

    **Invalid values**: NONE

    **Exception raised**: ``NV``

Half-Precision Floating-Point Classify Instruction
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **FCLASS.H**: Half-precision Classify

    **Format**: fclass.h rd, rs1

    **Description**: writes a 10-bit mask to ``x[rd]`` classifying the half-precision value in ``rs1`` (negative
    infinity, negative normal, negative subnormal, negative zero, positive zero, positive subnormal, positive
    normal, positive infinity, signaling NaN, or quiet NaN). Exactly one bit is set.

    **Pseudocode**: ``x[rd] = classify(f[rs1])``

    **Invalid values**: NONE

    **Exception raised**: NONE

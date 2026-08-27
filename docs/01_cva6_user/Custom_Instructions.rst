..
   Copyright (c) 2026 OpenHW Foundation
   Copyright (c) 2023 Thales DIS design services SAS

   SPDX-License-Identifier: Apache-2.0 WITH SHL-2.1

.. Level 1
   =======

   Level 2
   -------

   Level 3
   ~~~~~~~

   Level 4
   ^^^^^^^

.. _cva6_custom_instructions:

*This chapter is applicable to all configurations.*

Custom RISC-V instructions
==========================
CVA6 optionally implements the scalar ``Xcvbf16`` and ``Xcvf8`` extensions. These are CVA6-specific,
non-standard extensions: they are not ratified RISC-V extensions and have no official RISC-V encoding.
The ratified ``Zfh`` and ``Zfbfmin`` extensions that ``Xcvbf16`` builds on are documented separately in
:ref:`Zfh <cva6_riscv_instructions_RVZfh>` and :ref:`Zfbfmin <cva6_riscv_instructions_RVZfbfmin>`, alongside the other
standard RISC-V extensions implemented by CVA6.

``Xcvbf16`` supplements the standard Zfbfmin conversions with BFloat16 arithmetic and conversions that do not
have ratified scalar RISC-V encodings. It is controlled by the ``XCVBF16`` configuration flag and requires
``RVZFBFMIN`` and full Zfh support. ``Xcvf8`` provides scalar FP8 arithmetic, conversions, moves, and
load/store instructions; unlike Xcvbf16 there is no ratified extension it can lean on, so it defines its own
moves and loads/stores rather than reusing standard instructions. It is controlled by the ``XCVF8``
configuration flag and requires ``RVF``.

Configuration dependencies are checked at elaboration:

* ``XCVBF16`` requires ``RVZFBFMIN``.
* ``XCVF8`` requires ``RVF``.

*Applicability of this chapter to configurations:*

``Xcvbf16`` and ``Xcvf8`` are not enabled in any of the currently verified CVA6 configurations documented in
this manual (CV32A60AX, CV32A60X, CV64A6_MMU).

BFloat16-Precision Floating-Point Instructions
----------------------------------------------
BF16 operations with one or two source operands use the R-type format with Custom0 major opcode (``0x0b``) and 
an OP-FP-like layout. The ``fmt`` field is set to ``10`` to select the BF16 precision, and the ``funct5`` field specifies 
the particular BF16 operation.

.. TODO: Add layout diagrams for the R-type instruction

Arithmetic
~~~~~~~~~~

- **CV.FADD.BF16**: BF16 Add

    **Format**: cv.fadd.bf16 rd, rs1, rs2

    **Description**: adds the BF16 values in ``rs1`` and ``rs2``, and writes the rounded sum to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(bf16(f[rs1]) + bf16(f[rs2])))``

    **Encoding**: match ``0x0400000b``, mask ``0xfe00007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FSUB.BF16**: BF16 Subtract

    **Format**: cv.fsub.bf16 rd, rs1, rs2

    **Description**: subtracts the BF16 value in ``rs2`` from ``rs1``, and writes the rounded result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(bf16(f[rs1]) - bf16(f[rs2])))``

    **Encoding**: match ``0x0c00000b``, mask ``0xfe00007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FMUL.BF16**: BF16 Multiply

    **Format**: cv.fmul.bf16 rd, rs1, rs2

    **Description**: multiplies the BF16 values in ``rs1`` and ``rs2``, and writes the rounded product to
    ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(bf16(f[rs1]) * bf16(f[rs2])))``

    **Encoding**: match ``0x1400000b``, mask ``0xfe00007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FDIV.BF16**: BF16 Divide

    **Format**: cv.fdiv.bf16 rd, rs1, rs2

    **Description**: divides the BF16 value in ``rs1`` by ``rs2``, and writes the rounded quotient to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(bf16(f[rs1]) / bf16(f[rs2])))``

    **Encoding**: match ``0x1c00000b``, mask ``0xfe00007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``DZ``, ``OF``, ``UF``, ``NX``

- **CV.FSQRT.BF16**: BF16 Square Root

    **Format**: cv.fsqrt.bf16 rd, rs1

    **Description**: computes the rounded square root of the BF16 value in ``rs1`` and writes it to ``rd``.
    ``rs2`` must be zero.

    **Pseudocode**: ``f[rd] = NaN-box(round(sqrt(bf16(f[rs1]))))``

    **Encoding**: match ``0x5c00000b``, mask ``0xfff0007f``

    **Invalid values**: a non-zero ``rs2`` field is an illegal instruction.

    **Exception raised**: ``NV``, ``NX``

Sign Injection
~~~~~~~~~~~~~~

- **CV.FSGNJ.BF16**: BF16 Sign-Inject

    **Format**: cv.fsgnj.bf16 rd, rs1, rs2

    **Description**: writes the value of ``rs1`` to ``rd``, but with the sign bit replaced by the sign bit of
    ``rs2``.

    **Pseudocode**: ``f[rd] = NaN-box({f[rs2][15], f[rs1][14:0]})``

    **Encoding**: match ``0x2400000b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` (``funct3``) values under this ``funct5`` are reserved.

    **Exception raised**: NONE

- **CV.FSGNJN.BF16**: BF16 Sign-Inject Negate

    **Format**: cv.fsgnjn.bf16 rd, rs1, rs2

    **Description**: writes the value of ``rs1`` to ``rd``, but with the sign bit replaced by the negation of
    the sign bit of ``rs2``.

    **Pseudocode**: ``f[rd] = NaN-box({~f[rs2][15], f[rs1][14:0]})``

    **Encoding**: match ``0x2400100b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: NONE

- **CV.FSGNJX.BF16**: BF16 Sign-Inject XOR

    **Format**: cv.fsgnjx.bf16 rd, rs1, rs2

    **Description**: writes the value of ``rs1`` to ``rd``, but with the sign bit replaced by the XOR of the
    sign bits of ``rs1`` and ``rs2``.

    **Pseudocode**: ``f[rd] = NaN-box({f[rs1][15] ^ f[rs2][15], f[rs1][14:0]})``

    **Encoding**: match ``0x2400200b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: NONE

Minimum and Maximum
~~~~~~~~~~~~~~~~~~~~

- **CV.FMIN.BF16**: BF16 Minimum

    **Format**: cv.fmin.bf16 rd, rs1, rs2

    **Description**: writes the smaller of the BF16 values in ``rs1`` and ``rs2`` to ``rd``. If one operand is
    a quiet NaN and the other is not NaN, the non-NaN operand is returned without raising an exception.

    **Pseudocode**: ``f[rd] = NaN-box(min(bf16(f[rs1]), bf16(f[rs2])))``

    **Encoding**: match ``0x2c00000b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

- **CV.FMAX.BF16**: BF16 Maximum

    **Format**: cv.fmax.bf16 rd, rs1, rs2

    **Description**: writes the larger of the BF16 values in ``rs1`` and ``rs2`` to ``rd``, with the same
    NaN-handling rules as CV.FMIN.BF16.

    **Pseudocode**: ``f[rd] = NaN-box(max(bf16(f[rs1]), bf16(f[rs2])))``

    **Encoding**: match ``0x2c00100b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

Comparisons
~~~~~~~~~~~

- **CV.FLE.BF16**: BF16 Less Than or Equal

    **Format**: cv.fle.bf16 rd, rs1, rs2

    **Description**: writes 1 to ``x[rd]`` if ``rs1`` is less than or equal to ``rs2``, else 0. This is a
    signaling comparison.

    **Pseudocode**: ``x[rd] = (bf16(f[rs1]) <= bf16(f[rs2])) ? 1 : 0``

    **Encoding**: match ``0xa400000b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

- **CV.FLT.BF16**: BF16 Less Than

    **Format**: cv.flt.bf16 rd, rs1, rs2

    **Description**: writes 1 to ``x[rd]`` if ``rs1`` is less than ``rs2``, else 0. This is a signaling
    comparison.

    **Pseudocode**: ``x[rd] = (bf16(f[rs1]) < bf16(f[rs2])) ? 1 : 0``

    **Encoding**: match ``0xa400100b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

- **CV.FEQ.BF16**: BF16 Quiet Equal

    **Format**: cv.feq.bf16 rd, rs1, rs2

    **Description**: writes 1 to ``x[rd]`` if the BF16 values in ``rs1`` and ``rs2`` are equal, else 0. This is
    a quiet comparison: it does not raise an exception for a quiet NaN operand.

    **Pseudocode**: ``x[rd] = (bf16(f[rs1]) == bf16(f[rs2])) ? 1 : 0``

    **Encoding**: match ``0xa400200b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

Classification
~~~~~~~~~~~~~~

- **CV.FCLASS.BF16**: BF16 Classify

    **Format**: cv.fclass.bf16 rd, rs1

    **Description**: writes a 10-bit mask to ``x[rd]`` classifying the BF16 value in ``rs1``. Exactly one bit
    is set.

    **Pseudocode**: ``x[rd] = classify(bf16(f[rs1]))``

    **Encoding**: match ``0xe400100b``, mask ``0xfff0707f``

    **Invalid values**: this ``funct5`` (``11100``) with ``rm=000`` is reserved for BF16 -- unlike Xcvf8, there
    is no ``cv.fmv.x.bf16``; the standard Zfh ``fmv.x.h`` is used instead.

    **Exception raised**: NONE

Integer Conversions
~~~~~~~~~~~~~~~~~~~~

- **CV.FCVT.BF16.W** / **CV.FCVT.BF16.WU** / **CV.FCVT.BF16.L** / **CV.FCVT.BF16.LU**: Integer to BF16

    **Format**: cv.fcvt.bf16.w rd, rs1 (``.wu``, ``.l``, ``.lu`` variants select the source integer type via
    ``rs2``)

    **Description**: converts the 32-bit (``w``/``wu``) or 64-bit (``l``/``lu``, RV64 only) signed or unsigned
    integer in ``x[rs1]`` to a BF16 value, rounded per the ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(bf16(int(x[rs1]))))``

    **Encoding**: match ``0xc400000b`` / ``0xc410000b`` / ``0xc420000b`` / ``0xc430000b``, mask ``0xfff0007f``

    **Invalid values**: ``rs2`` values other than ``[00000``:``00011]`` are reserved; ``.l``/``.lu`` (bit
    21 set) are illegal instructions when ``XLEN`` is 32.

    **Exception raised**: ``OF``, ``NX``

- **CV.FCVT.W.BF16** / **CV.FCVT.WU.BF16** / **CV.FCVT.L.BF16** / **CV.FCVT.LU.BF16**: BF16 to Integer

    **Format**: cv.fcvt.w.bf16 rd, rs1 (``.wu``, ``.l``, ``.lu`` variants select the destination integer type
    via ``rs2``)

    **Description**: converts the BF16 value in ``rs1`` to a 32-bit (``w``/``wu``) or 64-bit (``l``/``lu``,
    RV64 only) signed or unsigned integer in ``x[rd]``, rounded per the ``rm`` field.

    **Pseudocode**: ``x[rd] = int(round(bf16(f[rs1])))``

    **Encoding**: match ``0xd400000b`` / ``0xd410000b`` / ``0xd420000b`` / ``0xd430000b``, mask ``0xfff0007f``

    **Invalid values**: ``rs2`` values other than ``[00000``:``00011]`` are reserved; ``.l``/``.lu`` are illegal
    instructions when ``XLEN`` is 32.

    **Exception raised**: ``NV``, ``NX``

Floating-Point Conversions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The conversion direction is selected by ``funct5`` rather than by argument order: ``funct5=01000`` converts a
non-BF16 format to BF16, and ``funct5=01001`` converts BF16 to a non-BF16 format. For both, ``fmt`` selects the
non-BF16 partner format (``01``=D, ``10``=H, ``11``=Byte); ``fmt=00`` is reserved, since FP32 conversions are
covered by the standard Zfbfmin ``fcvt.bf16.s``/``fcvt.s.bf16`` instructions instead.

- **CV.FCVT.BF16.D**: Double-precision to BF16 (requires RVD)

    **Format**: cv.fcvt.bf16.d rd, rs1

    **Description**: narrowing conversion of the double-precision value in ``rs1`` to a BF16 value, rounded per
    the ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(bf16(f[rs1])))``

    **Encoding**: match ``0x4200000b``, mask ``0xfff0007f``

    **Invalid values**: illegal instruction when ``RVD`` is not present.

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FCVT.BF16.H**: Half-precision to BF16 (requires Zfh)

    **Format**: cv.fcvt.bf16.h rd, rs1

    **Description**: converts the IEEE half-precision value in ``rs1`` to a BF16 value, rounded per the ``rm``
    field.

    **Pseudocode**: ``f[rd] = NaN-box(round(bf16(f[rs1])))``

    **Encoding**: match ``0x4400000b``, mask ``0xfff0007f``

    **Invalid values**: illegal instruction when ``ZfhEnabled`` is false.

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FCVT.BF16.XF8**: FP8 to BF16 (requires Xcvf8)

    **Format**: cv.fcvt.bf16.xf8 rd, rs1

    **Description**: widening conversion of the FP8 value in ``rs1`` to a BF16 value. The conversion is exact,
    since BF16 has both a wider mantissa and a wider exponent range than FP8.

    **Pseudocode**: ``f[rd] = NaN-box(bf16(f[rs1][7:0]))``

    **Encoding**: match ``0x4600000b``, mask ``0xfff0007f``

    **Invalid values**: illegal instruction when ``XCVF8`` is not enabled.

    **Exception raised**: ``NV``

- **CV.FCVT.D.BF16**: BF16 to Double-precision (requires RVD)

    **Format**: cv.fcvt.d.bf16 rd, rs1

    **Description**: widening conversion of the BF16 value in ``rs1`` to a double-precision value. The
    conversion is exact.

    **Pseudocode**: ``f[rd] = NaN-box(d(bf16(f[rs1])))``

    **Encoding**: match ``0x4a00000b``, mask ``0xfff0007f``

    **Invalid values**: illegal instruction when ``RVD`` is not present.

    **Exception raised**: ``NV``

- **CV.FCVT.H.BF16**: BF16 to Half-precision (requires Zfh)

    **Format**: cv.fcvt.h.bf16 rd, rs1

    **Description**: converts the BF16 value in ``rs1`` to an IEEE half-precision value, rounded per the ``rm``
    field. BF16's wider exponent range means this conversion can overflow or underflow relative to ``H``.

    **Pseudocode**: ``f[rd] = NaN-box(round(h(bf16(f[rs1]))))``

    **Encoding**: match ``0x4c00000b``, mask ``0xfff0007f``

    **Invalid values**: illegal instruction when ``ZfhEnabled`` is false.

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FCVT.XF8.BF16**: BF16 to FP8 (requires Xcvf8)

    **Format**: cv.fcvt.xf8.bf16 rd, rs1

    **Description**: narrowing conversion of the BF16 value in ``rs1`` to an FP8 value, rounded per the ``rm``
    field.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(bf16(f[rs1]))))``

    **Encoding**: match ``0x4e00000b``, mask ``0xfff0007f``

    **Invalid values**: illegal instruction when ``XCVF8`` is not enabled.

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

Fused-Multiply-Add Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The four-register FMA family uses Custom1 (opcode ``0x2b``) with the R4 layout
``rs3 | funct2 | rs2 | rs1 | rm | rd | opcode``. Bits ``[26:25]`` (``funct2``) select the operation and the mask
is ``0x0600007f``. Legal rounding modes are ``rm=000`` through ``100``, or dynamic ``rm=111``.

- **CV.FMADD.BF16**: BF16 Fused Multiply-Add

    **Format**: cv.fmadd.bf16 rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, adds ``rs3``, and writes the rounded result to ``rd``, with
    only a single rounding step applied to the final sum.

    **Pseudocode**: ``f[rd] = NaN-box(round(bf16(f[rs1]) * bf16(f[rs2]) + bf16(f[rs3])))``

    **Encoding**: match ``0x0000002b``, mask ``0x0600007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FMSUB.BF16**: BF16 Fused Multiply-Subtract

    **Format**: cv.fmsub.bf16 rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, subtracts ``rs3``, and writes the rounded result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(bf16(f[rs1]) * bf16(f[rs2]) - bf16(f[rs3])))``

    **Encoding**: match ``0x0200002b``, mask ``0x0600007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FNMSUB.BF16**: BF16 Negate Fused Multiply-Subtract

    **Format**: cv.fnmsub.bf16 rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, negates the product, adds ``rs3``, and writes the rounded
    result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(-(bf16(f[rs1]) * bf16(f[rs2])) + bf16(f[rs3])))``

    **Encoding**: match ``0x0400002b``, mask ``0x0600007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FNMADD.BF16**: BF16 Negate Fused Multiply-Add

    **Format**: cv.fnmadd.bf16 rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, negates the product, subtracts ``rs3``, and writes the
    rounded result to ``rd``.

    **Pseudocode**: f[rd] = NaN-box(round(-(bf16(f[rs1]) * bf16(f[rs2])) - bf16(f[rs3])))

    **Encoding**: match ``0x0600002b``, mask ``0x0600007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

8-bit Precision Floating-Point Instructions
-------------------------------------------
FP8 operations with one or two source operands share the Custom0 major opcode (``0x0b``) with Xcvbf16. 
For arithmetic, conversion, and move operations, the ``fmt`` field is set to ``11`` to select the FP8 precision, and the ``funct5`` field specifies the particular FP8 operation.

Loads and stores are structurally I-type/S-type rather than R-type, and are
identified by ``funct3`` values (``110`` for load, ``101`` for store) that are always-reserved rounding-mode
encodings for the R-type arithmetic instructions, so the two never collide.

Load and Store
~~~~~~~~~~~~~~

- **CV.FLB**: Load Byte (FP8)

    **Format**: cv.flb rd, imm(rs1)

    **Description**: loads an 8-bit value from memory into the low 8 bits of ``rd``, and NaN-boxes the result.
    The effective address is obtained by adding register ``rs1`` to the sign-extended 12-bit offset.

    **Pseudocode**: ``f[rd] = NaN-box(M[x[rs1] + sext(imm[11:0])][7:0])``

    **Encoding**: match ``0x0000600b``, mask ``0x0000707f`` (I-type, ``funct3``/width ``=110``)

    **Invalid values**:

    **Exception raised**: NONE

- **CV.FSB**: Store Byte (FP8)

    **Format**: cv.fsb rs2, imm(rs1)

    **Description**: stores the low 8 bits of ``rs2`` to memory. The effective address is obtained by adding
    register ``rs1`` to the sign-extended 12-bit offset.

    **Pseudocode**: ``M[x[rs1] + sext(imm[11:0])][7:0] = f[rs2][7:0]``

    **Encoding**: match ``0x0000500b``, mask ``0x0000707f`` (S-type, ``funct3``/width ``=101``)

    **Invalid values**:

    **Exception raised**: NONE

Arithmetic
~~~~~~~~~~

- **CV.FADD.B**: FP8 Add

    **Format**: cv.fadd.b rd, rs1, rs2

    **Description**: adds the FP8 values in ``rs1`` and ``rs2``, and writes the rounded sum to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(f[rs1]) + fp8(f[rs2])))``

    **Encoding**: match ``0x0600000b``, mask ``0xfe00007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FSUB.B**: FP8 Subtract

    **Format**: cv.fsub.b rd, rs1, rs2

    **Description**: subtracts the FP8 value in ``rs2`` from ``rs1``, and writes the rounded result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(f[rs1]) - fp8(f[rs2])))``

    **Encoding**: match ``0x0e00000b``, mask ``0xfe00007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FMUL.B**: FP8 Multiply

    **Format**: cv.fmul.b rd, rs1, rs2

    **Description**: multiplies the FP8 values in ``rs1`` and ``rs2``, and writes the rounded product to
    ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(f[rs1]) * fp8(f[rs2])))``

    **Encoding**: match ``0x1600000b``, mask ``0xfe00007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FDIV.B**: FP8 Divide

    **Format**: cv.fdiv.b rd, rs1, rs2

    **Description**: divides the FP8 value in ``rs1`` by ``rs2``, and writes the rounded quotient to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(f[rs1]) / fp8(f[rs2])))``

    **Encoding**: match ``0x1e00000b``, mask ``0xfe00007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``DZ``, ``OF``, ``UF``, ``NX``

- **CV.FSQRT.B**: FP8 Square Root

    **Format**: cv.fsqrt.b rd, rs1

    **Description**: computes the rounded square root of the FP8 value in ``rs1`` and writes it to ``rd``.
    ``rs2`` must be zero.

    **Pseudocode**: ``f[rd] = NaN-box(round(sqrt(fp8(f[rs1]))))``

    **Encoding**: match ``0x5e00000b``, mask ``0xfff0007f``

    **Invalid values**: a non-zero ``rs2`` field is an illegal instruction.

    **Exception raised**: ``NV``, ``NX``

Sign Injection
~~~~~~~~~~~~~~

- **CV.FSGNJ.B**: FP8 Sign-Inject

    **Format**: cv.fsgnj.b rd, rs1, rs2

    **Description**: writes the value of ``rs1`` to ``rd``, but with the sign bit replaced by the sign bit of
    ``rs2``.

    **Pseudocode**: ``f[rd] = NaN-box({f[rs2][7], f[rs1][6:0]})``

    **Encoding**: match ``0x2600000b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: NONE

- **CV.FSGNJN.B**: FP8 Sign-Inject Negate

    **Format**: cv.fsgnjn.b rd, rs1, rs2

    **Description**: writes the value of ``rs1`` to ``rd``, but with the sign bit replaced by the negation of
    the sign bit of ``rs2``.

    **Pseudocode**: ``f[rd] = NaN-box({~f[rs2][7], f[rs1][6:0]})``

    **Encoding**: match ``0x2600100b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: NONE

- **CV.FSGNJX.B**: FP8 Sign-Inject XOR

    **Format**: cv.fsgnjx.b rd, rs1, rs2

    **Description**: writes the value of ``rs1`` to ``rd``, but with the sign bit replaced by the XOR of the
    sign bits of ``rs1`` and ``rs2``.

    **Pseudocode**: ``f[rd] = NaN-box({f[rs1][7] ^ f[rs2][7], f[rs1][6:0]})``

    **Encoding**: match ``0x2600200b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: NONE

Minimum and Maximum
~~~~~~~~~~~~~~~~~~~~

- **CV.FMIN.B**: FP8 Minimum

    **Format**: cv.fmin.b rd, rs1, rs2

    **Description**: writes the smaller of the FP8 values in ``rs1`` and ``rs2`` to ``rd``. If one operand is a
    quiet NaN and the other is not NaN, the non-NaN operand is returned without raising an exception.

    **Pseudocode**: ``f[rd] = NaN-box(min(fp8(f[rs1]), fp8(f[rs2])))``

    **Encoding**: match ``0x2e00000b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

- **CV.FMAX.B**: FP8 Maximum

    **Format**: cv.fmax.b rd, rs1, rs2

    **Description**: writes the larger of the FP8 values in ``rs1`` and ``rs2`` to ``rd``, with the same
    NaN-handling rules as CV.FMIN.B.

    **Pseudocode**: ``f[rd] = NaN-box(max(fp8(f[rs1]), fp8(f[rs2])))``

    **Encoding**: match ``0x2e00100b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

Comparisons
~~~~~~~~~~~

- **CV.FLE.B**: FP8 Less Than or Equal

    **Format**: cv.fle.b rd, rs1, rs2

    **Description**: writes 1 to ``x[rd]`` if ``rs1`` is less than or equal to ``rs2``, else 0. This is a
    signaling comparison.

    **Pseudocode**: ``x[rd] = (fp8(f[rs1]) <= fp8(f[rs2])) ? 1 : 0``

    **Encoding**: match ``0xa600000b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

- **CV.FLT.B**: FP8 Less Than

    **Format**: cv.flt.b rd, rs1, rs2

    **Description**: writes 1 to ``x[rd]`` if ``rs1`` is less than ``rs2``, else 0. This is a signaling
    comparison.

    **Pseudocode**: ``x[rd] = (fp8(f[rs1]) < fp8(f[rs2])) ? 1 : 0``

    **Encoding**: match ``0xa600100b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

- **CV.FEQ.B**: FP8 Quiet Equal

    **Format**: cv.feq.b rd, rs1, rs2

    **Description**: writes 1 to ``x[rd]`` if the FP8 values in ``rs1`` and ``rs2`` are equal, else 0. This is
    a quiet comparison.

    **Pseudocode**: ``x[rd] = (fp8(f[rs1]) == fp8(f[rs2])) ? 1 : 0``

    **Encoding**: match ``0xa600200b``, mask ``0xfe00707f``

    **Invalid values**: other ``rm`` values under this ``funct5`` are reserved.

    **Exception raised**: ``NV``

Moves and Classification
~~~~~~~~~~~~~~~~~~~~~~~~~

- **CV.FMV.B.X**: Move Integer Register to FP8

    **Format**: cv.fmv.b.x rd, rs1

    **Description**: moves the low 8 bits of ``x[rs1]`` to ``rd``, NaN-boxing the result. Performs no
    arithmetic; NaN payloads are preserved.

    **Pseudocode**: ``f[rd] = NaN-box(x[rs1][7:0])``

    **Encoding**: match ``0xf600000b``, mask ``0xfff0707f`` (``funct5=11110``, ``rm=000``)

    **Invalid values**: illegal instruction when ``XCVF8`` is not enabled, or when ``fmt`` is not ``11``.

    **Exception raised**: NONE

- **CV.FMV.X.B**: Move FP8 to Integer Register

    **Format**: cv.fmv.x.b rd, rs1

    **Description**: moves the FP8 bit pattern in ``rs1`` to the low 8 bits of ``x[rd]``, filling the upper
    ``XLEN``-8 bits with copies of bit 7 (the FP8 sign bit).

    **Pseudocode**: ``x[rd] = sext(f[rs1][7:0])``

    **Encoding**: match ``0xe600000b``, mask ``0xfff0707f`` (``funct5=11100``, ``rm=000``)

    **Invalid values**:

    **Exception raised**: NONE

- **CV.FCLASS.B**: FP8 Classify

    **Format**: cv.fclass.b rd, rs1

    **Description**: writes a 10-bit mask to ``x[rd]`` classifying the FP8 value in ``rs1``. Exactly one bit is
    set.

    **Pseudocode**: ``x[rd] = classify(fp8(f[rs1]))``

    **Encoding**: match ``0xe600100b``, mask ``0xfff0707f`` (``funct5=11100``, ``rm=001``)

    **Invalid values**: NONE

    **Exception raised**: NONE

Integer Conversions
~~~~~~~~~~~~~~~~~~~~

- **CV.FCVT.B.W** / **CV.FCVT.B.WU** / **CV.FCVT.B.L** / **CV.FCVT.B.LU**: Integer to FP8

    **Format**: cv.fcvt.b.w rd, rs1 (``.wu``, ``.l``, ``.lu`` variants select the source integer type via
    ``rs2``)

    **Description**: converts the 32-bit (``w``/``wu``) or 64-bit (``l``/``lu``, RV64 only) signed or unsigned
    integer in ``x[rs1]`` to an FP8 value, rounded per the ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(int(x[rs1]))))``

    **Encoding**: match ``0xd600000b`` / ``0xd610000b`` / ``0xd620000b`` / ``0xd630000b``, mask ``0xfff0007f``

    **Invalid values**: ``rs2`` values other than ``00000``-``00011`` are reserved; ``.l``/``.lu`` are illegal
    instructions when ``XLEN`` is 32.

    **Exception raised**: ``OF``, ``NX``

- **CV.FCVT.W.B** / **CV.FCVT.WU.B** / **CV.FCVT.L.B** / **CV.FCVT.LU.B**: FP8 to Integer

    **Format**: cv.fcvt.w.b rd, rs1 (``.wu``, ``.l``, ``.lu`` variants select the destination integer type via
    ``rs2``)

    **Description**: converts the FP8 value in ``rs1`` to a 32-bit (``w``/``wu``) or 64-bit (``l``/``lu``,
    RV64 only) signed or unsigned integer in ``x[rd]``, rounded per the ``rm`` field.

    **Pseudocode**: ``x[rd] = int(round(fp8(f[rs1])))``

    **Encoding**: match ``0xc600000b`` / ``0xc610000b`` / ``0xc620000b`` / ``0xc630000b``, mask ``0xfff0007f``

    **Invalid values**: ``rs2`` values other than ``00000``-``00011`` are reserved; ``.l``/``.lu`` are illegal
    instructions when ``XLEN`` is 32.

    **Exception raised**: ``NV``, ``NX``

Floating-Point Conversions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The conversion direction is selected by ``funct5`` rather than by argument order: ``funct5=01100`` converts a
non-Byte format to Byte (FP8), and ``funct5=01101`` converts Byte to a non-Byte format. For both, ``fmt``
selects the non-Byte partner format (``00``=S, ``01``=D, ``10``=H); ``fmt=11`` is reserved.
``cv.fcvt.xf8.bf16`` and ``cv.fcvt.bf16.xf8`` are handled by Xcvbf16 and are not duplicated here.

- **CV.FCVT.B.S**: Single-precision to FP8 (requires RVF)

    **Format**: cv.fcvt.b.s rd, rs1

    **Description**: narrowing conversion of the single-precision value in ``rs1`` to an FP8 value, rounded per
    the ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(f[rs1])))``

    **Encoding**: match ``0x6000000b``, mask ``0xfff0007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FCVT.B.D**: Double-precision to FP8 (requires RVD)

    **Format**: cv.fcvt.b.d rd, rs1

    **Description**: narrowing conversion of the double-precision value in ``rs1`` to an FP8 value, rounded per
    the ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(f[rs1])))``

    **Encoding**: match ``0x6200000b``, mask ``0xfff0007f``

    **Invalid values**:

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FCVT.B.H**: Half-precision to FP8 (requires Zfh)

    **Format**: cv.fcvt.b.h rd, rs1

    **Description**: narrowing conversion of the half-precision value in ``rs1`` to an FP8 value, rounded per
    the ``rm`` field.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(f[rs1])))``

    **Encoding**: match ``0x6400000b``, mask ``0xfff0007f``

    **Invalid values**:

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FCVT.S.B**: FP8 to Single-precision (requires RVF)

    **Format**: cv.fcvt.s.b rd, rs1

    **Description**: widening conversion of the FP8 value in ``rs1`` to a single-precision value. The
    conversion is exact.

    **Pseudocode**: ``f[rd] = NaN-box(s(fp8(f[rs1])))``

    **Encoding**: match ``0x6800000b``, mask ``0xfff0007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``

- **CV.FCVT.D.B**: FP8 to Double-precision (requires RVD)

    **Format**: cv.fcvt.d.b rd, rs1

    **Description**: widening conversion of the FP8 value in ``rs1`` to a double-precision value. The
    conversion is exact.

    **Pseudocode**: ``f[rd] = NaN-box(d(fp8(f[rs1])))``

    **Encoding**: match ``0x6a00000b``, mask ``0xfff0007f``

    **Invalid values**:

    **Exception raised**: ``NV``

- **CV.FCVT.H.B**: FP8 to Half-precision (requires Zfh)

    **Format**: cv.fcvt.h.b rd, rs1

    **Description**: widening conversion of the FP8 value in ``rs1`` to an IEEE half-precision value. The
    conversion is exact, since ``H`` has both a wider mantissa and a wider exponent range than FP8.

    **Pseudocode**: ``f[rd] = NaN-box(h(fp8(f[rs1])))``

    **Encoding**: match ``0x6c00000b``, mask ``0xfff0007f``

    **Invalid values**:

    **Exception raised**: ``NV``

Fused-Multiply-Add Instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The four-register FMA family uses Custom2 (opcode ``0x5b``) with the same R4 layout and rounding-mode
legality its Xcvbf16 counterpart.

- **CV.FMADD.B**: FP8 Fused Multiply-Add

    **Format**: cv.fmadd.b rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, adds ``rs3``, and writes the rounded result to ``rd``, with
    only a single rounding step applied to the final sum.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(f[rs1]) * fp8(f[rs2]) + fp8(f[rs3])))``

    **Encoding**: match ``0x0000005b``, mask ``0x0600007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FMSUB.B**: FP8 Fused Multiply-Subtract

    **Format**: cv.fmsub.b rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, subtracts ``rs3``, and writes the rounded result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(fp8(f[rs1]) * fp8(f[rs2]) - fp8(f[rs3])))``

    **Encoding**: match ``0x0200005b``, mask ``0x0600007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FNMSUB.B**: FP8 Negate Fused Multiply-Subtract

    **Format**: cv.fnmsub.b rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, negates the product, adds ``rs3``, and writes the rounded
    result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(-(fp8(f[rs1]) * fp8(f[rs2])) + fp8(f[rs3])))``

    **Encoding**: match ``0x0400005b``, mask ``0x0600007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **CV.FNMADD.B**: FP8 Negate Fused Multiply-Add

    **Format**: cv.fnmadd.b rd, rs1, rs2, rs3

    **Description**: multiplies ``rs1`` by ``rs2``, negates the product, subtracts ``rs3``, and writes the
    rounded result to ``rd``.

    **Pseudocode**: ``f[rd] = NaN-box(round(-(fp8(f[rs1]) * fp8(f[rs2])) - fp8(f[rs3])))``

    **Encoding**: match ``0x0600005b``, mask ``0x0600007f``

    **Invalid values**: NONE

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

Custom opcode ownership and CV-X-IF interaction
------------------------------------------------

When ``XCVBF16`` or ``XCVF8`` is enabled, Custom0 is owned by the core decoder and unavailable for CV-X-IF
offload; Custom1 is likewise owned when ``XCVBF16`` is enabled, and Custom2 when ``XCVF8`` is enabled. Custom3
remains available. Reserved or disabled sub-encodings inside an owned opcode decode as illegal instructions
like any other undefined encoding; CVA6 does not add opcode-scoped exclusion logic on top of the existing
generic illegal-instruction-to-CV-X-IF offload path, so such an illegal instruction is still offered to CV-X-IF
when ``CvxifEn`` is set, exactly as any other illegal instruction would be. Vector extensions and their
existing encodings are unchanged.

Legacy XF16ALT transition
--------------------------

``XF16ALT`` retains the previous OP-FP ``fmt=10``/``rm``-overloaded datapath for one transition release. It is
deprecated and will be removed after software migrates to Zfbfmin plus Xcvbf16. The new path carries explicit
source and destination format metadata from decode to the FPU and does not infer BF16 from ``rm[2]``.

Legacy XF8 transition
-----------------------

``XF8`` retains the previous OP-FP ``fmt=11`` byte-format datapath, including ``cv.flb``/``cv.fsb`` at the
standard LOAD-FP/STORE-FP ``funct3=000`` encoding, for one transition release. It is deprecated and will be
removed after software migrates to Xcvf8.

The team is looking for contributors to implement the ``fence.t`` instruction that ensures that the execution time of subsequent instructions is unrelated with predecessor instructions.

The user or integrator can also use the CV-X-IF coprocessor interface to implement their own extensions, without modifying the core.

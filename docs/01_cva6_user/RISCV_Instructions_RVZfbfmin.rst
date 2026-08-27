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

.. _cva6_riscv_instructions_RVZfbfmin:

*Applicability of this chapter to configurations:*

This extension is supported by CVA6 (``RVZFBFMIN`` configuration parameter) but is not enabled in any of the
currently verified CVA6 configurations documented in this manual (CV32A60AX, CV32A60X, CV64A6_MMU).

RVZfbfmin: Scalar BFloat16 Conversion Instructions
---------------------------------------------------

Zfbfmin provides the minimal set of instructions needed to enable scalar support of the BFloat16 (BF16) format.
It enables BF16 as an interchange format by providing conversion between BF16 values and single-precision (FP32)
values. Zfbfmin depends on the ``F`` extension, and on the ``FLH``, ``FSH``, ``FMV.X.H``, and ``FMV.H.X``
instructions defined by the ``Zfh`` extension, which CVA6 requires to be fully enabled (``RVZFH``) whenever
``RVZFBFMIN`` is set.

Zfbfmin only supports conversion between BF16 and FP32. Native BF16 arithmetic (add, subtract, multiply, divide,
square-root, FMA, and conversions to/from formats other than FP32) is provided by the CVA6-specific ``Xcvbf16``
extension, described in :ref:`Custom RISC-V instructions <cva6_custom_instructions>`.

.. csv-table::
   :widths: auto
   :align: left
   :header: "Mnemonic", "Instruction"

   "fcvt.bf16.s", "Convert FP32 to BF16"
   "fcvt.s.bf16", "Convert BF16 to FP32"

BF16 values are held in the low 16 bits of a floating-point register and NaN-boxed in the same way as
half-precision (``H``) values: the upper ``FLEN``-16 bits are set to 1 for a valid BF16 value. The RISC-V
canonical NaN for BF16 is ``0x7fc0`` (the most significant 16 bits of the FP32 canonical NaN ``0x7fc00000``).

Conversion Instructions
~~~~~~~~~~~~~~~~~~~~~~~~

- **FCVT.BF16.S**: Convert FP32 to BF16

    **Format**: fcvt.bf16.s rd, rs1

    **Description**: narrowing conversion of the FP32 value in ``rs1`` to a BF16 value, rounded per the ``rm``
    field or the dynamic rounding mode in ``frm``. The result is written to the low 16 bits of ``rd`` and
    NaN-boxed. This instruction uses a dedicated encoding (``fmt=10``, ``rs2=01000``) distinct from the generic
    Zfh floating-to-floating conversion encoding.

    **Pseudocode**: ``f[rd] = NaN-box(bf16(round(f[rs1])))``

    **Invalid values**: reserved static rounding modes (``rm`` = 101 or 110), and a reserved dynamic rounding
    mode selected via ``frm``, are illegal instructions.

    **Exception raised**: ``NV``, ``OF``, ``UF``, ``NX``

- **FCVT.S.BF16**: Convert BF16 to FP32

    **Format**: fcvt.s.bf16 rd, rs1

    **Description**: widening conversion of the BF16 value in the low 16 bits of ``rs1`` to an FP32 value. The
    conversion is exact, since FP32 has both a wider mantissa and the same exponent range as BF16. This
    instruction reuses the existing floating-to-floating "convert to S" decode slot (``fmt=00``, ``rs2=00110``),
    gated by ``RVZFBFMIN``.

    **Pseudocode**: ``f[rd] = NaN-box(fp32(f[rs1][15:0]))``

    **Invalid values**: NONE

    **Exception raised**: ``NV``

.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Context Management
==================

Context management in heterogeneous-ISA systems handles ISA-specific
register formats.

Context Structure
-----------------

Each ISA has a different context format:

.. code-block:: text

   RISC-V Context:
   ┌──────────────────────────────────────────────────┐
   │ PC (64-bit)                                      │
   │ x1-x31 (31 × 64-bit)                            │
   │ F0-F31 (32 × 64-bit, if FP enabled)             │
   │ CSR subset (sstatus, etc.)                      │
   └──────────────────────────────────────────────────┘

   ARM64 Context:
   ┌──────────────────────────────────────────────────┐
   │ PC (64-bit)                                      │
   │ SP (64-bit)                                      │
   │ X0-X30 (31 × 64-bit)                            │
   │ V0-V31 (32 × 128-bit, SIMD/FP)                  │
   │ PSTATE subset                                    │
   └──────────────────────────────────────────────────┘

   x86-64 Context:
   ┌──────────────────────────────────────────────────┐
   │ RIP (64-bit)                                     │
   │ RSP (64-bit)                                     │
   │ RFLAGS (64-bit)                                  │
   │ RAX-R15 (16 × 64-bit)                           │
   │ XMM0-XMM15 (16 × 128-bit, SSE)                  │
   └──────────────────────────────────────────────────┘

Context Save on Domain Transition
---------------------------------

Hardware saves context in ISA-specific format:

.. code-block:: text

   On ascend/descend:

   Step 1: Determine current ISA
   ─────────────────────────────
   current_isa = current_dcb.isa_tag

   Step 2: Save in ISA-specific format
   ───────────────────────────────────
   switch current_isa:
       case RISCV:
           save_riscv_context(current_dcb)
       case ARM64:
           save_arm64_context(current_dcb)
       case X86_64:
           save_x86_64_context(current_dcb)

Context Restore
---------------

Context restore uses target ISA format:

.. code-block:: text

   On return:

   Step 1: Determine target ISA
   ────────────────────────────
   target_isa = child_dcb.isa_tag

   Step 2: Restore in ISA-specific format
   ──────────────────────────────────────
   switch target_isa:
       case RISCV:
           restore_riscv_context(child_dcb)
       case ARM64:
           restore_arm64_context(child_dcb)
       case X86_64:
           restore_x86_64_context(child_dcb)

Cross-ISA Calling Convention
----------------------------

Cross-ISA calls require argument mapping:

.. code-block:: text

   RISC-V → ARM64 argument mapping:

   ┌──────────────────────────────────────────────────┐
   │ RISC-V  │ ARM64  │ Purpose                       │
   ├──────────────────────────────────────────────────┤
   │ a0 (x10)│ X0     │ Argument 0 / Return value     │
   │ a1 (x11)│ X1     │ Argument 1                    │
   │ a2 (x12)│ X2     │ Argument 2                    │
   │ a3 (x13)│ X3     │ Argument 3                    │
   │ a4 (x14)│ X4     │ Argument 4                    │
   │ a5 (x15)│ X5     │ Argument 5                    │
   │ a6 (x16)│ X6     │ Argument 6                    │
   │ a7 (x17)│ X7     │ Argument 7                    │
   └──────────────────────────────────────────────────┘

   Hardware performs this mapping automatically on
   cross-ISA domain transitions.

Context Size
------------

Context size varies by ISA:

.. list-table:: Context Sizes
   :widths: 25 25 50
   :header-rows: 1

   * - ISA
     - Base Size
     - With Extensions
   * - RISC-V RV64
     - 264 bytes
     - +256 bytes (FP), + variable (V)
   * - ARM64
     - 528 bytes
     - +512 bytes (SVE)
   * - x86-64
     - 296 bytes
     - + variable (AVX-512)

.. code-block:: text

   DCB.ctrlblock_size must accommodate:
   - Context for domain's ISA
   - ISA-specific extensions
   - Implementation-defined extensions

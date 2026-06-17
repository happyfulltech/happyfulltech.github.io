.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

ISA Tag
=======

RPA supports heterogeneous-ISA systems through ISA tagging of domains.

ISA Tag Field
-------------

Each DCB contains an ISA identifier:

.. code-block:: text

   DCB Field: isa_tag
   ──────────────────
   Identifies the instruction set architecture

   Standard ISA Tags:
   ┌──────────────────────────────────────────────────┐
   │ 0x0000 : RISC-V (RV64)                           │
   │ 0x0001 : ARM64 (AArch64)                         │
   │ 0x0002 : x86-64                                  │
   │ 0x0003 : MIPS64                                  │
   │ 0x0004 : PowerPC64                               │
   │ 0xFF00-0xFFFF : Implementation-defined          │
   └──────────────────────────────────────────────────┘

ISA Switch on Domain Transition
-------------------------------

Domain transitions may involve ISA switch:

.. code-block:: text

   descend R0 (child_dcb):

   Step 1: Read child ISA tag
   ──────────────────────────
   child_isa = memory[R0 + isa_tag_offset]

   Step 2: ISA validation
   ──────────────────────
   if child_isa not in supported_isas:
       fault(UNSUPPORTED_ISA)

   Step 3: ISA switch
   ──────────────────
   current_isa = child_isa
   update_decoder(current_isa)

   Step 4: Context format switch
   ─────────────────────────────
   // Register file format may differ
   // Saved context uses target ISA format

ISA Boundary Rules
------------------

ISA transitions follow specific rules:

.. list-table:: ISA Transition Rules
   :widths: 30 70
   :header-rows: 1

   * - Transition
     - Behavior
   * - Same ISA
     - Normal domain switch
   * - Different ISA
     - ISA switch + context reformat
   * - Unsupported ISA
     - Fault (UNSUPPORTED_ISA)

.. code-block:: text

   Example: RISC-V to ARM64 transition:

   Domain 0 (RISC-V):
     └─ Domain 1 (ARM64):
        └─ Domain 2 (ARM64):

   descend from Domain 0 to Domain 1:
   - ISA switch: RISC-V → ARM64
   - Register reformat: x0-x31 → X0-X30, SP, PC

   descend from Domain 1 to Domain 2:
   - Same ISA (ARM64)
   - Normal domain switch

Binary Translation Integration
------------------------------

ISA tags enable binary translation:

.. code-block:: text

   Legacy application on modern ISA:

   ┌──────────────────────────────────────────────────┐
   │ Domain 0: ARM64 OS                               │
   │   └─ Domain 1: x86-32 Application                │
   │       (isa_tag = x86-32)                         │
   │       (Binary translator in DCB)                 │
   └──────────────────────────────────────────────────┘

   Hardware can:
   - Recognize ISA mismatch
   - Invoke binary translator
   - Execute translated code
   - Maintain x86-32 semantics

Implementation Requirements
---------------------------

Heterogeneous-ISA support requires:

.. code-block:: text

   Hardware:
   - Multiple instruction decoders
   - ISA-aware context save/restore
   - ISA tag validation

   Software:
   - ISA-aware compilers
   - Cross-ISA calling conventions
   - Binary translation support (optional)

.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

Hardware-Software Interface
===========================

This chapter describes the interface between RPA hardware and software.

DCB Memory Layout
-----------------

The Domain Control Block is the primary hardware-software interface:

.. code-block:: text

   DCB Memory Layout:
   ┌───────────┬────────────────────────────────────────┐
   │ Offset    │ Field                                  │
   ├───────────┼────────────────────────────────────────┤
   │ 0x00      │ ctrlblock_size (32-bit)                │
   │ 0x04      │ domain_id (32-bit)                     │
   │ 0x08      │ parent_block (64-bit)                  │
   │ 0x10      │ child_block (64-bit)                   │
   │ 0x18      │ trap_vector (64-bit)                   │
   │ 0x20      │ interrupt_vector (64-bit)              │
   │ 0x28      │ page_table_root (64-bit)               │
   │ 0x30      │ translation_mode (32-bit)              │
   │ 0x34      │ isa_tag (16-bit)                       │
   │ 0x36      │ security_group (8-bit)                 │
   │ 0x37      │ encryption_mode (8-bit)                │
   │ 0x38      │ saved_pc (64-bit)                      │
   │ 0x40      │ saved_sp (64-bit)                      │
   │ 0x48      │ saved_psr (64-bit)                     │
   │ 0x50+     │ ISA-specific registers                 │
   │ ...       │ ...                                    │
   │ variable  │ ipa_regions[]                          │
   └───────────┴────────────────────────────────────────┘

Instruction Encoding
--------------------

RPA primitives use standard instruction encoding:

.. code-block:: text

   descend encoding:
   ┌──────────────────────────────────────────────────┐
   │ [31:26] opcode = 0x3F (CUSTOM)                   │
   │ [25:20] func = 0x00 (DESCEND)                    │
   │ [19:15] rs1 = DCB address register               │
   │ [14:0]  unused                                  │
   └──────────────────────────────────────────────────┘

   ascend encoding:
   ┌──────────────────────────────────────────────────┐
   │ [31:26] opcode = 0x3F (CUSTOM)                   │
   │ [25:20] func = 0x01 (ASCEND)                     │
   │ [19:15] rs1 = service type register              │
   │ [14:0]  unused                                  │
   └──────────────────────────────────────────────────┘

   return encoding:
   ┌──────────────────────────────────────────────────┐
   │ [31:26] opcode = 0x3F (CUSTOM)                   │
   │ [25:20] func = 0x02 (RETURN)                     │
   │ [19:0]  unused                                  │
   └──────────────────────────────────────────────────┘

Control Registers
-----------------

RPA defines system control registers:

.. code-block:: text

   Control Registers:
   ┌──────────────────────────────────────────────────┐
   │ Address   │ Name          │ Description          │
   ├──────────────────────────────────────────────────┤
   │ 0x800     │ CURRENT_DCB   │ Current domain's DCB │
   │ 0x804     │ DOMAIN_ID     │ Current domain ID    │
   │ 0x808     │ PARENT_DCB    │ Parent's DCB         │
   │ 0x80C     │ CHILD_DCB     │ Child's DCB          │
   │ 0x810     │ TRAP_STATUS   │ Trap status register │
   │ 0x814     │ FAULT_CODE    │ Fault code register  │
   └──────────────────────────────────────────────────┘

   Access:
   - Read-only: DOMAIN_ID, CURRENT_DCB
   - Read-write: TRAP_STATUS (by handler)
   - Write-only: FAULT_CODE (hardware writes)

Software Conventions
--------------------

Software should follow these conventions:

.. code-block:: text

   DCB Allocation:
   - DCB must be 32-byte aligned
   - DCB size must be multiple of 32 bytes
   - DCB should be in memory accessible to parent

   Register Usage:
   - R0 (or X0/a0): Argument/return value
   - R1 (or X1/a1): Second argument
   - PC: Saved on domain transition
   - SP: Saved on domain transition

   Trap Handler Convention:
   ┌──────────────────────────────────────────────────┐
   │ Entry:                                           │
   │   R0 = trap_type                                 │
   │   R1 = faulting_address                          │
   │   SP = handler stack (configured)                │
   │                                                  │
   │ Exit:                                            │
   │   return instruction                             │
   │   R0 = return value                              │
   └──────────────────────────────────────────────────┘

Error Codes
-----------

Hardware reports errors via fault codes:

.. list-table:: Fault Codes
   :widths: 20 80
   :header-rows: 1

   * - Code
     - Description
   * - 0x00
     - No error
   * - 0x01
     - DCB alignment error
   * - 0x02
     - DCB size error
   * - 0x03
     - No child domain
   * - 0x04
     - Child not suspended
   * - 0x10
     - IPA boundary violation
   * - 0x11
     - Permission denied
   * - 0x12
     - Security group violation
   * - 0x20
     - Unhandled trap
   * - 0x21
     - Unhandled interrupt
   * - 0x30
     - Unsupported ISA

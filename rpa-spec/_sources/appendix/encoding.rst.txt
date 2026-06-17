.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

DCB Field Encoding
==================

This appendix provides detailed encoding for DCB fields.

Field Offsets
-------------

.. code-block:: text

   DCB Field Offsets (bytes):
   ┌───────────┬────────────────────────────────────────┐
   │ Offset    │ Field                                  │
   ├───────────┼────────────────────────────────────────┤
   │ 0x00      │ ctrlblock_size                         │
   │ 0x04      │ domain_id                              │
   │ 0x08      │ parent_block                           │
   │ 0x10      │ child_block                            │
   │ 0x18      │ trap_vector                            │
   │ 0x20      │ interrupt_vector                       │
   │ 0x28      │ interrupt_mask                         │
   │ 0x30      │ page_table_root                        │
   │ 0x38      │ translation_control                    │
   │ 0x40      │ saved_pc                               │
   │ 0x48      │ saved_sp                               │
   │ 0x50      │ saved_psr                              │
   │ 0x58      │ saved_lr                               │
   │ 0x60      │ isa_tag                                │
   │ 0x62      │ security_group                         │
   │ 0x63      │ encryption_mode                        │
   │ 0x64      │ encryption_key_id                      │
   │ 0x68      │ attestation_key_id                     │
   │ 0x70      │ ipa_region_count                       │
   │ 0x78      │ ipa_regions[0].base                    │
   │ 0x80      │ ipa_regions[0].limit                   │
   │ 0x88      │ ipa_regions[0].attributes              │
   │ ...       │ (subsequent regions)                   │
   └───────────┴────────────────────────────────────────┘

Field Descriptions
------------------

ctrlblock_size (Offset 0x00)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   Size: 32 bits
   Description: Size of DCB in bytes
   Constraints:
   - Must be ≥ 8 (minimum header size)
   - Must be multiple of 32 (alignment)
   Usage: Hardware validates on descend

domain_id (Offset 0x04)
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   Size: 32 bits
   Description: Unique domain identifier
   Constraints:
   - Assigned by hardware on descend
   - Read-only after assignment
   Usage: Identifies domain in TLB, attestation

parent_block (Offset 0x08)
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   Size: 64 bits
   Description: Physical address of parent's DCB
   Constraints:
   - Set by parent on domain creation
   - Must be valid DCB address
   Usage: Hardware traverses to parent on ascend

child_block (Offset 0x10)
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   Size: 64 bits
   Description: Physical address of child's DCB
   Constraints:
   - 0 if no active child
   - Set by parent on descend
   Usage: Hardware uses on return

trap_vector (Offset 0x18)
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   Size: 64 bits
   Description: Entry point for trap handler
   Constraints:
   - 0 = no handler (propagate to parent)
   - Must be in accessible IPA region
   Usage: Hardware jumps here on trap

IPA Region Attributes
---------------------

.. code-block:: text

   ipa_regions[].attributes encoding:
   ┌──────────────────────────────────────────────────┐
   │ Bits  │ Field                                   │
   ├──────────────────────────────────────────────────┤
   │ 0     │ Readable (R)                            │
   │ 1     │ Writable (W)                            │
   │ 2     │ Executable (X)                          │
   │ 3     │ Device memory (D)                       │
   │ 4     │ Cacheable (C)
   │ 5     │ Bufferable (B)                          │
   │ 6-7   │ Reserved                                │
   │ 8-15  │ Memory type extension                   │
   └──────────────────────────────────────────────────┘

Translation Control
-------------------

.. code-block:: text

   translation_control encoding:
   ┌──────────────────────────────────────────────────┐
   │ Bits  │ Field                                   │
   ├──────────────────────────────────────────────────┤
   │ 0     │ Translation enable                      │
   │       │   0 = INHERIT (use parent's tables)     │
   │       │   1 = OWN (use own page tables)         │
   │ 1-2   │ Page table format                       │
   │       │   0 = RISC-V Sv39                       │
   │       │   1 = RISC-V Sv48                       │
   │       │   2 = ARM64                             │
   │       │   3 = x86-64                            │
   │ 3-7   │ Reserved                                │
   └──────────────────────────────────────────────────┘

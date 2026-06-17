.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

DCB Fields
==========

RPA Spec Fields
---------------

.. list-table:: RPA Spec Field Encoding
   :widths: 10 20 15 55
   :header-rows: 1

   * - Offset
     - Field
     - Set By
     - Description
   * - 0x00
     - ctrlblock_size
     - Parent
     - Control block size in words (min 8)
   * - 0x04
     - domain_id
     - System
     - Unique domain identifier
   * - 0x08
     - trap_vector
     - Child
     - Trap handler entry (0 = propagate to parent)
   * - 0x0C
     - interrupt_ctrl
     - System
     - Interrupt controller handle
   * - 0x10
     - ipa_regions
     - Parent
     - IPA boundary table address
   * - 0x14
     - pagetable
     - Child
     - Page table base (0 = INHERIT mode)
   * - 0x18
     - child_block
     - Parent
     - Child DCB address for return
   * - 0x1C
     - security_group
     - System
     - Security group handle
   * - 0x20
     - isa_tag
     - Parent
     - ISA identifier

Field Semantics
---------------

ctrlblock_size
~~~~~~~~~~~~~~

Size of the entire DCB in words. Used by hardware to validate DCB during descend.

- Minimum: 8 words (RPA Spec Fields only)
- Alignment: 8 words
- Must be set by parent before descend

domain_id
~~~~~~~~~

System-assigned unique identifier for the domain.

- Assigned by hardware on first descend
- Used for DMA access control
- Used for security group binding

trap_vector
~~~~~~~~~~~

Entry point for trap handling.

- Set by child domain after initialization
- Value 0: Traps propagate to parent
- Non-zero: Hardware jumps to this address on trap

interrupt_ctrl
~~~~~~~~~~~~~~

Handle for the domain's interrupt controller.

- Allocated by system via sysop instruction
- Determines which interrupts this domain handles

ipa_regions
~~~~~~~~~~~

Address of IPA boundary table.

- Set by parent, read-only for child
- Defines accessible address space
- Hardware checks all translations against these bounds

pagetable
~~~~~~~~~

Page table base address for VA→IPA translation.

- Set by child domain
- Value 0: INHERIT mode (share parent address space)
- Non-zero: Independent address translation

child_block
~~~~~~~~~~~

Address of child domain's DCB.

- Maintained by parent
- Used by return instruction to resume child
- Cleared when child is destroyed

security_group
~~~~~~~~~~~~~~

Security group handle for memory isolation.

- Allocated by system
- Enables memory encryption
- Separates management rights from access rights

isa_tag
~~~~~~~

ISA identifier for heterogeneous-ISA support.

.. list-table:: ISA Tag Values
   :widths: 20 80
   :header-rows: 1

   * - Value
     - ISA
   * - 0x0000
     - INHERIT (same as parent)
   * - 0x0001
     - ARM/AArch32
   * - 0x0002
     - RISC-V
   * - 0x0003
     - x86-64
   * - 0x0004
     - IBM Z

Access Permissions
------------------

.. list-table:: DCB Field Access
   :widths: 25 25 25 25
   :header-rows: 1

   * - Field
     - Parent
     - Child
     - Hardware
   * - ctrlblock_size
     - R/W
     - -
     - R
   * - domain_id
     - -
     - -
     - R/W
   * - trap_vector
     - -
     - R/W
     - R
   * - ipa_regions
     - R/W
     - R
     - R
   * - pagetable
     - -
     - R/W
     - R
   * - child_block
     - R/W
     - -
     - R
   * - security_group
     - -
     - -
     - R/W
   * - isa_tag
     - R/W
     - -
     - R

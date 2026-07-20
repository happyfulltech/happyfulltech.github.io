.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Custom Extensions
=================

RPA is designed to be extensible. This appendix describes how to add
custom extensions.

Extension Points
----------------

RPA provides several extension points:

.. list-table:: Extension Points
   :widths: 30 70
   :header-rows: 1

   * - Extension Point
     - Description
   * - DCB fields
     - Add implementation-defined fields
   * - ISA tags
     - Support custom ISAs
   * - Trap types
     - Define custom trap reasons
   * - Memory attributes
     - Custom memory types

Custom DCB Fields
-----------------

.. code-block:: text

   Standard DCB fields end at offset defined by ctrlblock_size.
   Implementation-defined fields follow:

   ┌──────────────────────────────────────────────────┐
   │ Standard DCB fields (0x00 - 0x6F)                │
   ├──────────────────────────────────────────────────┤
   │ IPA regions (variable)                           │
   ├──────────────────────────────────────────────────┤
   │ Implementation-defined fields                    │
   │   0xN+00: custom_field_0                         │
   │   0xN+08: custom_field_1                         │
   │   ...                                            │
   └──────────────────────────────────────────────────┘

   Guidelines:
   - ctrlblock_size must include custom fields
   - Custom fields must be documented
   - Software must check ctrlblock_size before accessing

Custom ISA Tags
---------------

.. code-block:: text

   ISA tag allocation:
   ┌──────────────────────────────────────────────────┐
   │ 0x0000 - 0x00FF : Standard ISAs (RISC-V, ARM, x86)│
   │ 0x0100 - 0xFEFF : Reserved for future standard    │
   │ 0xFF00 - 0xFFFF : Implementation-defined          │
   └──────────────────────────────────────────────────┘

   Custom ISA implementation:
   1. Define ISA tag value (0xFF00+)
   2. Implement instruction decoder
   3. Define context save/restore format
   4. Document in implementation manual

Custom Trap Types
-----------------

.. code-block:: text

   Trap type allocation:
   ┌──────────────────────────────────────────────────┐
   │ 0x00 - 0x7F : Standard traps                     │
   │ 0x80 - 0xFF : Implementation-defined             │
   └──────────────────────────────────────────────────┘

   Standard trap types:
   ┌──────────────────────────────────────────────────┐
   │ 0x00 : System call                               │
   │ 0x01 : Page fault                                │
   │ 0x02 : Privilege violation                       │
   │ 0x03 : IPA boundary violation                    │
   │ 0x04 : Alignment fault                           │
   │ 0x05 : Security group violation                  │
   └──────────────────────────────────────────────────┘

   Custom trap handling:
   1. Hardware detects custom condition
   2. Hardware sets trap_type = 0x80+
   3. Standard trap propagation applies
   4. Handler interprets custom trap_type

Custom Memory Attributes
------------------------

.. code-block:: text

   Memory attribute extension:
   ┌──────────────────────────────────────────────────┐
   │ Standard attributes (bits 0-7)                    │
   │   R, W, X, D, C, B                               │
   ├──────────────────────────────────────────────────┤
   │ Extension attributes (bits 8-15)                  │
   │   Implementation-defined                          │
   └──────────────────────────────────────────────────┘

   Example custom attributes:
   ┌──────────────────────────────────────────────────┐
   │ Bit 8  : Scratchpad memory                       │
   │ Bit 9  : Tightly-coupled memory                  │
   │ Bit 10 : Non-coherent                            │
   └──────────────────────────────────────────────────┘

Extension Discovery
-------------------

Software can discover available extensions:

.. code-block:: text

   Extension Discovery Register:
   ┌──────────────────────────────────────────────────┐
   │ Address: 0xF00                                   │
   │ Bits 0-7  : Supported standard features          │
   │ Bits 8-15 : Supported custom extensions          │
   │ Bits 16-23: Extension version                    │
   │ Bits 24-31: Implementation ID                    │
   └──────────────────────────────────────────────────┘

   Software should:
   1. Read extension discovery register
   2. Check required features are present
   3. Adapt behavior based on available extensions

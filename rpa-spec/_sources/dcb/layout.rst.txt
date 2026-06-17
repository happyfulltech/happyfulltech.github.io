.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

DCB Memory Layout
=================

The Domain Control Block resides in ordinary memory, aligned to 8-word boundaries.

Three-Section Layout
--------------------

.. code-block:: text

   ┌─────────────────────────────────────────┐
   │ RPA Spec Fields (8 words, fixed)        │  Offset 0x00-0x1F
   │ Architecture-defined, cross-platform    │
   ├─────────────────────────────────────────┤
   │ RPA Impdef Fields (variable)            │  Offset 0x20+
   │ Implementation-defined extensions       │
   ├─────────────────────────────────────────┤
   │ ISA Context Fields (variable)           │
   │ ISA-specific context save area          │
   └─────────────────────────────────────────┘

Alignment Requirements
----------------------

- DCB address must be 8-word aligned (32-byte boundary for 32-bit systems)
- Minimum size: 8 words (RPA Spec Fields only)
- ctrlblock_size field specifies total size in words

Memory Allocation
-----------------

The parent domain allocates DCB memory:

.. code-block:: text

   // Allocate aligned memory
   dcb_addr = aligned_alloc(32, dcb_size * 4);

   // Initialize to zero
   memset(dcb_addr, 0, dcb_size * 4);

   // Set size field
   dcb_addr->ctrlblock_size = dcb_size;

The child domain cannot access its own DCB through direct memory access.

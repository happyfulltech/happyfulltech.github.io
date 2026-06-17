.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

IPA Boundary
============

IPA (Intermediate Physical Address) boundaries define the memory regions
accessible to each domain.

IPA Region Definition
---------------------

Each DCB contains a list of permitted IPA regions:

.. code-block:: text

   DCB Field: ipa_regions[]
   ────────────────────────
   Array of IPA region descriptors

   Region Descriptor:
   ┌──────────────────────────────────────────────────┐
   │ base_ipa   : Starting IPA (64-bit)               │
   ├──────────────────────────────────────────────────┤
   │ limit_ipa  : Ending IPA (64-bit, inclusive)      │
   ├──────────────────────────────────────────────────┤
   │ attributes : Access permissions                  │
   │   - Readable (R)                                 │
   │   - Writable (W)                                 │
   │   - Executable (X)                               │
   │   - Device memory (D)                            │
   └──────────────────────────────────────────────────┘

Boundary Check
--------------

Hardware checks IPA boundaries on every memory access:

.. code-block:: text

   check_ipa_boundary(ipa, access_type):

   for region in current_dcb.ipa_regions:
       if ipa >= region.base_ipa and ipa <= region.limit_ipa:
           // Check access permissions
           if access_type == READ and not region.R:
               fault(PERMISSION_DENIED)
           if access_type == WRITE and not region.W:
               fault(PERMISSION_DENIED)
           if access_type == EXECUTE and not region.X:
               fault(PERMISSION_DENIED)
           return ALLOWED

   // IPA not in any permitted region
   fault(IPA_BOUNDARY_VIOLATION)

Violation Handling
------------------

IPA boundary violations trigger traps:

.. code-block:: text

   On IPA_BOUNDARY_VIOLATION:
   1. Hardware saves context to current DCB
   2. Trap propagates to parent domain
   3. Parent handler decides:
      - Map new region (extend access)
      - Terminate domain (security violation)
      - Emulate access (device I/O)

Example: Application Memory Isolation
-------------------------------------

.. code-block:: text

   Application DCB:
   ┌──────────────────────────────────────────────────┐
   │ ipa_regions[0]:                                  │
   │   base  = 0x00010000                            │
   │   limit = 0x0001FFFF  (64KB code)               │
   │   attr  = R-X                                    │
   ├──────────────────────────────────────────────────┤
   │ ipa_regions[1]:                                  │
   │   base  = 0x00020000                            │
   │   limit = 0x0002FFFF  (64KB data)               │
   │   attr  = RW-                                    │
   └──────────────────────────────────────────────────┘

   Attempt to access IPA 0x00030000:
   → IPA_BOUNDARY_VIOLATION
   → Trap to parent (OS)
   → OS may extend region or terminate application

Nested Virtualization Example
-----------------------------

.. code-block:: text

   Guest VM IPA regions (set by hypervisor):

   Guest DCB:
   ┌──────────────────────────────────────────────────┐
   │ ipa_regions[0]:                                  │
   │   base  = 0x40000000                            │
   │   limit = 0x4FFFFFFF  (256MB guest physical)    │
   │   attr  = RWX                                    │
   └──────────────────────────────────────────────────┘

   Hypervisor translates guest IPA to host PA:
   - Guest IPA 0x40000000 → Host PA 0x80000000
   - Guest IPA 0x4FFFFFFF → Host PA 0x8FFFFFFF

   Guest cannot access outside its assigned region.

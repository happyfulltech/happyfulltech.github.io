.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

INHERIT Mode
============

INHERIT mode enables efficient address space sharing between parent
and child domains.

Concept
-------

INHERIT mode allows a child domain to use the parent's page tables:

.. code-block:: text

   DCB Field: translation_mode
   ───────────────────────────
   INHERIT  (0): Use parent's page tables
   OWN      (1): Use own page tables

   When INHERIT is set:
   - VA → IPA translation uses parent's page tables
   - No separate page table for child
   - Address space fully shared with parent

Performance Benefit
-------------------

INHERIT mode eliminates double translation overhead:

.. code-block:: text

   Traditional (no INHERIT):
   VA → IPA (child page table) → PA (parent page table)
   Cost: 2 page table walks

   INHERIT mode:
   VA → PA (parent page table directly)
   Cost: 1 page table walk

   System call overhead comparison:
   ┌─────────────────────────────────────────┐
   │ Traditional: 100-500 cycles             │
   │ INHERIT:     1-10 cycles                │
   └─────────────────────────────────────────┘

Use Cases
---------

System Calls
~~~~~~~~~~~~

.. code-block:: text

   Application domain (INHERIT mode):
   - Shares OS page table
   - ascend to OS is nearly free
   - No TLB flush on domain switch

   Timeline:
   1. Application executes in child domain
   2. ascend for system call
   3. OS handler runs with same address space
   4. return to application
   - No page table switch
   - No TLB invalidation

Nested Virtualization
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   Guest hypervisor with INHERIT:
   - Can share root hypervisor's address space
   - VM exit handling is fast
   - Nested virtualization overhead reduced

   Domain 0: Root hypervisor
     └─ Domain 1: Guest hypervisor (INHERIT from Domain 0)
        └─ Domain 2: Guest OS
           └─ Domain 3: Guest application

Security Considerations
-----------------------

INHERIT mode does NOT bypass IPA boundaries:

.. code-block:: text

   Even with INHERIT mode:
   1. Child's IPA regions still enforced
   2. Child cannot access memory outside its regions
   3. Parent can restrict child's view

   Example:
   ┌──────────────────────────────────────────────────┐
   │ Parent (OS) has full memory access               │
   │ Child (Application) with INHERIT:                │
   │   - Uses parent's page tables                    │
   │   - But IPA regions limit to:                    │
   │     - Code: 0x10000-0x1FFFF                      │
   │     - Data: 0x20000-0x2FFFF                      │
   │   - Cannot access other memory                   │
   └──────────────────────────────────────────────────┘

Mode Switching
--------------

Translation mode can be changed by parent:

.. code-block:: text

   // Parent switches child from OWN to INHERIT
   child_dcb.translation_mode = INHERIT
   child_dcb.page_table_root = 0  // Not used

   // Must flush TLB
   TLB.flush_domain(child_domain.id)

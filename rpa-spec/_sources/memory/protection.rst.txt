.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Memory Protection
=================

RPA provides multi-layer memory protection through IPA boundaries,
page permissions, and domain isolation.

Protection Layers
-----------------

.. code-block:: text

   Layer 1: Page Table Permissions
   ───────────────────────────────
   Per-page R/W/X attributes

   Layer 2: IPA Boundary Checks
   ────────────────────────────
   Region-level access control

   Layer 3: Domain Isolation
   ────────────────────────
   Complete address space separation

Permission Propagation
----------------------

Child domains cannot exceed parent permissions:

.. code-block:: text

   Permission intersection rule:

   child_effective_perm = child_page_perm ∩ parent_page_perm

   Example:
   ┌──────────────────────────────────────────────────┐
   │ Parent page: RW-                                 │
   │ Child page: RWX                                  │
   │ Effective:   RW-  (execute denied)               │
   └──────────────────────────────────────────────────┘

   Hardware enforces this automatically:
   - Child page table may claim RWX
   - Parent page table limits to RW-
   - Actual access is intersection

Write-XOR-Execute (W^X)
-----------------------

RPA supports W^X enforcement:

.. code-block:: text

   W^X rule: A page cannot be both writable and executable

   Implementation:
   if page.W and page.X:
       fault(W_X_VIOLATION)

   This can be enforced at:
   - Page table level (hardware check)
   - IPA region level (region attributes)
   - Domain level (security policy)

Memory Types
------------

RPA distinguishes memory types:

.. list-table:: Memory Types
   :widths: 20 80
   :header-rows: 1

   * - Type
     - Description
   * - Normal
     - Cacheable, reorderable memory
   * - Device
     - Non-cacheable, side-effect memory
   * - Strongly-ordered
     - Non-cacheable, non-reorderable

.. code-block:: text

   Memory type in IPA region:
   region.attributes.device = 1  // Device memory

   Hardware behavior:
   - Device memory: No speculative reads
   - Device memory: No write combining
   - Device memory: Strict ordering

Protection Example: Secure Enclave
----------------------------------

.. code-block:: text

   Secure enclave memory layout:

   ┌──────────────────────────────────────────────────┐
   │ Domain: Secure Enclave                           │
   │                                                  │
   │ IPA Region 0 (Code):                             │
   │   base  = 0x00000000                            │
   │   limit = 0x0000FFFF                            │
   │   attr  = R-X (read-only, executable)           │
   │                                                  │
   │ IPA Region 1 (Data):                             │
   │   base  = 0x00010000                            │
   │   limit = 0x0001FFFF                            │
   │   attr  = RW- (read-write, no execute)          │
   │                                                  │
   │ IPA Region 2 (Shared buffer):                    │
   │   base  = 0x00020000                            │
   │   limit = 0x0002FFFF                            │
   │   attr  = RW- (shared with parent)              │
   │                                                  │
   │ No other regions → Cannot access outside memory │
   └──────────────────────────────────────────────────┘

   Parent (OS) can:
   - Read/write shared buffer
   - Cannot read enclave private memory
   - If security groups enabled, OS cannot even access

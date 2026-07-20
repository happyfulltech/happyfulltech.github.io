.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

descend Primitive
=================

The descend primitive enters a child domain for execution.

Instruction Format
------------------

.. code-block:: text

   descend <dcb_address>

   dcb_address: Register or immediate containing DCB address

Pre-conditions
--------------

Parent domain must:

1. Allocate and initialize DCB
2. Set ctrlblock_size
3. Configure ipa_regions
4. Set entry point in saved_lr field

Execution Steps
---------------

.. code-block:: text

   descend R0 (R0 = child_dcb_address):

   Step 1: DCB Validation
   ────────────────────────
   size = memory[R0 + 0x00]

   if R0 % 32 != 0:
       fault(DCB_ALIGNMENT_ERROR)

   if size < 8:
       fault(DCB_SIZE_ERROR)

   Step 2: Domain ID Assignment
   ─────────────────────────────
   domain_id = next_domain_id++
   memory[R0 + 0x04] = domain_id

   Step 3: Parent Context Save
   ───────────────────────────
   parent_dcb.saved_pc = PC + 4
   parent_dcb.saved_sp = SP
   parent_dcb.saved_psr = PSR

   Step 4: Link Child
   ──────────────────
   parent_dcb.child_block = R0

   Step 5: Domain Switch
   ─────────────────────
   current_domain = child_domain

   Step 6: Load Child Entry
   ─────────────────────────
   PC = memory[R0 + saved_lr_offset]

Error Conditions
----------------

.. list-table:: descend Errors
   :widths: 25 75
   :header-rows: 1

   * - Error
     - Cause
   * - DCB_ALIGNMENT
     - DCB address not 8-word aligned
   * - DCB_SIZE
     - ctrlblock_size < 8
   * - IPA_FAULT
     - DCB address not in accessible memory
   * - PERMISSION
     - Insufficient privilege for descend

Post-conditions
---------------

After successful descend:

- current_domain points to child
- PC points to child entry point
- Parent's context saved in parent DCB
- Child's domain_id assigned

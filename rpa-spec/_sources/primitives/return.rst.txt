.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

return Primitive
================

The return primitive resumes child domain after parent completes service.

Instruction Format
------------------

.. code-block:: text

   return

Execution Steps
---------------

.. code-block:: text

   return:

   Step 1: Locate Child DCB
   ─────────────────────────
   child_dcb = current_dcb.child_block

   if child_dcb == 0:
       fault(NO_CHILD_DOMAIN)

   Step 2: Domain Switch
   ─────────────────────
   current_domain = child_domain

   Step 3: Context Restore
   ───────────────────────
   PC = child_dcb.saved_pc
   SP = child_dcb.saved_sp
   PSR = child_dcb.saved_psr

   // Return value in designated register
   R0 = child_dcb.return_value

Use Cases
---------

System Call Return
~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // In parent (kernel)
   // After handling system call
   R0 = return_value    // Set return value
   return               // Resume child

Nested Virtualization
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Hypervisor returns to guest after VM exit handling
   return

   // Hardware restores guest context from child DCB

Error Conditions
----------------

.. list-table:: return Errors
   :widths: 25 75
   :header-rows: 1

   * - Error
     - Cause
   * - NO_CHILD
     - child_block is 0
   * - CHILD_NOT_SUSPENDED
     - Child is not in suspended state

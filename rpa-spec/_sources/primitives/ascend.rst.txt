.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

ascend Primitive
================

The ascend primitive returns to parent domain to request service.

Instruction Format
------------------

.. code-block:: text

   ascend <service_type>

   service_type: Register indicating requested service

Execution Steps
---------------

.. code-block:: text

   ascend R0 (R0 = service_type):

   Step 1: Context Save
   ─────────────────────
   current_dcb.saved_pc = PC + 4
   current_dcb.saved_sp = SP
   current_dcb.saved_psr = PSR

   // Pass service type in designated register
   parent_context.arg_reg = R0

   Step 2: Domain Switch
   ─────────────────────
   current_domain = current_domain.parent

   Step 3: Trap Vector Resolution
   ───────────────────────────────
   while current_dcb.trap_vector == 0:
       if current_domain.is_root:
           fault(UNHANDLED_TRAP)
       current_domain = current_domain.parent

   Step 4: Jump to Handler
   ───────────────────────
   PC = current_dcb.trap_vector

Trap Propagation
----------------

When trap_vector is 0, the trap propagates up the hierarchy:

.. code-block:: text

   Domain 3: ascend (trap_vector = 0)
       │
       ▼ Propagate
   Domain 2: (trap_vector = 0)
       │
       ▼ Propagate
   Domain 1: (trap_vector = handler)
       │
       ▼ Jump to handler

This enables:

- Default fault handling at root
- Selective interception at intermediate levels
- Security policy enforcement

Service Types
-------------

.. list-table:: Standard Service Types
   :widths: 15 85
   :header-rows: 1

   * - Value
     - Service
   * - 0x00
     - System call
   * - 0x01
     - Page fault
   * - 0x02
     - I/O request
   * - 0x03
     - Security violation
   * - 0x04-0xFF
     - Implementation-defined

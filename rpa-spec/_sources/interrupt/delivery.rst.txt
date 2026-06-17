.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

Interrupt Delivery
==================

Interrupt delivery in RPA follows a domain-relative model with automatic
propagation support.

Delivery Sequence
-----------------

When an interrupt occurs:

.. code-block:: text

   Step 1: Identify Target Domain
   ───────────────────────────────
   target_domain = interrupt_routing[interrupt_id]

   Step 2: Check Interrupt Mask
   ─────────────────────────────
   if target_dcb.interrupt_mask[interrupt_id] == 0:
       pending_interrupts |= (1 << interrupt_id)
       return  // Interrupt not delivered

   Step 3: Check Handler
   ─────────────────────
   if target_dcb.interrupt_vector == 0:
       // No handler: propagate to parent
       propagate_interrupt(target_domain, interrupt_id)
       return

   Step 4: Deliver to Handler
   ──────────────────────────
   // Save context
   target_dcb.saved_pc = PC
   target_dcb.saved_sp = SP
   target_dcb.saved_psr = PSR

   // Jump to handler
   PC = target_dcb.interrupt_vector + (interrupt_id * 16)

Interrupt Propagation
---------------------

If a domain has no interrupt handler (interrupt_vector = 0):

.. code-block:: text

   propagate_interrupt(domain, interrupt_id):

   while domain.dcb.interrupt_vector == 0:
       if domain.is_root:
           // Unhandled interrupt at root
           fault(UNHANDLED_INTERRUPT)
           return

       domain = domain.parent

   // Deliver to ancestor with handler
   deliver_to_handler(domain, interrupt_id)

Example: Nested Virtualization
-------------------------------

.. code-block:: text

   Interrupt delivery in nested VM:

   ┌─────────────────────────────────────────┐
   │ Domain 0: Root (Hypervisor)              │
   │   interrupt_vector = 0x8000             │
   └─────────────────────────────────────────┘
                     ▲
                     │ Propagate (no handler)
   ┌─────────────────────────────────────────┐
   │ Domain 1: Guest Hypervisor               │
   │   interrupt_vector = 0                   │
   └─────────────────────────────────────────┘
                     ▲
                     │ Propagate (no handler)
   ┌─────────────────────────────────────────┐
   │ Domain 2: Guest OS                       │
   │   interrupt_vector = 0                   │
   └─────────────────────────────────────────┘
                     ▲
                     │ Interrupt occurs
   ┌─────────────────────────────────────────┐
   │ Domain 3: Guest Application              │
   │   (Running, interrupted)                 │
   └─────────────────────────────────────────┘

   Result: Root hypervisor handles all device interrupts
   Guest hypervisor can intercept by setting interrupt_vector

Interrupt Priority
------------------

Interrupts have configurable priority levels:

.. code-block:: text

   DCB Field: interrupt_priority[32]
   ─────────────────────────────────
   Priority for each interrupt (0 = highest)

   Hardware behavior:
   if pending_interrupts != 0:
       highest = find_highest_priority(pending_interrupts)
       deliver(highest)

Interrupt Latency
-----------------

RPA minimizes interrupt latency through:

.. list-table:: Latency Factors
   :widths: 30 70
   :header-rows: 1

   * - Factor
     - Impact
   * - No privilege escalation
     - Direct delivery to domain
   * - Single context switch
     - Hardware saves/restores context
   * - Maskable propagation
     - Optional interception at any level

.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Interrupt Handling Examples
===========================

This chapter demonstrates two interrupt scenarios: one handled locally, one propagated to parent.

Interrupt Delivery Overview
---------------------------

When an interrupt occurs:

1. Hardware identifies target domain
2. If domain has interrupt handler, deliver
3. If not, propagate to parent domain
4. Repeat until handled

Example 1: Locally Handled Interrupt
-------------------------------------

Scenario: Timer interrupt for Guest OS scheduler.

Setup
~~~~~

.. code-block:: text

   Domain 2: Guest OS
   - Has timer configured
   - Has interrupt handler registered

   Domain 3: User Application (running)
   - No timer access
   - Will be preempted

Interrupt Configuration
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Guest OS registered timer interrupt
   irq_timer = irq_request(IRQ_TYPE_TIMER)
   dcb2.interrupt_ctrl = irq_controller

   // Registered handler address
   timer_handler = 0x8000_1000

Interrupt Arrival
~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Hardware timer fires
   // Interrupt controller signals Domain 2

Hardware Actions
~~~~~~~~~~~~~~~~

.. code-block:: text

   // Domain 3 is running, Domain 2 owns the interrupt

   1. Save Domain 3 context (if different owner)
      dcb3.saved_pc = PC
      dcb3.saved_sp = SP
      dcb3.saved_psr = PSR

   2. Switch to Domain 2
      current_domain = domain_2

   3. Check if Domain 2 handles this interrupt
      irq_owner = interrupt_get_owner(irq_timer)
      if irq_owner == domain_2:
          // Deliver locally

   4. Jump to interrupt handler
      PC = dcb2.interrupt_handler

Guest OS Handler
~~~~~~~~~~~~~~~~

.. code-block:: text

   timer_handler:
       // Save current context
       push {r0-r12, lr}

       // Handle timer: scheduler tick
       bl scheduler_tick

       // Restore and return to Domain 3
       pop {r0-r12, lr}
       return

Timeline
~~~~~~~~

.. code-block:: text

   Time │ Domain │ Action
   ─────┼────────┼──────────────────────────────────
    T0  │   3    │ Application running
    T1  │   HW   │ Timer interrupt arrives
    T2  │   HW   │ Save Domain 3 context
    T3  │   HW   │ Switch to Domain 2
    T4  │   HW   │ Jump to timer_handler
    T5  │   2    │ Guest OS scheduler runs
    T6  │   2    │ Execute return
    T7  │   HW   │ Restore Domain 3 context
    T8  │   3    │ Application resumes

Example 2: Propagated Interrupt
-------------------------------

Scenario: Network interrupt while User Application running.

Setup
~~~~~

.. code-block:: text

   Domain 2: Guest OS
   - Has network driver
   - trap_vector = 0 (propagate unhandled traps)

   Domain 3: User Application
   - No interrupt handlers
   - Running computation

Interrupt Configuration
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Network interrupt owned by Hypervisor (Domain 1)
   // Guest OS does not own network hardware

   dcb2.interrupt_ctrl = virtual_irq_only
   dcb2.trap_vector = 0    // Propagate unhandled

Interrupt Arrival
~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Network interrupt arrives
   // Target: Domain 1 (Hypervisor)

Hardware Actions
~~~~~~~~~~~~~~~~

.. code-block:: text

   // Domain 3 running, interrupt for Domain 1

   1. Implicit ascend from Domain 3
      // Domain 3 has no handler, trap to Domain 2
      dcb3.saved_pc = PC
      current_domain = domain_2

   2. Check Domain 2 trap_vector
      if dcb2.trap_vector == 0:
          // Propagate to Domain 1

   3. Implicit ascend to Domain 1
      dcb2.saved_pc = PC
      current_domain = domain_1

   4. Check Domain 1 interrupt ownership
      if interrupt_owner == domain_1:
          PC = dcb1.network_handler

Hypervisor Handler
~~~~~~~~~~~~~~~~~~

.. code-block:: text

   network_handler:
       // Read network packet
       // Determine target guest
       // Inject virtual interrupt to Domain 2
       virq_inject(domain_2, VIRQ_NETWORK)

       // Return down the chain
       return    // To Domain 2
       return    // To Domain 3

Timeline
~~~~~~~~

.. code-block:: text

   Time │ Domain │ Action
   ─────┼────────┼──────────────────────────────────
    T0  │   3    │ Application running
    T1  │   HW   │ Network interrupt (for Domain 1)
    T2  │   HW   │ Implicit ascend: 3 → 2
    T3  │   HW   │ trap_vector = 0, propagate
    T4  │   HW   │ Implicit ascend: 2 → 1
    T5  │   HW   │ Jump to network_handler
    T6  │   1    │ Hypervisor handles packet
    T7  │   1    │ return (to Domain 2)
    T8  │   HW   │ Restore Domain 2 context
    T9  │   2    │ return (to Domain 3)
    T10 │   HW   │ Restore Domain 3 context
    T11 │   3    │ Application resumes

Comparison
----------

.. list-table:: Interrupt Handling Comparison
   :widths: 20 40 40
   :header-rows: 1

   * - Aspect
     - Local Handling
     - Propagated Handling
   * - Handler location
     - Same domain as interrupt owner
     - Ancestor domain
   * - Context switches
     - 1 (current → handler)
     - N (propagate through N levels)
   * - Latency
     - Lower
     - Higher (proportional to levels)
   * - Use case
     - Guest OS timer, device
     - Shared hardware, security

Interrupt Controller Configuration
----------------------------------

Per-Domain Configuration
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   struct interrupt_config {
       irq_mask;           // Which IRQs this domain handles
       irq_handler;        // Handler address
       propagate_mask;     // IRQs to always propagate
   }

   // Domain 2 (Guest OS)
   interrupt_config.irq_mask = {TIMER, UART}
   interrupt_config.propagate_mask = {NETWORK, DISK}

   // Domain 1 (Hypervisor)
   interrupt_config.irq_mask = {ALL}

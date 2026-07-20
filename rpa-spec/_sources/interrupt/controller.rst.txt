.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Interrupt Controller
====================

RPA domains interact with interrupt controllers through domain-relative
interrupt routing.

Interrupt Routing
-----------------

Each domain can receive interrupts from:

.. list-table:: Interrupt Sources
   :widths: 25 75
   :header-rows: 1

   * - Source
     - Description
   * - Device interrupts
     - From devices assigned to the domain
   * - Timer interrupts
     - Per-domain virtual timers
   * - Inter-processor interrupts
     - From other domains or processors
   * - Software interrupts
     - Self-generated via IPI mechanism

Interrupt Vector Table
----------------------

Each DCB contains an interrupt vector table base:

.. code-block:: text

   DCB Field: interrupt_vector
   ───────────────────────────
   Address of interrupt vector table

   Vector Table Entry (per interrupt):
   ┌──────────────────────────────────┐
   │ handler_address (64-bit)         │
   ├──────────────────────────────────┤
   │ interrupt_attributes (32-bit)    │
   └──────────────────────────────────┘

Interrupt Enable Mask
---------------------

DCB contains an interrupt enable mask:

.. code-block:: text

   DCB Field: interrupt_mask
   ──────────────────────────
   Bit N = 1: Interrupt N enabled
   Bit N = 0: Interrupt N disabled

   Hardware checks mask before delivery:
   if interrupt_mask[interrupt_id] == 0:
       // Interrupt pending but not delivered
       pending_interrupts |= (1 << interrupt_id)

Domain Isolation
----------------

Interrupts are isolated between domains:

.. code-block:: text

   Domain A interrupt does NOT affect Domain B:
   ┌─────────────────────────────────────────┐
   │ Domain A: Timer interrupt → Handler A    │
   ├─────────────────────────────────────────┤
   │ Domain B: No interrupt visible           │
   └─────────────────────────────────────────┘

   This isolation is enforced by:
   1. Per-domain interrupt_mask
   2. Per-domain interrupt_vector
   3. Hardware routing based on domain assignment

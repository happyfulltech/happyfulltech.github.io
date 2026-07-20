.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Trap Mechanism
==============

Traps are synchronous exceptions that cause implicit ascend to parent domain.

Trap Sources
------------

.. list-table:: Trap Sources
   :widths: 25 75
   :header-rows: 1

   * - Source
     - Description
   * - System call
     - Explicit ascend by software
   * - Page fault
     - Address translation failure
   * - Privilege violation
     - Privileged instruction in user mode
   * - IPA boundary violation
     - Access outside permitted regions
   * - Alignment fault
     - Misaligned memory access

Trap Handling
-------------

.. code-block:: text

   On trap:
   1. Hardware saves context to current DCB
   2. Hardware performs implicit ascend
   3. Parent's trap_handler executes
   4. Handler either:
      - Resolves and returns
      - Propagates to its parent
      - Terminates domain

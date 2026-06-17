.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

Trap Propagation
================

When a domain's trap_vector is 0, traps propagate to parent.

Propagation Chain
-----------------

.. code-block:: text

   Domain N: trap occurs
       │
       │ trap_vector = 0?
       ├─ No → Execute handler
       │
       └─ Yes → Propagate to Domain N-1
              │
              │ trap_vector = 0?
              ├─ No → Execute handler
              │
              └─ Yes → Continue propagation

Root Domain Behavior
--------------------

If trap reaches root domain with trap_vector = 0:

.. code-block:: text

   // Root must handle all unhandled traps
   if root_dcb.trap_vector == 0:
       // System error
       halt()

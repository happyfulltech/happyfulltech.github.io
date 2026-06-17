.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

Privilege Model
===============

RPA replaces traditional fixed privilege levels with flexible
domain-based privilege management.

Traditional vs RPA
------------------

.. code-block:: text

   Traditional (e.g., ARM, x86, RISC-V):
   ┌──────────────────────────────────────────────────┐
   │ Level 0: User Mode                               │
   │ Level 1: Supervisor (OS)                         │
   │ Level 2: Hypervisor                              │
   │ Level 3: Secure Monitor                          │
   │                                                  │
   │ Levels are FIXED in hardware                     │
   └──────────────────────────────────────────────────┘

   RPA:
   ┌──────────────────────────────────────────────────┐
   │ Domain N: Arbitrary privilege level              │
   │ Domain N+1: Higher privilege                     │
   │ Domain N+2: Even higher privilege                │
   │                                                  │
   │ Privilege determined by position in hierarchy    │
   │ Number of levels is SOFTWARE-DEFINED             │
   └──────────────────────────────────────────────────┘

Privilege Determination
-----------------------

A domain's privilege is relative to its position:

.. code-block:: text

   Relative privilege:

   Domain A has HIGHER privilege than Domain B if:
   - A is an ancestor of B in the domain tree
   - A can intercept B's traps
   - A can access B's memory (subject to security groups)

   Domain A has EQUAL privilege to Domain B if:
   - A and B are the same domain

   Domain A has LOWER privilege than Domain B if:
   - A is a descendant of B in the domain tree

Privilege Inheritance
---------------------

Child domains inherit restrictions from parents:

.. code-block:: text

   Child cannot:
   - Access memory outside parent-granted IPA regions
   - Use instructions parent has disabled
   - Bypass parent's security policies

   Parent can:
   - Intercept any child trap
   - Terminate child domain
   - Modify child's DCB

Privilege Escalation
--------------------

Privilege escalation requires explicit domain transition:

.. code-block:: text

   To gain privilege:
   - ascend: Request service from higher-privilege domain
   - Higher domain decides whether to grant access

   Explicit escalation:
   ┌──────────────────────────────────────────────────┐
   │ Application (low privilege):                     │
   │   ascend SYSTEM_CALL                             │
   │       │                                          │
   │       ▼                                          │
   │ OS (higher privilege):                           │
   │   - Validates request                            │
   │   - Performs privileged operation                │
   │   - return with result                           │
   └──────────────────────────────────────────────────┘

   No implicit escalation:
   - Cannot "switch to kernel mode"
   - Must go through domain boundary
   - Parent always in control

Capability Model
----------------

RPA can implement capability-based security:

.. code-block:: text

   DCB as capability:

   A DCB represents a capability to:
   - Execute code
   - Access specific memory regions
   - Create child domains

   Capability rules:
   1. Can only create capabilities for resources you own
   2. Cannot amplify your own capabilities
   3. Can only reduce capabilities of children

   Example:
   ┌──────────────────────────────────────────────────┐
   │ Parent DCB: Full memory access                   │
   │                                                  │
   │ Child DCB creation:                              │
   │   - Parent selects subset of memory              │
   │   - Child gets restricted IPA regions            │
   │   - Child cannot exceed parent's access          │
   └──────────────────────────────────────────────────┘

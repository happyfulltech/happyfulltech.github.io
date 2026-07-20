.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

System Diagram
==============

This chapter provides visual overview of RPA system organization.

Domain Hierarchy
----------------

.. code-block:: text

   Example: Full System Stack

   ┌──────────────────────────────────────────────────┐
   │ Domain 0: Secure Monitor (Root)                  │
   │ ┌──────────────────────────────────────────────┐│
   │ │ Domain 1: Hypervisor                          ││
   │ │ ┌────────────────────────────────────────────┐││
   │ │ │ Domain 2: Guest OS                          │││
   │ │ │ ┌──────────────────────────────────────────┐│││
   │ │ │ │ Domain 3: Application                    ││││
   │ │ │ └──────────────────────────────────────────┘│││
   │ │ │ ┌──────────────────────────────────────────┐│││
   │ │ │ │ Domain 4: Secure Enclave                 ││││
   │ │ │ └──────────────────────────────────────────┘│││
   │ │ └────────────────────────────────────────────┘││
   │ │ ┌────────────────────────────────────────────┐││
   │ │ │ Domain 5: Second Guest OS                  │││
   │ │ │ ┌──────────────────────────────────────────┐│││
   │ │ │ │ Domain 6: Application                    ││││
   │ │ │ └──────────────────────────────────────────┘│││
   │ │ └────────────────────────────────────────────┘││
   │ └──────────────────────────────────────────────┘│
   └──────────────────────────────────────────────────┘

Address Translation Flow
------------------------

.. code-block:: text

   Memory Access in Nested Domain:

   Application (Domain 3)
   │
   │ Virtual Address
   ▼
   ┌──────────────────────────────────────────────────┐
   │ Domain 3 Page Table                              │
   │ VA → IPA (Guest Physical)                        │
   └──────────────────────────────────────────────────┘
   │
   │ Intermediate Physical Address
   ▼
   ┌──────────────────────────────────────────────────┐
   │ IPA Boundary Check                               │
   │ Is IPA within Domain 3's permitted regions?      │
   └──────────────────────────────────────────────────┘
   │
   ▼
   ┌──────────────────────────────────────────────────┐
   │ Domain 2 (Guest OS) Page Table                   │
   │ IPA → IPA (Hypervisor Physical)                  │
   └──────────────────────────────────────────────────┘
   │
   ▼
   ┌──────────────────────────────────────────────────┐
   │ Domain 1 (Hypervisor) Page Table                 │
   │ IPA → PA (Physical)                              │
   └──────────────────────────────────────────────────┘
   │
   │ Physical Address
   ▼
   ┌──────────────────────────────────────────────────┐
   │ Memory                                           │
   └──────────────────────────────────────────────────┘

Domain Transition
-----------------

.. code-block:: text

   ascend Primitive:

   Domain 3 (Application)          Domain 2 (OS)
   ┌──────────────────────┐        ┌──────────────────────┐
   │ Executing            │        │                      │
   │                      │        │                      │
   │ ascend SYSCALL       │        │                      │
   │ ─────────────────────┼───────>│                      │
   │                      │        │ Trap handler         │
   │                      │        │ executes             │
   │                      │        │                      │
   │                      │        │ return               │
   │ <────────────────────┼────────│                      │
   │ Resumes execution    │        │                      │
   └──────────────────────┘        └──────────────────────┘

   Timeline:
   1. Application executes normally
   2. Application executes ascend instruction
   3. Hardware saves context to Domain 3 DCB
   4. Hardware switches to Domain 2
   5. Domain 2 trap handler executes
   6. Domain 2 executes return instruction
   7. Hardware restores Domain 3 context
   8. Application resumes execution

Security Group Isolation
------------------------

.. code-block:: text

   Security Group Access Control:

   ┌──────────────────────────────────────────────────┐
   │ Domain 1: OS (Group 1)                           │
   │ ┌──────────────────────────────────────────────┐│
   │ │ Domain 2: App A (Group 2)                     ││
   │ │ [Memory: 0x10000-0x1FFFF]                     ││
   │ └──────────────────────────────────────────────┘│
   │ ┌──────────────────────────────────────────────┐│
   │ │ Domain 3: App B (Group 2)                     ││
   │ │ [Memory: 0x20000-0x2FFFF]                     ││
   │ └──────────────────────────────────────────────┘│
   │ ┌──────────────────────────────────────────────┐│
   │ │ Domain 4: Enclave (Group 3)                   ││
   │ │ [Memory: 0x30000-0x3FFFF]                     ││
   │ │ [Encrypted with Key 1]                        ││
   │ └──────────────────────────────────────────────┘│
   └──────────────────────────────────────────────────┘

   Access Matrix:
   ┌─────────────────────────────────────────────────┐
   │ From/To   │ App A │ App B │ Enclave │ OS       │
   ├─────────────────────────────────────────────────┤
   │ OS        │  R/W  │  R/W  │   -     │  R/W     │
   │ App A     │  R/W  │   -   │   -     │   -      │
   │ App B     │   -   │  R/W  │   -     │   -      │
   │ Enclave   │   -   │   -   │  R/W    │   -      │
   └─────────────────────────────────────────────────┘

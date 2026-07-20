.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Key Features
============

Extensible Isolation
--------------------

RPA scales from microcontrollers to servers:

.. list-table:: Scalability Comparison
   :widths: 25 25 50
   :header-rows: 1

   * - Platform
     - Configuration
     - Capability
   * - MCU
     - INHERIT mode, 64-byte DCB
     - Lightweight kernel-user separation
   * - Embedded
     - 2-3 levels, minimal page tables
     - RTOS isolation, application sandboxing
   * - Desktop
     - Full page tables, 4-8 levels
     - OS, hypervisor, sandbox support
   * - Server
     - Unlimited levels, full features
     - Deep nesting, confidential computing

INHERIT Mode
~~~~~~~~~~~~

When DCB pagetable field is 0, child shares parent's address space:

- Transition overhead: 5-10 cycles (vs 100-500 for full switch)
- No TLB flush required
- Suitable for lightweight isolation

Page Table Stacking
~~~~~~~~~~~~~~~~~~~

Each domain can have independent page table:

- VA → IPA₁ → IPA₂ → ... → PA
- Each level checks IPA boundaries
- Unlimited nesting depth

ISA-Independent Privilege Management
------------------------------------

RPA overlays on any ISA with minimal modification:

.. list-table:: ISA Integration
   :widths: 20 30 50
   :header-rows: 1

   * - ISA
     - Native Privilege
     - RPA Integration
   * - ARM
     - EL0-EL3 (4 levels)
     - RPA replaces EL mechanism
   * - x86
     - Ring 0-3 (4 levels)
     - RPA replaces Ring mechanism
   * - RISC-V
     - M/S/U (3 modes)
     - RPA replaces mode mechanism
   * - IBM Z
     - Supervisor/Problem (2 levels)
     - RPA extends nesting

New ISA design only needs to:

1. Define context save format
2. Implement two primitives (descend/ascend)
3. Define privilege level mapping

Native Heterogeneous-ISA Support
--------------------------------

Domains with different ISAs can coexist:

- DCB includes isa_tag field
- Hardware performs context conversion on cross-ISA transitions
- Calling convention mapping is automatic

Use case: IBM-Arm collaboration for enterprise systems

Security Domain Separation
--------------------------

Management rights vs access rights:

.. list-table:: Rights Separation
   :widths: 25 75
   :header-rows: 1

   * - Right Type
     - Description
   * - Management
     - Parent configures child's ipa_regions, pagetable
   * - Access
     - Parent cannot read child's private memory

This enables confidential computing:

- Cloud provider manages resources
- Cannot access tenant data
- Nested confidential VMs supported natively

Unified Abstraction
-------------------

All privilege transitions use the same mechanism:

.. list-table:: Unified Transitions
   :widths: 25 25 50
   :header-rows: 1

   * - Traditional Mechanism
     - RPA Equivalent
     - Description
   * - syscall
     - ascend
     - User → kernel
   * - VM entry
     - descend
     - Hypervisor → guest
   * - VM exit
     - ascend
     - Guest → hypervisor
   * - SMC
     - descend
     - Normal → secure world
   * - Exception
     - implicit ascend
     - Fault propagation

This unification simplifies hardware design and enables composition of isolation mechanisms.

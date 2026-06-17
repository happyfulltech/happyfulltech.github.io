.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

Terminology
===========

Core Concepts
-------------

Domain
   A privilege domain is the fundamental isolation unit in RPA.
   Each domain has its own address space and privilege level.
   Domains are organized in a tree hierarchy.

Domain Control Block (DCB)
   Memory-resident data structure containing domain configuration.
   Stores page table base, IPA regions, exception vectors, and context.

descend
   Primitive operation to enter a child domain.
   Saves current context, loads child configuration, switches execution.

ascend
   Primitive operation to return to parent domain.
   Saves current context, switches to parent, jumps to trap handler.

return
   Primitive operation to resume child domain after service.
   Restores child context from DCB.

Address Concepts
----------------

Virtual Address (VA)
   Address used by software in current domain.

Intermediate Physical Address (IPA)
   Address after page table translation in child domain.
   Subject to IPA boundary checking before further translation.

Physical Address (PA)
   Final address in root domain's address space (actual DRAM).

IPA Regions
   Boundary constraints defining accessible address space for a domain.
   Set by parent, enforced by hardware.

Modes
-----

INHERIT Mode
   Child shares parent's address space (pagetable = 0).
   No address translation, only context save/restore.

Independent Mode
   Child has its own page table (pagetable ≠ 0).
   Full VA → IPA translation with boundary checking.

Security Concepts
-----------------

Security Group
   Mechanism separating management rights from access rights.
   Binds domain to encryption keys and access policies.

Attestation
   Report proving domain identity and integrity.
   Signed by hardware-derived keys.

Confidential Domain
   Domain with memory encryption enabled.
   Parent cannot read child's data despite configuring resources.

ISA Concepts
------------

isa_tag
   DCB field identifying domain's instruction set.
   Enables heterogeneous-ISA support.

Context Conversion
   Hardware process mapping registers between ISAs.
   Preserves calling convention semantics during cross-ISA transitions.

Hierarchy Concepts
------------------

Root Domain
   Domain 0, the trust anchor.
   Highest privilege, owns all physical memory.

Child Domain
   Domain created by parent via descend.
   Has restricted address space and lower privilege.

Relative Perspective
   Each domain views itself as Domain 0.
   Children are Domain 1, 2, 3...
   Parent is Domain -1.

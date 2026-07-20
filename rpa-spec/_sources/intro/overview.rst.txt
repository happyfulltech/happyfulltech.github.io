.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Overview
========

Modern computing systems face two growing challenges: increasing virtualization nesting depth and expanding security domain requirements.

The Challenge
-------------

Virtualization Nesting Depth
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cloud infrastructure increasingly requires multiple layers of virtualization:

- Cloud providers run hypervisors on bare metal
- Tenants run nested hypervisors for isolation
- Container orchestrators add additional abstraction layers
- Sandbox environments require further isolation

Traditional architectures face limitations:

- x86 VT-x: 2-level nesting requires software emulation
- ARM: No native nested virtualization support
- RISC-V: H-extension limited to 2 levels

Each additional layer incurs 10-30% performance overhead when software emulation is required.

Security Domain Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Confidential computing demands hardware-enforced isolation:

- Cloud tenants need protection from providers
- Multi-tenant systems require inter-tenant isolation
- Secure enclaves need nesting for complex workloads

Existing solutions lack nesting:

- Intel SGX/TDX: Single-level Trust Domains
- AMD SEV/SNP: No nested confidential VMs
- ARM CCA: Fixed-depth Realms

RPA addresses these challenges through a unified recursive model.

The RPA Approach
----------------

Recursive Privilege Architecture (RPA) decouples privilege management from instruction set design through:

Memory-Resident Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Privilege configuration is stored in ordinary memory (Domain Control Block), not hardware registers:

- Unlimited nesting depth bounded by memory, not hardware
- Configuration is data-driven, not instruction-driven
- New privilege levels require no hardware changes

Unified Primitives
~~~~~~~~~~~~~~~~~~

All privilege transitions use two symmetric primitives:

- **descend**: Enter a child domain
- **ascend**: Return to parent domain for service

These primitives replace:

- System calls (syscall/sysret)
- VM entry/exit
- TEE switches (SMC)
- Exception handling

ISA Independence
~~~~~~~~~~~~~~~~

Privilege management is ISA-agnostic:

- Same architecture works on ARM, x86, RISC-V, IBM Z
- New ISAs need only define context format
- Heterogeneous-ISA domains can coexist

Target Audience
---------------

This specification is intended for:

- Hardware architects implementing RPA
- System software developers targeting RPA platforms
- Security architects designing confidential computing solutions
- Researchers studying privilege architecture

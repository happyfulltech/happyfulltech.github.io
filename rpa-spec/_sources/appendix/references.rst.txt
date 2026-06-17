.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

References
==========

Related Architectures
---------------------

.. list-table:: Related Work
   :widths: 30 70
   :header-rows: 1

   * - Architecture
     - Relationship
   * - ARM v8-A
     - Exception levels (EL0-EL3), virtualization
   * - RISC-V
     - Privilege modes, physical memory protection
   * - x86-64
     - Protection rings, VMX
   * - Intel SGX
     - Enclave isolation, attestation
   * - AMD SEV
     - Encrypted VMs, secure nested paging
   * - ARM CCA
     - Realm domains, granule protection

Normative References
--------------------

.. [ARMARM] ARM Architecture Reference Manual ARMv8, for ARMv8-A architecture profile, ARM Ltd.

.. [RISCVPRIV] The RISC-V Instruction Set Manual, Volume II: Privileged Architecture, RISC-V Foundation.

.. [INTELSDM] Intel 64 and IA-32 Architectures Software Developer's Manual, Intel Corporation.

.. [AMDAPM] AMD64 Architecture Programmer's Manual, AMD Inc.

Informative References
----------------------

.. [PYSIM] RPA Python Simulator (pysim), https://github.com/arthurdev000/rpa-pysim, Version 0.7.0, 107 test cases.

.. [SGX] Intel SGX Explained, Costan and Devadas, IACR Cryptology ePrint Archive, 2016.

.. [SEV] SEV-ES Architecture Overview, AMD Inc., 2020.

.. [CCA] Arm Confidential Compute Architecture, ARM Ltd., 2021.

.. [PMP] Physical Memory Protection in RISC-V, RISC-V Foundation.

.. [NESTED] Performance Analysis of Nested Virtualization, Armbrust et al., USENIX ATC, 2022.

Glossary
--------

.. glossary::

   DCB
      Domain Control Block. Memory-resident data structure containing
      domain configuration.

   IPA
      Intermediate Physical Address. Address after first-level
      translation in nested systems.

   INHERIT
      Translation mode where child uses parent's page tables.

   ascend
      RPA primitive for returning to parent domain.

   descend
      RPA primitive for entering a child domain.

   return
      RPA primitive for resuming child domain execution.

   Security Group
      Access control mechanism separating management rights from
      access rights.

   Domain
      Isolated execution context with configurable privilege.

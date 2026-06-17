.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

Confidential Computing
======================

RPA enables confidential computing through domain isolation and
hardware-enforced security.

Confidential Computing Model
----------------------------

Confidential computing protects data in use:

.. code-block:: text

   Traditional Protection:
   ┌──────────────────────────────────────────────────┐
   │ Data at rest:    Encryption ✓                    │
   │ Data in transit: Encryption ✓                    │
   │ Data in use:     Plaintext ✗                    │
   └──────────────────────────────────────────────────┘

   Confidential Computing:
   ┌──────────────────────────────────────────────────┐
   │ Data at rest:    Encryption ✓                    │
   │ Data in transit: Encryption ✓                    │
   │ Data in use:     Isolated ✓                     │
   └──────────────────────────────────────────────────┘

Threat Model
------------

Confidential computing protects against privileged attackers:

.. list-table:: Protected Against
   :widths: 30 70
   :header-rows: 1

   * - Attacker
     - Mitigation
   * - Malicious admin
     - Domain isolation, security groups
   * - Compromised OS
     - Memory encryption, IPA boundaries
   * - Compromised hypervisor
     - Nested isolation, attestation
   * - Physical attacker
     - Memory encryption, secure key storage

RPA Confidential Computing Architecture
---------------------------------------

.. code-block:: text

   Confidential VM Architecture:

   ┌──────────────────────────────────────────────────┐
   │ Cloud Provider (Untrusted)                       │
   │ ┌──────────────────────────────────────────────┐│
   │ │ Domain 0: Cloud Hypervisor                    ││
   │ │   - Manages VM lifecycle                      ││
   │ │   - Cannot access VM memory                   ││
   │ │   - Cannot read VM secrets                    ││
   │ │ ┌────────────────────────────────────────────┐││
   │ │ │ Domain 1: Confidential VM                  │││
   │ │ │   - Security group: Isolated                │││
   │ │ │   - Memory: Encrypted                      │││
   │ │ │   - Attestation: Enabled                   │││
   │ │ │                                            │││
   │ │ │   - Customer workload                     │││
   │ │ │   - Customer data                          │││
   │ │ │   - Customer keys                          │││
   │ │ └────────────────────────────────────────────┘││
   │ └──────────────────────────────────────────────┘│
   └──────────────────────────────────────────────────┘

Security Configuration
----------------------

Confidential VM DCB configuration:

.. code-block:: text

   Confidential VM DCB:
   ┌──────────────────────────────────────────────────┐
   │ security_group    = 3  (Isolated)                │
   │ encryption_mode   = 3  (Confidentiality+Integrity)│
   │ encryption_key_id = 1  (Unique to VM)            │
   │ attestation_key_id= 1  (Unique to VM)            │
   │ ipa_regions[]    = VM's assigned memory          │
   └──────────────────────────────────────────────────┘

   Effect:
   - Hypervisor cannot read VM memory (security group)
   - VM memory encrypted with unique key
   - VM can attest its configuration

VM Launch Protocol
------------------

.. code-block:: text

   Confidential VM Launch:

   1. Image Loading
   ──────────────────
   - Hypervisor loads VM image
   - Image is encrypted (customer key)
   - Hypervisor cannot read plaintext

   2. Domain Creation
   ───────────────────
   - Hypervisor creates domain
   - Sets security_group = Isolated
   - Sets encryption_key_id
   - Sets attestation_key_id

   3. Secret Injection
   ───────────────────
   - Customer sends secrets
   - Encrypted with VM's public key
   - Only VM can decrypt

   4. Attestation
   ──────────────
   - VM generates attestation report
   - Customer verifies report
   - Customer trusts VM

Use Cases
---------

.. list-table:: Confidential Computing Use Cases
   :widths: 25 75
   :header-rows: 1

   * - Use Case
     - Description
   * - Cloud ML training
     - Protect training data from cloud provider
   * - Financial processing
     - Protect transaction data
   * - Healthcare
     - Process patient data in compliance
   * - Blockchain
     - Secure key management for nodes

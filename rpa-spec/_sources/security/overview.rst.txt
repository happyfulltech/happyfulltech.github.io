.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Security Overview
=================

RPA provides hardware-enforced security through domain isolation,
access control, and memory protection.

Security Principles
-------------------

RPA security is built on three principles:

.. list-table:: Security Principles
   :widths: 25 75
   :header-rows: 1

   * - Principle
     - Description
   * - Least Privilege
     - Domains have minimum necessary access
   * - Isolation
     - Domains cannot access each other's memory
   * - Delegation
     - Access can be delegated but not amplified

Threat Model
------------

RPA addresses the following threats:

.. list-table:: Threats Addressed
   :widths: 30 70
   :header-rows: 1

   * - Threat
     - Mitigation
   * - Malicious application
     - IPA boundary enforcement
   * - Compromised OS
     - Security groups prevent OS access
   * - Malicious hypervisor
     - Nested isolation, confidential computing
   * - Side-channel attacks
     - Memory encryption, timing isolation

Security Mechanisms
-------------------

RPA provides multiple security mechanisms:

.. code-block:: text

   Mechanism           │ Level        │ Purpose
   ────────────────────┼──────────────┼──────────────────────────
   IPA Boundaries       │ Memory       │ Region-level isolation
   Page Permissions     │ Page         │ Fine-grained access control
   Domain Hierarchy     │ System       │ Privilege separation
   Security Groups      │ Management   │ Access control by group
   Memory Encryption    │ Physical     │ Memory confidentiality
   Attestation          │ Integrity    │ Remote verification

Defense in Depth
----------------

RPA supports defense in depth:

.. code-block:: text

   Layer 1: Application Isolation
   ──────────────────────────────
   Application cannot access other applications

   Layer 2: OS Isolation
   ─────────────────────
   OS cannot access security-group-protected memory

   Layer 3: Hypervisor Isolation
   ─────────────────────────────
   Hypervisor isolated from VMs via nested domains

   Layer 4: Hardware Isolation
   ────────────────────────────
   Memory encryption protects against physical attacks

Security Configuration
----------------------

Security features are configurable per domain:

.. code-block:: text

   DCB Security Fields:
   ┌──────────────────────────────────────────────────┐
   │ security_group    : Group membership             │
   │ encryption_enabled : Memory encryption flag       │
   │ attestation_key    : Attestation key reference    │
   │ ipa_regions[]      : Memory access control       │
   └──────────────────────────────────────────────────┘

   Parent sets these fields on domain creation.
   Child cannot modify its own security configuration.

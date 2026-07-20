.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Attestation
===========

Attestation enables remote verification of domain integrity and
isolation.

Attestation Overview
--------------------

Attestation proves to a remote party:

.. code-block:: text

   Attestation proves:
   ┌──────────────────────────────────────────────────┐
   │ 1. Domain is running expected code               │
   │ 2. Domain is properly isolated                   │
   │ 3. Domain has expected configuration             │
   │ 4. Hardware security features are enabled        │
   └──────────────────────────────────────────────────┘

Attestation Components
----------------------

.. code-block:: text

   Attestation Report Structure:
   ┌──────────────────────────────────────────────────┐
   │ Header                                           │
   │   - Report version                               │
   │   - Domain ID                                    │
   │   - Timestamp                                    │
   ├──────────────────────────────────────────────────┤
   │ Measurements                                     │
   │   - Code hash (SHA-256)                         │
   │   - Initial data hash                           │
   │   - Configuration hash                          │
   ├──────────────────────────────────────────────────┤
   │ Security State                                   │
   │   - Security group                               │
   │   - Encryption mode                              │
   │   - IPA regions                                  │
   │   - Parent domain ID                             │
   ├──────────────────────────────────────────────────┤
   │ Platform State                                   │
   │   - Hardware version                             │
   │   - Firmware version                             │
   │   - Security features enabled                    │
   ├──────────────────────────────────────────────────┤
   │ Signature                                        │
   │   - Sign(report, attestation_key)               │
   └──────────────────────────────────────────────────┘

Attestation Keys
----------------

Each domain has an attestation key:

.. code-block:: text

   DCB Field: attestation_key_id
   ─────────────────────────────
   Reference to attestation key

   Key Hierarchy:
   ┌──────────────────────────────────────────────────┐
   │ Root Attestation Key (RAK)                       │
   │   - Provisioned by manufacturer                  │
   │   - Never leaves hardware                        │
   │   - Signs platform certificates                  │
   ├──────────────────────────────────────────────────┤
   │ Domain Attestation Keys (DAK)                    │
   │   - Derived from RAK                             │
   │   - Unique per domain                            │
   │   - Signs domain attestation reports             │
   └──────────────────────────────────────────────────┘

Attestation Protocol
--------------------

.. code-block:: text

   Remote Attestation Flow:

   Verifier                Domain              Hardware
     │                       │                    │
     │  1. Challenge (nonce) │                    │
     │ ─────────────────────>│                    │
     │                       │                    │
     │                       │ 2. Request report  │
     │                       │ ──────────────────>│
     │                       │                    │
     │                       │ 3. Attestation     │
     │                       │    report          │
     │                       │ <──────────────────│
     │                       │                    │
     │  4. Attestation report │                    │
     │ <─────────────────────│                    │
     │                       │                    │
     │  5. Verify signature  │                    │
     │  6. Check measurements│                    │
     │  7. Trust decision    │                    │
     │                       │                    │

Measurement Chain
-----------------

Measurements form a chain of trust:

.. code-block:: text

   Measurement Chain:
   ┌──────────────────────────────────────────────────┐
   │ M0: Boot firmware                               │
   │ M1: Hypervisor (hash of M0 + hypervisor code)   │
   │ M2: OS (hash of M1 + OS code)                   │
   │ M3: Application (hash of M2 + app code)         │
   └──────────────────────────────────────────────────┘

   Each measurement includes:
   - Hash of code
   - Hash of parent measurement
   - Configuration at creation time

   Verification:
   - Verifier knows expected M3
   - Can compute expected M2, M1, M0
   - Compares with attestation report

.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

Security Groups
===============

Security groups provide access control independent of domain hierarchy,
separating management rights from access rights.

Concept
-------

Traditional domain hierarchy grants parent full access to child.
Security groups restrict this access:

.. code-block:: text

   Without security groups:
   ┌──────────────────────────────────────────────────┐
   │ Parent can:                                      │
   │   - Access all child memory                      │
   │   - Modify child's DCB                           │
   │   - Intercept all child traps                    │
   └──────────────────────────────────────────────────┘

   With security groups:
   ┌──────────────────────────────────────────────────┐
   │ Parent can:                                      │
   │   - Manage child (create, terminate)             │
   │   - BUT cannot access child's memory             │
   │   - IF child is in different security group      │
   └──────────────────────────────────────────────────┘

Security Group Definition
-------------------------

Each domain belongs to a security group:

.. code-block:: text

   DCB Field: security_group
   ─────────────────────────
   Identifier of security group (0-255)

   Security Group Table (system-wide):
   ┌──────────────────────────────────────────────────┐
   │ Group 0: Root (full access)                      │
   │ Group 1: Trusted OS                              │
   │ Group 2: Applications                            │
   │ Group 3: Secure Enclave                          │
   │ ...                                              │
   └──────────────────────────────────────────────────┘

Access Control Matrix
---------------------

Security groups define an access control matrix:

.. code-block:: text

   Access Matrix:
   ┌──────────────────────────────────────────────────┐
   │          │ Group 0 │ Group 1 │ Group 2 │ Group 3 │
   ├──────────────────────────────────────────────────┤
   │ Group 0  │   R/W   │   R/W   │   R/W   │   R/W   │
   │ Group 1  │    -    │   R/W   │   R/W   │    -    │
   │ Group 2  │    -    │    -    │   R/W   │    -    │
   │ Group 3  │    -    │    -    │    -    │   R/W   │
   └──────────────────────────────────────────────────┘

   R/W = Can read and write
   -   = No access

Memory Access Check
-------------------

Hardware checks security group on memory access:

.. code-block:: text

   check_access(source_domain, target_memory):

   source_group = source_domain.dcb.security_group
   target_group = target_memory.owner_group

   if access_matrix[source_group][target_group] == DENY:
       fault(SECURITY_GROUP_VIOLATION)

Example: Secure Enclave
-----------------------

.. code-block:: text

   Secure enclave protected from OS:

   Domain 0: Root (Group 0)
     └─ Domain 1: OS (Group 1)
        ├─ Domain 2: Application (Group 2)
        └─ Domain 3: Secure Enclave (Group 3)

   Access:
   ┌──────────────────────────────────────────────────┐
   │ OS → Application memory: ALLOWED                 │
   │ OS → Secure Enclave memory: DENIED               │
   │ Application → Secure Enclave: DENIED             │
   │ Secure Enclave → Own memory: ALLOWED             │
   └──────────────────────────────────────────────────┘

   OS can still:
   - Create/terminate enclave domain
   - Allocate memory for enclave
   - But cannot read enclave's private data

Management vs Access
--------------------

Security groups separate management from access:

.. code-block:: text

   Management rights (always inherited from hierarchy):
   - Create child domains
   - Terminate child domains
   - Configure child's DCB

   Access rights (controlled by security groups):
   - Read child's memory
   - Write child's memory
   - Execute child's code

   This separation enables:
   - Trusted execution environments
   - Confidential computing
   - Secure enclaves

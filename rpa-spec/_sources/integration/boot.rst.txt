.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

Boot Sequence
=============

This chapter describes the RPA system boot sequence.

Boot Stages
-----------

.. code-block:: text

   Boot Sequence:

   Stage 0: Hardware Reset
   ───────────────────────
   - Hardware initializes
   - Creates root domain (Domain 0)
   - Loads initial DCB at fixed address

   Stage 1: Boot Firmware
   ──────────────────────
   - Firmware executes in Domain 0
   - Initializes memory, devices
   - Verifies next stage (optional)
   - Creates Domain 1 for bootloader/OS

   Stage 2: Bootloader/Hypervisor
   ──────────────────────────────
   - Executes in Domain 1
   - Loads OS/hypervisor
   - Creates Domain 2 for OS

   Stage 3: Operating System
   ─────────────────────────
   - Executes in Domain 2
   - Initializes kernel
   - Creates Domain 3+ for applications

Initial DCB Configuration
-------------------------

.. code-block:: text

   Hardware creates initial DCB:

   Root DCB (created by hardware):
   ┌──────────────────────────────────────────────────┐
   │ ctrlblock_size    = minimum_size                │
   │ domain_id         = 0                           │
   │ parent_block      = 0 (no parent)               │
   │ child_block       = 0 (no child yet)            │
   │ trap_vector       = 0 (must be set by firmware) │
   │ interrupt_vector  = 0                           │
   │ page_table_root   = identity_mapping            │
   │ translation_mode  = OWN                         │
   │ isa_tag           = system_isa                  │
   │ security_group    = 0 (root)                    │
   │ saved_pc          = reset_vector                │
   │ saved_sp          = 0                           │
   │ ipa_regions[]     = full_physical_memory        │
   └──────────────────────────────────────────────────┘

Domain Creation During Boot
---------------------------

.. code-block:: text

   Firmware creates bootloader domain:

   Step 1: Allocate DCB
   ─────────────────────
   dcb_addr = allocate_memory(DCB_SIZE)
   assert(dcb_addr % 32 == 0)  // Must be aligned

   Step 2: Initialize DCB
   ──────────────────────
   memory[dcb_addr + ctrlblock_size_offset] = DCB_SIZE
   memory[dcb_addr + domain_id_offset] = 0  // Hardware assigns
   memory[dcb_addr + parent_block_offset] = root_dcb
   memory[dcb_addr + trap_vector_offset] = trap_handler_addr
   memory[dcb_addr + page_table_root_offset] = page_table_addr
   memory[dcb_addr + isa_tag_offset] = RISCV
   memory[dcb_addr + security_group_offset] = 0
   memory[dcb_addr + saved_pc_offset] = bootloader_entry

   Step 3: Configure IPA Regions
   ─────────────────────────────
   memory[dcb_addr + ipa_regions_offset + 0] = {
       base:  0x80000000,
       limit: 0x80FFFFFF,
       attr:  RWX
   }

   Step 4: Descend to New Domain
   ─────────────────────────────
   R0 = dcb_addr
   descend R0

Boot Example: Full System
-------------------------

.. code-block:: text

   Timeline:

   T0: Hardware reset
       - Create Domain 0 (root)
       - PC = 0xFFFFFFF0 (reset vector)

   T1: Firmware initialization
       - Initialize memory controller
       - Initialize essential devices
       - Load bootloader

   T2: Create Domain 1 (bootloader)
       - Allocate DCB for bootloader
       - Configure IPA regions
       - descend to bootloader

   T3: Bootloader execution
       - Load hypervisor/OS
       - Create Domain 2 (hypervisor)
       - descend to hypervisor

   T4: Hypervisor initialization
       - Initialize virtualization
       - Create Domain 3 (guest OS)
       - descend to guest OS

   T5: Guest OS initialization
       - Initialize kernel
       - Start services
       - Create Domain 4+ (applications)

Secure Boot
-----------

RPA supports secure boot through measurement:

.. code-block:: text

   Secure Boot Flow:

   1. Hardware verifies firmware signature
      - Using ROM public key
      - Firmware hash stored in TPM

   2. Firmware verifies bootloader
      - Using firmware's public key
      - Bootloader hash extends measurement

   3. Bootloader verifies OS
      - Using bootloader's public key
      - OS hash extends measurement

   4. Measurement chain available for attestation
      - Full chain of trust
      - Remote verification possible

   DCB fields for secure boot:
   ┌──────────────────────────────────────────────────┐
   │ Each domain's DCB contains:                       │
   │   - Code measurement (hash)                      │
   │   - Configuration measurement                    │
   │   - Parent measurement reference                 │
   └──────────────────────────────────────────────────┘

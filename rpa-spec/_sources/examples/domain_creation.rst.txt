.. SPDX-FileCopyrightText: Copyright 2026 RPA Architecture Team
.. SPDX-License-Identifier: CC-BY-SA-4.0

Domain Creation Example
=======================

This chapter walks through a complete example: OS boot to application execution, showing how each layer creates child domains.

Scenario Overview
-----------------

.. code-block:: text

   Domain 0: Firmware/Bootloader (Root)
   │
   ├── Domain 1: Hypervisor
   │   │
   │   ├── Domain 2: Guest OS Kernel
   │   │   │
   │   │   └── Domain 3: User Application
   │   │
   │   └── Domain 2': Another Guest VM
   │
   └── Domain 1': System Management Mode

Each transition demonstrates DCB configuration for different subsystems.

Step 1: Firmware Creates Hypervisor Domain
-------------------------------------------

Firmware (Domain 0) creates the hypervisor domain:

Memory Controller Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Firmware allocates memory for hypervisor
   HYPERVISOR_BASE = 0x0010_0000    // 1 MB
   HYPERVISOR_SIZE = 0x0100_0000    // 16 MB

   // Create IPA regions for hypervisor
   ipa_regions[0] = {
       base: 0x0000_0000,
       size: 0x0100_0000,    // 16 MB
       permissions: R|W|X
   }

   // Set DCB field
   dcb.ipa_regions = &ipa_regions[0]

Interrupt Controller Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Request interrupt controller for hypervisor
   irq_handle = sysop_irq_request(IRQ_POOL_HYPERVISOR)

   // Store handle in DCB
   dcb.interrupt_ctrl = irq_handle

Exception Handler Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Hypervisor will set its own trap handler after boot
   // Firmware sets trap_vector = 0 initially
   // (traps propagate to firmware if hypervisor not ready)
   dcb.trap_vector = 0

Security Configuration
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // No encryption for hypervisor (trusted)
   dcb.security_group = 0

   // ISA same as firmware
   dcb.isa_tag = ISA_TAG_INHERIT

Page Table Configuration
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Hypervisor gets direct physical memory access
   // Use INHERIT mode initially
   dcb.pagetable = 0    // INHERIT mode

Entry Point
~~~~~~~~~~~

.. code-block:: text

   // Write hypervisor entry point
   dcb.saved_lr = HYPERVISOR_ENTRY_POINT

   // Size and alignment
   dcb.ctrlblock_size = 8    // Minimum DCB

Execute descend
~~~~~~~~~~~~~~~~

.. code-block:: text

   // Firmware executes
   descend &dcb

   // Hardware:
   // 1. Assigns domain_id = 1
   // 2. Saves firmware context
   // 3. Switches to Domain 1
   // 4. Jumps to HYPERVISOR_ENTRY_POINT

Step 2: Hypervisor Creates Guest OS Domain
------------------------------------------

Hypervisor (Domain 1) creates a guest VM:

Memory Configuration
~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Allocate guest memory (may be encrypted)
   GUEST_BASE = 0x1000_0000    // 256 MB
   GUEST_SIZE = 0x1000_0000    // 256 MB

   // IPA regions for guest
   ipa_regions[0] = {
       base: 0x0000_0000,       // Guest sees 0-based addresses
       size: 0x1000_0000,
       permissions: R|W|X
   }

   dcb.ipa_regions = &ipa_regions[0]

Security Configuration
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Guest memory is encrypted (confidential computing)
   sec_group = security_group_create(ENCRYPTION_ENABLED)
   dcb.security_group = sec_group

   // Now hypervisor cannot read guest memory
   // (management rights only, no access rights)

Page Table Configuration
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Guest has independent address space
   pagetable = create_guest_pagetable()
   dcb.pagetable = pagetable

   // Hardware will:
   // 1. Translate guest VA → IPA
   // 2. Check IPA against ipa_regions
   // 3. Translate IPA → physical address

Interrupt Controller
~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Virtual interrupt controller for guest
   virq_handle = virq_controller_create()
   dcb.interrupt_ctrl = virq_handle

   // Hypervisor will intercept guest interrupts

Execute descend
~~~~~~~~~~~~~~~

.. code-block:: text

   descend &guest_dcb

Step 3: Guest OS Creates User Application
------------------------------------------

Guest OS kernel (Domain 2) creates user process:

Memory Configuration
~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // User process gets limited memory
   ipa_regions[0] = {
       base: 0x0000_0000,
       size: 0x0100_0000,    // 1 MB for code/data
       permissions: R|X
   }
   ipa_regions[1] = {
       base: 0x0100_0000,
       size: 0x0010_0000,    // 64 KB for heap
       permissions: R|W
   }
   ipa_regions[2] = {
       base: 0x7FFF_0000,
       size: 0x0001_0000,    // 64 KB for stack
       permissions: R|W
   }

   dcb.ipa_regions = &ipa_regions[0]

Privilege Restriction
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // User mode: no access to privileged instructions
   // Trap handler in kernel
   dcb.trap_vector = syscall_handler

   // Independent address space
   dcb.pagetable = user_pagetable

Execute descend
~~~~~~~~~~~~~~~

.. code-block:: text

   // Kernel creates user process
   descend &user_dcb

   // Application runs in Domain 3
   // Can only access its assigned memory regions

Summary Table
-------------

.. list-table:: Domain Configuration Summary
   :widths: 15 20 20 20 25
   :header-rows: 1

   * - Domain
     - Memory
     - Interrupt
     - Security
     - Page Table
   * - 0 (Firmware)
     - Full physical
     - Physical IRQ
     - None
     - Physical
   * - 1 (Hypervisor)
     - 16 MB region
     - IRQ pool
     - None
     - INHERIT → Physical
   * - 2 (Guest OS)
     - 256 MB encrypted
     - Virtual IRQ
     - Encrypted
     - Guest VA → IPA → PA
   * - 3 (Application)
     - 1 MB code + heap
     - None (trap to OS)
     - Inherit guest
     - User VA → Guest IPA

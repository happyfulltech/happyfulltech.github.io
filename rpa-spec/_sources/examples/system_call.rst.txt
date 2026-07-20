.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

System Call Example
===================

This chapter demonstrates how an application executes a system call through the ascend primitive.

Scenario
--------

Application (Domain 3) requests file I/O from Guest OS (Domain 2).

Setup
-----

.. code-block:: text

   Domain 3: User Application
   Domain 2: Guest OS Kernel
   Domain 1: Hypervisor (optional interception)
   Domain 0: Firmware

Application wants to read from a file:

.. code-block:: c

   // Application code
   int fd = open("/data/file.txt", O_RDONLY);
   char buffer[1024];
   read(fd, buffer, 1024);

Step 1: Application Prepares Arguments
---------------------------------------

.. code-block:: text

   // Application prepares system call
   R0 = SYS_read        // System call number
   R1 = fd              // File descriptor
   R2 = buffer          // Buffer address
   R3 = 1024            // Count

Step 2: Application Executes ascend
-----------------------------------

.. code-block:: text

   ascend SYS_read

Hardware Actions
----------------

Context Save
~~~~~~~~~~~~

.. code-block:: text

   // Hardware saves current context to Domain 3 DCB
   dcb3.saved_pc = PC + 4
   dcb3.saved_sp = SP
   dcb3.saved_psr = PSR

   // Arguments preserved in registers (ISA-specific)

Domain Switch
~~~~~~~~~~~~~

.. code-block:: text

   // Hardware switches to parent
   current_domain = domain_2

Trap Vector Resolution
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

   // Hardware checks Domain 2 trap_vector
   if dcb2.trap_vector != 0:
       PC = dcb2.trap_vector    // Jump to syscall handler
   else:
       // Propagate to Domain 1 (and so on)

Step 3: Guest OS Handles System Call
-------------------------------------

.. code-block:: text

   // Guest OS syscall handler
   syscall_handler:
       // Save user registers
       push {r4-r11}

       // Dispatch based on R0
       cmp R0, #SYS_read
       beq handle_read
       cmp R0, #SYS_write
       beq handle_write
       ...

   handle_read:
       // Validate fd and buffer address
       // Check buffer is in user's ipa_regions
       // Perform read operation
       // Store result in R0
       // Return to user

Step 4: Guest OS Returns to Application
----------------------------------------

.. code-block:: text

   // Kernel prepares return
   R0 = bytes_read    // Return value

   // Execute return
   return

Hardware Actions for return
---------------------------

.. code-block:: text

   // Hardware restores Domain 3 context
   PC = dcb3.saved_pc
   SP = dcb3.saved_sp
   PSR = dcb3.saved_psr

   // Application continues with return value in R0

Complete Timeline
-----------------

.. code-block:: text

   Time │ Domain │ Action
   ─────┼────────┼──────────────────────────────────
    T0  │   3    │ Application executes ascend R0
    T1  │   HW   │ Save Domain 3 context to DCB3
    T2  │   HW   │ Switch current_domain → 2
    T3  │   HW   │ Load PC = dcb2.trap_vector
    T4  │   2    │ Guest OS syscall handler runs
    T5  │   2    │ Handle read request
    T6  │   2    │ Execute return
    T7  │   HW   │ Restore Domain 3 context
    T8  │   3    │ Application continues

Overhead Analysis
-----------------

.. list-table:: System Call Overhead
   :widths: 25 25 50
   :header-rows: 1

   * - Operation
     - Cycles
     - Notes
   * - Context save
     - 5-10
     - Register store to DCB
   * - Domain switch
     - 1-2
     - Update current_domain pointer
   * - Trap dispatch
     - 2-5
     - Load trap_vector, jump
   * - Total (ascend)
     - 8-17
     - Ascend overhead only

Compare to traditional syscall:

- x86 syscall/sysret: 100-500 cycles
- ARM SVC: 50-200 cycles
- RPA ascend: 8-17 cycles

Hypervisor Interception (Optional)
-----------------------------------

If hypervisor needs to intercept guest system calls:

.. code-block:: text

   // Guest OS sets trap_vector = 0
   dcb2.trap_vector = 0

   // When application ascends:
   // 1. Hardware checks dcb2.trap_vector = 0
   // 2. Propagates to Domain 1
   // 3. Hypervisor's trap handler executes

This enables:

- System call filtering
- Virtual device emulation
- Security policy enforcement

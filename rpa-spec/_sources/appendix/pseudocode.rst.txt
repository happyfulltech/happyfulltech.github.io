.. SPDX-FileCopyrightText: Copyright 2026 Shanghai HappyFull Technologies
.. SPDX-License-Identifier: CC-BY-SA-4.0

Pseudocode
==========

This appendix provides pseudocode for RPA primitive implementations.

Reference Implementation
------------------------

A Python executable specification (pysim) validates the RPA design [PYSIM]_.
The pseudocode examples in this chapter are derived from the pysim implementation.

.. _pysim:

**Repository**: https://github.com/arthurdev000/rpa-pysim

**Version**: 0.7.0

**Test Coverage**: 107 test cases

descend Primitive
-----------------

.. code-block:: text

   procedure descend(dcb_address: physical_address)

   // Step 1: DCB Validation
   size = memory[dcb_address + 0x00]
   if dcb_address % 32 != 0:
       fault(DCB_ALIGNMENT_ERROR)

   if size < 8:
       fault(DCB_SIZE_ERROR)

   // Step 2: Domain ID Assignment
   domain_id = next_domain_id
   next_domain_id = next_domain_id + 1
   memory[dcb_address + 0x04] = domain_id

   // Step 3: Save Parent Context
   current_dcb.saved_pc = PC + 4
   current_dcb.saved_sp = SP
   current_dcb.saved_psr = PSR

   // Step 4: Link Child
   current_dcb.child_block = dcb_address
   memory[dcb_address + 0x08] = current_dcb_physical_address

   // Step 5: Domain Switch
   current_domain = domain_id
   current_dcb_physical_address = dcb_address

   // Step 6: Load Child Entry
   entry_point = memory[dcb_address + saved_lr_offset]
   PC = entry_point

   end procedure

ascend Primitive
----------------

.. code-block:: text

   procedure ascend(service_type: integer)

   // Step 1: Context Save
   current_dcb.saved_pc = PC + 4
   current_dcb.saved_sp = SP
   current_dcb.saved_psr = PSR

   // Step 2: Get Parent DCB
   parent_dcb_address = current_dcb.parent_block
   if parent_dcb_address == 0:
       fault(UNHANDLED_TRAP)
       return

   // Step 3: Trap Vector Resolution with Propagation
   trap_handler = 0
   while trap_handler == 0:
       trap_handler = memory[parent_dcb_address + trap_vector_offset]

       if trap_handler != 0:
           break

       parent_dcb_address = memory[parent_dcb_address + parent_block_offset]
       if parent_dcb_address == 0:
           fault(UNHANDLED_TRAP)
           return

   // Step 4: Domain Switch
   current_dcb_physical_address = parent_dcb_address
   current_domain = memory[parent_dcb_address + domain_id_offset]

   // Step 5: Jump to Handler
   PC = trap_handler

   // Pass service type in argument register
   R0 = service_type

   end procedure

return Primitive
----------------

.. code-block:: text

   procedure return()

   // Step 1: Locate Child DCB
   child_dcb_address = current_dcb.child_block
   if child_dcb_address == 0:
       fault(NO_CHILD_DOMAIN)
       return

   // Step 2: Domain Switch
   current_dcb_physical_address = child_dcb_address
   current_domain = memory[child_dcb_address + domain_id_offset]

   // Step 3: Context Restore
   PC = memory[child_dcb_address + saved_pc_offset]
   SP = memory[child_dcb_address + saved_sp_offset]
   PSR = memory[child_dcb_address + saved_psr_offset]

   // Return value in R0
   // (Handler sets R0 before return)

   // Step 4: Clear Child Link
   current_dcb.child_block = 0

   end procedure

IPA Boundary Check
------------------

.. code-block:: text

   function check_ipa_access(ipa: address, access_type: access_type)
                             returns boolean

   region_count = current_dcb.ipa_region_count

   for i = 0 to region_count - 1:
       region_base = current_dcb.ipa_regions[i].base
       region_limit = current_dcb.ipa_regions[i].limit
       region_attr = current_dcb.ipa_regions[i].attributes

       if ipa >= region_base and ipa <= region_limit:
           // Found matching region, check permissions
           if access_type == READ and region_attr.R == 0:
               return PERMISSION_DENIED
           if access_type == WRITE and region_attr.W == 0:
               return PERMISSION_DENIED
           if access_type == EXECUTE and region_attr.X == 0:
               return PERMISSION_DENIED

           return ALLOWED

   // IPA not in any permitted region
   return IPA_BOUNDARY_VIOLATION

   end function

Security Group Check
--------------------

.. code-block:: text

   function check_security_group_access(source_domain: domain_id,
                                        target_address: address)
                                        returns boolean

   source_group = domain_table[source_domain].security_group
   target_group = get_memory_owner_group(target_address)

   // Check access matrix
   if access_matrix[source_group][target_group] == DENY:
       return SECURITY_GROUP_VIOLATION

   return ALLOWED

   end function

# MPTComm

#### MPTComm.c Overview

The `MPTComm.c` file is responsible for initializing the page directory for each process, managing kernel and user memory mappings, and implementing functions to allocate and free page tables for user processes. The file introduces three core functions:

1. **`pdir_init`**: Initializes the page directory for each process, setting up identity mappings for kernel space and leaving user space unmapped.
2. **`alloc_ptbl`**: Allocates a page for a page table, registers it in the page directory, and clears all entries in the newly mapped page table.
3. **`free_ptbl`**: Frees a page table by removing its mapping in the page directory and releasing the physical memory used by the page table.

Each function serves a specific role in setting up and managing virtual memory mappings for processes.

***

#### Function: `pdir_init`

```c
void pdir_init(unsigned int mbi_addr)
{
    int i, j;

    idptbl_init(mbi_addr);

    for (i = 0; i < NUM_IDS; i++)
    {
        // Kernel address space: set to identity map
        for(j = 0; j < (VM_USERLO_PI >> 10); j++)
        {
            set_pdir_entry_identity(i, j);
        }

        // User address space: set to unmapped
        for(j = (VM_USERLO_PI >> 10); j < (VM_USERHI_PI >> 10); j++)
        {
            rmv_pdir_entry(i, j);
        }

        // Kernel address space (upper range): set to identity map
        for(j = (VM_USERHI_PI >> 10); j < 1024; j++)
        {
            set_pdir_entry_identity(i, j);
        }
    }
}
```

* **Purpose**: Initializes the page directory for each process, setting up kernel space as identity-mapped and user space as unmapped.
* **Mechanism**:
  1. **Call to `idptbl_init`**: This function initializes the identity-mapped page tables for kernel addresses, ensuring that every process shares the same kernel memory mappings.
  2. **Kernel Address Mapping**:
     * Loops over all processes (`i = 0` to `NUM_IDS - 1`).
     * For each process, it maps the lower portion of memory (up to `VM_USERLO_PI >> 10`) to kernel addresses using `set_pdir_entry_identity`. This ensures that the virtual addresses in this range directly map to the same physical addresses, providing direct access to kernel memory.
  3. **User Address Unmapping**:
     * The loop from `VM_USERLO_PI >> 10` to `VM_USERHI_PI >> 10` unmapped the middle section of the page directory, corresponding to user-space memory.
     * For each entry in this range, `rmv_pdir_entry` removes the mapping in the page directory, ensuring that user memory is protected until explicitly mapped.
  4. **Upper Kernel Address Mapping**:
     * The final loop maps the remaining upper addresses (`VM_USERHI_PI >> 10` to 1024) to the kernel using identity mapping.
     * These addresses are also direct mappings to physical memory in kernel space, allowing access to high memory areas.
* **Use Case**: This function is called once during OS initialization to set up each process’s page directory structure, ensuring that kernel memory is accessible and user memory is initially protected.

***

#### Function: `alloc_ptbl`

```c
unsigned int alloc_ptbl(unsigned int proc_index, unsigned int vaddr)
{
    unsigned int addr;
    unsigned int page_index;

    page_index = container_alloc(proc_index);

    if(page_index == 0)
    {
        return 0;
    }

    // Register the new page table in the page directory
    set_pdir_entry_by_va(proc_index, vaddr, page_index);

    // Clear all entries in the new page table
    for(addr = page_index << 12; addr < (page_index + 1) << 12; addr += 4)
    {
        *(unsigned int *)addr &= 0x00000000;
    }

    return page_index;
}
```

* **Purpose**: Allocates a page for a page table, registers it in the page directory at the specified virtual address, and clears all page table entries.
* **Mechanism**:
  1. **Page Allocation**:
     * Calls `container_alloc` to allocate a page for the page table from the process’s memory container.
     * If no physical page is available, the function returns `0`.
  2. **Page Directory Registration**:
     * Maps the newly allocated page in the page directory of `proc_index` using `set_pdir_entry_by_va`.
     * This registers the page as a page table associated with `vaddr`.
  3. **Clearing Page Table Entries**:
     * Loops through each entry in the allocated page table and sets it to `0` to ensure it starts with no mappings.
     * The loop iterates over the page table entries, incrementing `addr` by `4` (the size of each entry), and sets each entry to zero using `*(unsigned int *)addr &= 0x00000000`.
     * Clearing the entries prevents uninitialized access and ensures controlled mapping.
* **Use Case**: Called when a process requires a new page table for a specific virtual address range. This function prepares a fresh page table, registers it in the directory, and ensures it’s clean and ready for mappings.

***

#### Function: `free_ptbl`

```c
void free_ptbl(unsigned int proc_index, unsigned int vaddr)
{
    unsigned int pdir_entry;
    unsigned int page_index;

    pdir_entry = get_pdir_entry_by_va(proc_index, vaddr);
    page_index = pdir_entry >> 12;

    // Remove the page directory entry
    rmv_pdir_entry_by_va(proc_index, vaddr);

    // Free the page used for the page table
    container_free(proc_index, page_index);
}
```

* **Purpose**: Releases a page table by removing its mapping in the page directory and freeing the page used for storing the page table.
* **Mechanism**:
  1. **Retrieve Page Directory Entry**:
     * Calls `get_pdir_entry_by_va` to retrieve the page directory entry for the specified virtual address.
     * The retrieved `pdir_entry` contains the physical address of the page table (shifted by 12 bits).
  2. **Extract Page Index**:
     * Calculates the page index from the page directory entry by shifting `pdir_entry` 12 bits to the right. This yields the page’s base address.
  3. **Remove Page Directory Mapping**:
     * Calls `rmv_pdir_entry_by_va` to remove the page directory entry, effectively unmapping the page table.
  4. **Free Page Table Page**:
     * Uses `container_free` to release the memory allocated for the page table, updating the container’s usage counter.
* **Use Case**: Called when a page table is no longer needed by a process, such as during process termination or memory deallocation. This function ensures that the page table is unmapped, and the physical memory is freed.

***

#### Summary

The `MPTComm.c` file provides core memory management functions crucial for setting up and releasing page tables in mCertiKOS. Here’s a recap of each function’s role:

1. **`pdir_init`**: Initializes page directories for each process, setting up identity-mapped kernel space and protected user space.
2. **`alloc_ptbl`**: Allocates and prepares a new page table for mapping user memory.
3. **`free_ptbl`**: Releases a page table, unmapping it from the page directory and freeing its allocated memory.

These functions form the foundation of page table management in the kernel, facilitating controlled memory mappings and efficient allocation and deallocation of memory resources for process isolation and protection.

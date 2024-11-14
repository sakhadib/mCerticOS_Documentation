# MPTKern

The `MPTKern.c` file provides critical functions for managing kernel-level memory mappings. This layer is responsible for setting up identity mappings for process 0 (the kernel process) and managing mappings between virtual and physical memory for other processes. It includes functions to initialize page directories for the kernel, map pages with specified permissions, and unmap pages as needed.

#### Function: `pdir_init_kern`

```c
void pdir_init_kern(unsigned int mbi_addr)
{
    unsigned int i, j;
    unsigned int pde_index;

    pdir_init(mbi_addr);

    for (pde_index = 0; pde_index < 1024; pde_index++)
    {
        set_pdir_entry_identity(0, pde_index);
    }
}
```

* **Purpose**: Sets up an identity mapping for the entire address space of process 0 (the kernel).
* **Mechanism**:
  1. **Initialization Call**:
     * The function begins by calling `pdir_init(mbi_addr)`, which sets up initial mappings based on the configuration established in the `MPTComm` layer. This prepares the page directory entries for kernel and user space for each process.
  2. **Identity Mapping**:
     * The loop iterates over all 1024 entries in the page directory for process 0 (`pde_index = 0` to `1023`).
     * `set_pdir_entry_identity(0, pde_index)` is called to create an identity mapping for each page directory entry. An identity mapping means that each virtual address directly maps to the same physical address in memory.
     * This ensures that the kernel’s virtual addresses map directly to the same physical addresses, facilitating predictable memory access for kernel operations.
* **Use Case**: This function is called during system initialization to ensure that process 0 (the kernel) has direct access to the entire memory space with an identity map. This setup simplifies kernel access to memory by allowing predictable mappings across the kernel's address space.

***

#### Function: `map_page`

```c
unsigned int map_page(unsigned int proc_index, unsigned int vaddr,
                      unsigned int page_index, unsigned int perm)
{
    unsigned int pde = get_pdir_entry_by_va(proc_index, vaddr);
    unsigned int ptbl;

    if ((pde & PTE_P) == 0)
    {
        ptbl = alloc_ptbl(proc_index, vaddr);
        if (ptbl == 0)
        {
            return MagicNumber;
        }
    }

    set_ptbl_entry_by_va(proc_index, vaddr, page_index, perm);
    pde = get_pdir_entry_by_va(proc_index, vaddr);

    return pde >> 12;
}
```

* **Purpose**: Maps a physical page to a specified virtual address with given permissions for a process. If the page table for this address does not exist, it allocates one first.
* **Mechanism**:
  1. **Check Page Directory Entry**:
     * Calls `get_pdir_entry_by_va(proc_index, vaddr)` to retrieve the page directory entry for the given virtual address.
     * If the entry’s presence bit (`PTE_P`) is not set (`pde & PTE_P == 0`), the page table has not been set up, so the function proceeds to allocate it.
  2. **Allocate Page Table if Necessary**:
     * Calls `alloc_ptbl(proc_index, vaddr)` to allocate a new page table. If this allocation fails (returns `0`), `MagicNumber` is returned to indicate an error.
  3. **Map the Physical Page**:
     * Calls `set_ptbl_entry_by_va(proc_index, vaddr, page_index, perm)` to map the physical page (`page_index`) to the specified virtual address (`vaddr`) with the provided permissions (`perm`).
  4. **Return Page Directory Entry**:
     * Retrieves the page directory entry again using `get_pdir_entry_by_va(proc_index, vaddr)` and shifts it 12 bits to the right to isolate the page index before returning it.
* **Use Case**: This function is essential for setting up virtual-to-physical page mappings for user processes or specific kernel operations. By mapping pages with appropriate permissions, it enables safe memory access for each process, enforcing memory isolation and protection as defined by the `perm` argument.

***

#### Function: `unmap_page`

```c
unsigned int unmap_page(unsigned int proc_index, unsigned int vaddr)
{
    unsigned int pte = get_ptbl_entry_by_va(proc_index, vaddr);
    if ((pte & PTE_P) == 0) //* mapping no longer exists
    {
        return pte;
    }

    rmv_ptbl_entry_by_va(proc_index, vaddr);
    pte = get_ptbl_entry_by_va(proc_index, vaddr);
    return pte;
}
```

* **Purpose**: Removes the mapping for a given virtual address in the specified process. It verifies that a mapping exists before attempting to remove it.
* **Mechanism**:
  1. **Check Page Table Entry**:
     * Calls `get_ptbl_entry_by_va(proc_index, vaddr)` to retrieve the page table entry for the virtual address.
     * If the entry does not have the presence bit (`PTE_P`) set (`pte & PTE_P == 0`), it means there is no valid mapping, so the function returns the page table entry as is.
  2. **Remove Page Table Entry**:
     * Calls `rmv_ptbl_entry_by_va(proc_index, vaddr)` to remove the page table entry for the specified virtual address.
  3. **Return Updated Page Table Entry**:
     * After removing the entry, it retrieves the updated page table entry to verify the change and returns it.
* **Use Case**: This function is used during memory deallocation or when remapping is needed. By safely removing the mapping, it frees up the virtual address for other uses while ensuring that no unintended access occurs to the unmapped page.

***

#### Summary

The `MPTKern.c` file implements essential functions for managing kernel-level memory mappings in mCertiKOS. Here’s a recap of each function:

1. **`pdir_init_kern`**: Initializes the kernel’s page directory by setting up an identity map across the entire address space, ensuring predictable memory access for kernel operations.
2. **`map_page`**: Maps a physical page to a specified virtual address with defined permissions. It allocates a new page table if one does not exist, ensuring the virtual address is fully mapped.
3. **`unmap_page`**: Removes a mapping for a given virtual address, ensuring safe deallocation or reallocation of the virtual address space.

These functions are critical for managing memory mappings at the kernel level, enabling mCertiKOS to provide robust and protected memory access for both kernel and user processes.

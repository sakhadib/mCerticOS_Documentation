# MPTIntro

#### Overview of Virtual Memory Management in `MPTIntro`

The `MPTIntro` layer manages the **page directory** and **page table entries** required for setting up virtual memory for processes in mCertiKOS. Virtual memory in a typical x86 environment involves a **two-level page table structure**:

1. **Page Directory**: Each process has a page directory containing 1024 entries, where each entry points to a page table.
2. **Page Table**: Each page table also contains 1024 entries, with each entry pointing to a physical page in memory.

These structures are represented in `MPTIntro` by:

* **`PDirPool`**: Stores a page directory for each process (total of `NUM_IDS` processes). Each entry in `PDirPool` can reference a page table or hold specific permissions.
* **`IDPTbl`**: A predefined set of page tables that map kernel memory identically across processes, ensuring consistency in kernel address spaces.

With these definitions, `MPTIntro` provides functions that manage page directory entries, set up page tables, and apply permission settings. Let’s go through each function in detail.

***

#### Detailed Explanation of Each Function

**`set_pdir_base`**

```c
void set_pdir_base(unsigned int index)
{
    set_cr3(PDirPool[index]);   
}
```

* **Purpose**: Sets the `CR3` register to point to the page directory for a specific process.
* **Mechanism**:
  * The `CR3` register in x86 architecture holds the address of the current **page directory base**. This base address is used by the MMU (Memory Management Unit) to translate virtual addresses to physical addresses.
  * By calling `set_cr3(PDirPool[index])`, the system sets the page directory base to `PDirPool[index]`, which contains the page directory for the specified process.
* **Code Insight**: `PDirPool[index]` is the base address of the page directory for the process with ID `index`.
* **Use Case**: This function is crucial for **context switching**. When the OS switches from one process to another, it updates `CR3` to point to the new process's page directory, allowing seamless virtual memory handling for each process.

**`get_pdir_entry`**

```c
unsigned int get_pdir_entry(unsigned int proc_index, unsigned int pde_index)
{
    return (unsigned int)PDirPool[proc_index][pde_index];
}
```

* **Purpose**: Retrieves the page directory entry at a specific index within a process’s page directory.
* **Mechanism**:
  * Accesses `PDirPool[proc_index][pde_index]`, which points to the page table (or contains permissions) for the requested index.
  * Page directory entries can hold page tables or address spaces in memory. By retrieving this entry, we can check if a mapping exists for that directory.
* **Code Insight**: This function casts the entry to an `unsigned int` to extract both the page table base address and permission bits.
* **Use Case**: **Memory mapping checks**. This function can verify if a virtual address has an allocated page table and whether it’s accessible.

**`set_pdir_entry`**

```c
void set_pdir_entry(unsigned int proc_index, unsigned int pde_index,
                    unsigned int page_index)
{
    unsigned int value = (page_index << 12) | PT_PERM_PTU;
    PDirPool[proc_index][pde_index] = (unsigned int *)value;
}
```

* **Purpose**: Maps a page directory entry to the physical page indicated by `page_index` with permission bits `PTE_P`, `PTE_W`, and `PTE_U`.
* **Mechanism**:
  * Calculates the **base address** of the page table by shifting `page_index` left by 12 bits (`page_index << 12`). This 12-bit shift aligns the page base address since each page is 4KB (2^12).
  * Sets permission bits:
    * **PTE\_P** (`Present`): Indicates the page is present in memory.
    * **PTE\_W** (`Writable`): Allows write access.
    * **PTE\_U** (`User`): Allows access from user mode.
  * Combines these bits into a single value and assigns it to `PDirPool[proc_index][pde_index]`.
* **Code Insight**: This function creates a page directory entry with both the base address and permissions, allowing safe access to the page.
* **Use Case**: **Process setup**. This function is used to map new page tables or memory pages for a process as it requests additional memory.

**`set_pdir_entry_identity`**

```c
void set_pdir_entry_identity(unsigned int proc_index, unsigned int pde_index)
{
    unsigned int value = (unsigned int)IDPTbl[pde_index];
    value |= PT_PERM_PTU;
    PDirPool[proc_index][pde_index] = (unsigned int *)value;
}
```

* **Purpose**: Maps a page directory entry to an identity page table in `IDPTbl`.
* **Mechanism**:
  * Retrieves `IDPTbl[pde_index]`, which holds an identity-mapped page table base.
  * Sets permissions `PTE_P`, `PTE_W`, and `PTE_U` and assigns this mapped entry to `PDirPool[proc_index][pde_index]`.
* **Code Insight**: The identity-mapping approach ensures consistent kernel memory access across processes.
* **Use Case**: **Kernel memory management**. This function allows kernel regions to be mapped identically across all processes, simplifying kernel operations that require consistent memory addresses.

**`rmv_pdir_entry`**

```c
void rmv_pdir_entry(unsigned int proc_index, unsigned int pde_index)
{
    PDirPool[proc_index][pde_index] = 0;
}
```

* **Purpose**: Removes a page directory entry, setting it to zero.
* **Mechanism**:
  * Sets `PDirPool[proc_index][pde_index]` to `0`, effectively removing the mapping.
* **Code Insight**: By setting the entry to `0`, it invalidates any access via this directory entry, which will raise a page fault if accessed.
* **Use Case**: **Memory deallocation**. When freeing memory, this function clears page directory entries, releasing the associated resources.

**`get_ptbl_entry`**

```c
unsigned int get_ptbl_entry(unsigned int proc_index, unsigned int pde_index,
                            unsigned int pte_index)
{
    unsigned int pte_addr = (unsigned int)PDirPool[proc_index][pde_index];
    pte_addr &= 0xFFFFF000; 
    pte_addr += pte_index << 2; 
    return *(unsigned int *)pte_addr;
}
```

* **Purpose**: Retrieves a page table entry within a specific page directory entry for a process.
* **Mechanism**:
  * `PDirPool[proc_index][pde_index]` gives the page table base address. The `&= 0xFFFFF000` mask clears permissions, isolating the base address.
  * Calculates the specific page table entry address by adding `(pte_index << 2)` (4 bytes per entry).
* **Code Insight**: The function returns the value at the calculated page table entry, which includes the page address and permissions.
* **Use Case**: **Mapping validation**. Verifies if a page table entry points to a valid page and has the expected permissions.

**`set_ptbl_entry`**

```c
void set_ptbl_entry(unsigned int proc_index, unsigned int pde_index,
                    unsigned int pte_index, unsigned int page_index,
                    unsigned int perm)
{
    unsigned int* pte;
    unsigned int pte_addr = (unsigned int)PDirPool[proc_index][pde_index];
    pte_addr &= 0xFFFFF000; 
    pte_addr += pte_index << 2; 

    pte = (unsigned int *)pte_addr; 
    *pte &= 0x00000000;
    *pte = (page_index << 12); 
    *pte |= (perm & 0x00000fff);
}
```

* **Purpose**: Maps a page table entry to a physical page with the specified permissions.
* **Mechanism**:
  * Gets the base address of the page table, clears permission bits, and calculates the address of the page table entry.
  * Clears the entry and assigns `page_index << 12` (base address) plus the `perm` bits.
* **Code Insight**: This maps the entry to a physical page and sets permissions, controlling process access.
* **Use Case**: **Memory allocation**. This function enables new memory allocations by mapping virtual addresses to physical pages.

**`set_ptbl_entry_identity`**

```c
void set_ptbl_entry_identity(unsigned int pde_index, unsigned int pte_index,
                             unsigned int perm)
{
    IDPTbl[pde_index][pte_index] = ((pde_index << 10) + pte_index) << 12;
    IDPTbl[pde_index][pte_index] |= perm;
}
```

* **Purpose**: Sets up an identity mapping in `IDPTbl`.
* **Mechanism**:
  * Maps `pde_index` and `pte_index` directly to the physical address,

applying the identity map.

* Adds `perm` to control access.
* **Code Insight**: This function ensures kernel addresses are consistent across processes.
* **Use Case**: **Kernel consistency**. Essential for kernel space mappings shared across all processes.

**`rmv_ptbl_entry`**

```c
void rmv_ptbl_entry(unsigned int proc_index, unsigned int pde_index,
                    unsigned int pte_index)
{
    unsigned int *pte;
    unsigned int pte_addr = (unsigned int)PDirPool[proc_index][pde_index];
    pte_addr &= 0xFFFFF000; 
    pte_addr += pte_index << 2; 
    pte = (unsigned int *)pte_addr; 
    *pte &= 0x00000000; 
}
```

* **Purpose**: Removes a specific page table entry by setting it to zero.
* **Mechanism**:
  * Identifies the page table entry address and sets it to `0`.
* **Use Case**: Used during **memory release**, clearing individual page mappings as part of deallocation.

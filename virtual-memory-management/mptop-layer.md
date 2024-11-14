# MPTOp Layer

#### Virtual Memory Definitions and Constants

```c
#define PTE_P           0x001                           //* Present.
#define PTE_W           0x002                           //* Write.
#define PTE_U           0x004                           //* User.
#define PTE_G           0x100                           //* Global.
#define PAGESIZE        4096                            //* Page size.
#define VA_PDIR_MASK    0xFFC00000                       //* Mask for bits [31:22] for the page directory index.
#define VA_PTBL_MASK    0x003FF000                       //* Mask for bits [21:12] for the page table index.
#define VM_USERLO       0x40000000                       //* Starting user address.
#define VM_USERHI       0xF0000000                       //* Ending user address.
#define PT_PERM_PWG    (PTE_P | PTE_W | PTE_G)          //* Permission for kernel memory.
#define PT_PERM_PW     (PTE_P | PTE_W)                  //* Permission for the rest of memory.
```

* **Purpose**: Defines bit masks and permission settings for virtual memory management.
* **Mechanism**:
  * `PTE_P`, `PTE_W`, `PTE_U`, `PTE_G` are permission bits indicating if the page is present, writable, usable by user mode, and global, respectively.
  * `PAGESIZE`, `VA_PDIR_MASK`, and `VA_PTBL_MASK` are used to compute page directory and table indices based on virtual addresses.
  * `VM_USERLO` and `VM_USERHI` define the user space memory boundaries.
  * `PT_PERM_PWG` and `PT_PERM_PW` define permission sets for kernel and user memory mappings.

#### Function: `get_ptbl_entry_by_va`

```c
unsigned int get_ptbl_entry_by_va(unsigned int proc_index, unsigned int vaddr)
{
    unsigned int pde_index = (vaddr & VA_PDIR_MASK) >> 22;
    unsigned int pte_index = (vaddr & VA_PTBL_MASK) >> 12;
    unsigned int pde = get_pdir_entry(proc_index, pde_index);
    if ((pde & PTE_P) == 0) { return 0; }
    unsigned int pte = get_ptbl_entry(proc_index, pde_index, pte_index);
    if ((pte & PTE_P) == 0) { return 0; }
    return pte;
}
```

* **Purpose**: Retrieves the page table entry for a given virtual address in a process's page structure.
* **Mechanism**:
  * Extracts the page directory (`pde_index`) and page table indices (`pte_index`) from the virtual address using masks.
  * Validates the presence (`PTE_P`) of the page directory entry and page table entry, returning `0` if not present.

#### Function: `get_pdir_entry_by_va`

```c
unsigned int get_pdir_entry_by_va(unsigned int proc_index, unsigned int vaddr)
{
    unsigned int pde_index = (vaddr & VA_PDIR_MASK) >> 22;
    return get_pdir_entry(proc_index, pde_index);
}
```

* **Purpose**: Fetches the page directory entry corresponding to a given virtual address for a process.
* **Mechanism**:
  * Computes the page directory index from the virtual address and retrieves the corresponding directory entry.

#### Function: `rmv_ptbl_entry_by_va`

```c
void rmv_ptbl_entry_by_va(unsigned int proc_index, unsigned int vaddr)
{
    unsigned int pde_index = (vaddr & VA_PDIR_MASK) >> 22;
    unsigned int pte_index = (vaddr & VA_PTBL_MASK) >> 12;
    unsigned int pde = get_pdir_entry(proc_index, pde_index);
    if ((pde & PTE_P) == 0) { return; }
    rmv_ptbl_entry(proc_index, pde_index, pte_index);
}
```

* **Purpose**: Removes a page table entry for a given virtual address.
* **Mechanism**:
  * Validates the presence of the page directory entry before attempting to remove the page table entry.

#### Function: `rmv_pdir_entry_by_va`

```c
void rmv_pdir_entry_by_va(unsigned int proc_index, unsigned int vaddr)
{
    unsigned int pde_index = (vaddr & VA_PDIR_MASK) >> 22;
    rmv_pdir_entry(proc_index, pde_index);
}
```

* **Purpose**: Removes a page directory entry corresponding to a specific virtual address.
* **Mechanism**:
  * Calculates the page directory index and removes the entry using `rmv_pdir_entry`.

#### Function: `set_ptbl_entry_by_va`

```c
void set_ptbl_entry_by_va(unsigned int proc_index, unsigned int vaddr,
                          unsigned int page_index, unsigned int perm)
{
    unsigned int pde_index = (vaddr & VA_PDIR_MASK) >> 22;
    unsigned int pte_index = (vaddr & VA_PTBL_MASK) >> 12;
    set_ptbl_entry(proc_index, pde_index, pte_index, page_index, perm);
}
```

* **Purpose**: Maps a virtual address to a physical page with specified permissions.
* **Mechanism**:
  * Computes indices and directly maps the virtual address in the page table with the provided permissions.

#### Function: `set_pdir_entry_by_va`

```c
void set_pdir_entry_by_va(unsigned int proc_index, unsigned int vaddr,
                          unsigned int page_index)
{
    unsigned int pde_index = (vaddr & VA_PDIR_MASK) >> 22;
    set_pdir_entry(proc_index, pde_index, page_index);
}
```

* **Purpose**: Registers a mapping in the page directory from a virtual address to a physical page.
* **Mechanism**:
  * Calculates the page directory index and sets the page directory entry to map to the specified physical page.

#### Function: `idptbl_init`

```c
void idptbl_init(unsigned int mbi_addr)
{
    unsigned int addr;
    container_init(mbi_addr);
    for (addr = 0; addr < 0xFFFFF000; addr += PAGESIZE) {
        unsigned int pde_index = (addr & VA_PDIR_MASK) >> 22;
        unsigned int pte_index = (addr & VA_PTBL_MASK) >> 12;
        if (addr < VM_USERLO || addr >= VM_USERHI) {
            set_ptbl_entry_identity(pde_index, pte_index, PT_PERM_PWG);
        } else {
            set_ptbl_entry_identity(pde_index, pte_index, PT_PERM_PW);
        }
    }
}
```

* **Purpose**: Initializes the identity page table for both kernel and user memory spaces.
* **Mechanism**:
  * Iterates over all possible memory addresses, setting up identity mappings with appropriate permissions. This includes different permissions for kernel and user spaces, using global flags for kernel space to ensure they remain constant across all processes.
* **Use Case**: Called during system initialization to set up a consistent and protected memory space across the operating system.

Each of these functions plays a critical role in managing the virtual memory space of each process, ensuring that memory accesses are correctly translated and protected according to the operating system's policies.

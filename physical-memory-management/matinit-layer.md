# MATinit Layer

Here is a detailed explanation of the **MATINIT** layer for your GitBook documentation, covering each important aspect of the code, its logic, and use cases:

***

The **MATINIT** layer is responsible for initializing the physical memory allocation table by detecting the total memory available in the system and categorizing it based on usability and accessibility. This initialization is crucial for setting up safe memory boundaries between kernel and user spaces, ensuring that certain areas of physical memory are restricted or accessible only as required.

### Function: `pmem_init`

The `pmem_init` function is the primary initialization function in the **MATINIT** layer. It performs two main tasks:

1. **Calculate Total Physical Memory Pages**: Determines the total number of physical pages (`NUM_PAGES`) based on the highest available memory address.
2. **Initialize Memory Allocation Table (AT)**: Sets the permission status of each page based on usability and access levels defined in the system.

The following is a breakdown of each part of the function.

***

#### Step 1: Hardware Initialization and Device Setup

Before initializing memory, the `pmem_init` function calls `devinit` to perform lower-level device initialization.

```c
devinit(mbi_addr);
```

The `mbi_addr` (Multiboot Information Address) parameter is passed to `devinit`, which helps initialize the hardware setup and memory information used later in this function.

***

#### Step 2: Determine Total Physical Memory Pages

To manage memory effectively, the kernel needs to know the total amount of available physical memory. `pmem_init` does this by retrieving the highest available address from the system's memory map table, calculating the total physical memory pages (`NUM_PAGES`) from this value.

**Code Example:**

```c
table_nrow = get_size();  // Get the number of rows in the memory map table.

if (table_nrow == 0) 
{
    nps = 0;
}
else
{
    start_addr = get_mms(table_nrow - 1);  // Start address of the last memory range.
    length = get_mml(table_nrow - 1);      // Length of the last memory range.
    highest_addr = start_addr + length - 1;  // Calculate the highest address.
    
    // Calculate total pages by dividing the highest address by page size (4096 bytes).
    nps = start_addr / PAGESIZE + length / PAGESIZE + 
          (start_addr % PAGESIZE + length % PAGESIZE) / PAGESIZE;
}

set_nps(nps);  // Set the number of pages in NUM_PAGES.
```

**Explanation:**

1. **`get_size`**: Returns the number of rows in the memory map table.
2. **Highest Address Calculation**: If rows exist, the function finds the highest address by adding the start address of the last row to its length.
3. **Calculate Total Pages (`nps`)**: The highest address is divided by the page size (`4096` bytes) to get the total number of pages.
4. **Set NUM\_PAGES**: The result (`nps`) is stored as the total number of physical pages available.

**Use Case:**

The total page count (`NUM_PAGES`) is essential for higher layers, like the **MATOP** layer, which needs this information to manage page allocations across the system efficiently.

***

#### Step 3: Initialize Permissions for Different Memory Regions

The **MATINIT** layer sets permissions on pages based on their usability. The memory space is divided into:

* **Kernel-only memory**: Memory accessible only by the kernel.
* **User memory**: Memory accessible to user processes.
* **BIOS-reserved or unusable memory**: Memory marked as unavailable for use by the system.

**Key Memory Boundaries**

```c
#define VM_USERLO    0x40000000  // Lower bound for user-accessible memory.
#define VM_USERHI    0xF0000000  // Upper bound for user-accessible memory.
#define VM_USERLO_PI (VM_USERLO / PAGESIZE)  // Page index for VM_USERLO.
#define VM_USERHI_PI (VM_USERHI / PAGESIZE)  // Page index for VM_USERHI.
```

These constants define the memory range for user-accessible memory. The page indices `VM_USERLO_PI` and `VM_USERHI_PI` help identify the starting and ending pages for user memory.

**Code Example:**

```c
for(i = 0; i < VM_USERLO_PI; i++)
{
    at_set_perm(i, 1);  // Mark pages before user memory as kernel-only.
}

for(i = VM_USERHI_PI; i < nps; i++)
{
    at_set_perm(i, 1);  // Mark pages after user memory as kernel-only.
}

for(i = VM_USERLO_PI; i < VM_USERHI_PI; i++)
{
    at_set_perm(i, 0);  // Mark user memory as accessible.
}
```

**Explanation:**

1. **Kernel-only Pages**: All pages below `VM_USERLO_PI` and above `VM_USERHI_PI` are marked as kernel-only (permission `1`).
2. **User Pages**: Pages between `VM_USERLO_PI` and `VM_USERHI_PI` are set as accessible for user processes.

**Use Case:**

This setup helps the operating system protect kernel memory while allowing user processes to use specific regions. Any attempt by user-level code to access kernel-only pages would be restricted, enforcing memory access control.

***

#### Step 4: Mark Usable and Unusable Memory

The system marks memory pages as either usable or reserved based on the BIOS memory map. The **MATINTRO** layer functions are used here to set permissions on each page.

**Code Example:**

```c
for(i = 0; i < table_nrow; i++)
{
    start_addr = get_mms(i);
    length = get_mml(i);
    perm = is_usable(i);         // Check if the memory range is usable.
    perm = perm == 1 ? 2 : 0;    // Set permission to 2 if usable, 0 if not.
    page_idx = start_addr / PAGESIZE;

    // Adjust starting page if the address isn’t page-aligned.
    if (page_idx * PAGESIZE < start_addr)
    {
        page_idx++;
    }

    // Set permissions for pages within the usable range.
    while ((page_idx + 1) * PAGESIZE <= start_addr + length)
    {
        if (page_idx < VM_USERLO_PI)
        {
            page_idx++;
            continue;
        }
        if (page_idx >= VM_USERHI_PI)
        {
            break;
        }

        at_set_perm(page_idx, perm);
        page_idx++;
    }
}
```

**Explanation:**

1. **Loop Through Memory Map Table**: Each memory range in the table is checked.
2. **Usability Check**: `is_usable(i)` returns `1` if the range is usable and `0` if reserved. The permission is set to `2` for usable ranges and `0` for reserved ones.
3. **Set Permissions for Usable Pages**: For each page in the range, the permission is applied, marking the page as either usable or reserved within the user memory bounds.

**Use Case:**

This approach ensures that only pages marked as usable by the BIOS are allocated to processes, preventing conflicts with hardware-reserved or BIOS-reserved memory areas. This setup provides a reliable memory map, keeping reserved memory safe and inaccessible to regular processes.

***

#### Summary of `pmem_init`

| Code Section              | Description                                                                               |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| `devinit(mbi_addr);`      | Initializes hardware and device information.                                              |
| Calculate `NUM_PAGES`     | Sets the total number of physical pages for the system.                                   |
| Set Kernel and User Pages | Separates memory into kernel-only and user-accessible regions based on predefined limits. |
| Mark Usable Memory        | Marks pages as usable or reserved based on BIOS-provided memory map.                      |

***

#### Practical Applications of MATINIT

The **MATINIT** layer is crucial for setting up a structured memory map for the operating system. Here’s how it’s practically used:

1. **Boot-Time Memory Mapping**: Ensures memory boundaries are respected at boot time, preventing conflicts between kernel, user, and BIOS-reserved areas.
2. **Protection Against Unauthorized Access**: By marking pages with specific permissions, **MATINIT** prevents user-level code from accessing restricted areas, preserving kernel stability.
3. **Foundation for Memory Allocation**: Provides the necessary groundwork for the **MATOP** layer to perform dynamic memory allocation, as it depends on having a well-defined memory map.

The **MATINIT** layer establishes a secure, usable memory layout, ensuring that the kernel and user-space programs have access to only the regions they need, while preserving system integrity and stability.

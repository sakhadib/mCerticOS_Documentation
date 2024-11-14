# MATintro Layer

The **MATINTRO** layer in CertiKOS is the foundational layer of the physical memory management system. This layer establishes the data structures and basic functions necessary to represent and manage the allocation and permission status of each page in physical memory. In a 32-bit system, this is critical for tracking up to 4GB of memory, divided into 4KB pages. The MATINTRO layer provides essential functions to set and retrieve permissions and allocation statuses for these pages, which will be used by the higher layers of the memory management system.

#### Key Data Structures

The **MATINTRO** layer uses the following data structures to manage page-level memory information:

1. **NUM\_PAGES**: This global variable keeps track of the number of physical pages available in the system. It is set during the initialization phase in the **MATINIT** layer.
2. **ATStruct**: This structure represents the status of a single physical page and includes:
   * **perm**: An integer indicating the page's permission status:
     * `0`: Reserved by the BIOS (not available to the kernel).
     * `1`: Restricted to kernel-only access.
     * `>1`: Normal permission, available for general allocation.
   * **allocated**: An integer flag for allocation status:
     * `0`: The page is currently unallocated and available.
     * `>0`: The page is allocated.
3. **AT**: An array of **ATStruct** entries, with a maximum size of (2^{20}) (1 million entries) to support up to 4GB of memory in 4KB pages. Each index in the array corresponds to a page in physical memory, allowing CertiKOS to track the state of each page individually.

#### Functions

The **MATINTRO** layer provides getter and setter functions to interact with the page's permission and allocation flags.

**Getter and Setter for NUM\_PAGES**

*   **`get_nps`**: Returns the current number of physical pages available (`NUM_PAGES`). This function allows other parts of the kernel to check how much physical memory is available.

    ```c
    unsigned int get_nps(void)
    {
        return NUM_PAGES;
    }
    ```
*   **`set_nps`**: Sets the number of physical pages (`NUM_PAGES`) to a specified value. This function is typically called by the initialization function in the MATINIT layer once the system’s total memory capacity is determined.

    ```c
    void set_nps(unsigned int nps)
    {
        NUM_PAGES = nps;
    }
    ```

**Page Permission Functions**

The MATINTRO layer allows checking and setting page permissions to control access levels for each page.

*   **`at_is_norm`**: This function checks if a page has “normal” permission, returning `1` if true and `0` if the page has restricted or BIOS-reserved permissions. The permission status of a page determines whether it is available for allocation to user or kernel processes.

    ```c
    unsigned int at_is_norm(unsigned int page_index)
    {
        return AT[page_index].perm > 1 ? 1 : 0;
    }
    ```
*   **`at_set_perm`**: This function sets the permission level for a specified page and marks it as unallocated. The `perm` parameter defines the new permission level, allowing a page to be designated as BIOS-reserved, kernel-only, or normal.

    ```c
    void at_set_perm(unsigned int page_index, unsigned int perm)
    {
        AT[page_index].perm = perm;
        AT[page_index].allocated = 0;
    }
    ```

**Page Allocation Functions**

These functions manage the allocation status of pages, allowing them to be marked as allocated or unallocated.

*   **`at_is_allocated`**: This function checks whether a specified page is currently allocated. It returns `1` if the page is allocated and `0` if unallocated.

    ```c
    unsigned int at_is_allocated(unsigned int page_index)
    {
        return AT[page_index].allocated ? 1 : 0;
    }
    ```
*   **`at_set_allocated`**: This function updates the allocation status of a page. The `allocated` parameter allows marking the page as either allocated or unallocated based on the value provided.

    ```c
    void at_set_allocated(unsigned int page_index, unsigned int allocated)
    {
        AT[page_index].allocated = allocated;
    }
    ```

#### Summary of Functions

| Function           | Description                                                       |
| ------------------ | ----------------------------------------------------------------- |
| `get_nps`          | Returns the total number of pages available.                      |
| `set_nps`          | Sets the total number of pages available.                         |
| `at_is_norm`       | Checks if a page has normal (available) permission.               |
| `at_set_perm`      | Sets the permission level for a page and marks it as unallocated. |
| `at_is_allocated`  | Checks if a page is allocated.                                    |
| `at_set_allocated` | Sets the allocation status of a page.                             |

#### Usage and Purpose

The **MATINTRO** layer provides the basic infrastructure for physical memory management by recording permissions and allocation flags for each page. This information is crucial for the **MATINIT** layer to initialize the memory layout and for the **MATOP** layer to handle dynamic allocation and deallocation of pages.



#### Use Cases of the MATINTRO Layer

The **MATINTRO** layer provides essential building blocks for physical memory management, supporting key operations within the kernel and offering a range of functionalities that can be used in several scenarios.

**1. Memory Allocation and Deallocation Tracking**

* **Use Case**: When the kernel allocates memory to processes or system components, it needs to know which pages are already in use to avoid overwriting. The **MATINTRO** layer’s `at_is_allocated` and `at_set_allocated` functions allow the kernel to track which pages are allocated, marking them as used or free as needed.
* **Purpose**: Ensuring efficient memory usage by marking pages as allocated only when needed. This also helps to avoid double-allocation, where two processes might otherwise be assigned the same physical memory.

**2. Permission Management and Access Control**

* **Use Case**: Some memory pages need special permissions, such as BIOS-reserved pages or kernel-only pages. By setting permissions at the page level using `at_set_perm` and checking them with `at_is_norm`, the **MATINTRO** layer helps enforce memory access rules. For example, memory reserved by the BIOS for hardware-level functions should not be overwritten by the kernel or user processes.
* **Purpose**: Prevent unauthorized access to restricted memory areas, which could lead to security vulnerabilities, data corruption, or system instability. Permissions also support basic separation of user and kernel space.

**3. Boot-Time Memory Initialization**

* **Use Case**: During system startup, the kernel (through the **MATINIT** layer) must initialize its understanding of available physical memory. This involves identifying which memory is available and marking BIOS-reserved or kernel-reserved pages as restricted. The **MATINTRO** functions are used here to set initial permissions and allocation statuses based on information from the BIOS.
* **Purpose**: Setting a clear and accurate memory map at boot time allows the kernel to correctly allocate memory for system processes and avoid areas reserved by other components or reserved hardware.

**4. System Diagnostics and Debugging**

* **Use Case**: When debugging memory issues or diagnosing system performance problems, it is essential to understand the current allocation and permission status of physical pages. The **MATINTRO** layer can provide real-time information on whether a page is allocated or free, helping identify memory leaks, unauthorized access, or incorrect permissions.
* **Purpose**: Debugging tools or diagnostic routines can use the `at_is_allocated` and `at_is_norm` functions to verify memory usage patterns, ensuring the system behaves as expected and that memory resources are correctly managed.

**5. Resource Optimization for Embedded or Limited-Memory Systems**

* **Use Case**: In systems with limited physical memory, such as embedded devices, efficient memory management is critical. By leveraging the **MATINTRO** layer, the kernel can carefully track and allocate only necessary memory, ensuring optimal use of resources.
* **Purpose**: Avoiding wastage of memory, especially in systems with limited physical resources, can improve overall performance and reliability. The MATINTRO layer provides the flexibility to dynamically manage pages, allowing memory to be reassigned as processes complete or as new ones start.

**6. Layered Memory Abstraction for Kernel Operations**

* **Use Case**: The **MATINTRO** layer abstracts lower-level memory details for higher layers like **MATOP**, enabling those layers to perform page-level memory operations without needing direct hardware access. For instance, the **MATOP** layer can allocate or free pages by simply invoking **MATINTRO**’s allocation functions, rather than manually managing each page's state.
* **Purpose**: Abstraction makes memory management modular and easier to maintain, as each layer relies on a clear set of operations defined by lower layers, reducing code complexity and improving kernel stability.

#### Summary of MATINTRO’s Importance

The **MATINTRO** layer is critical for initializing and managing the system's physical memory. By providing functions to set and retrieve the allocation and permission status of pages, it ensures:

* **Accurate and efficient memory usage** through allocation tracking.
* **Robust security and system stability** by enforcing permission restrictions.
* **Optimized resource usage** in limited-memory environments.
* **Support for debugging and diagnostics**, assisting in memory-related troubleshooting.

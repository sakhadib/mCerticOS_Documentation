# MPTnew

The `MPTNew.c` file includes two main functions:

1. **`alloc_page`**: Allocates a physical page in response to an unmapped virtual address (e.g., a page fault) and maps it with the specified permissions.
2. **`alloc_mem_quota`**: Allocates a memory quota for a child process by splitting the parent process's resources.

These functions enable dynamic memory allocation, supporting both on-demand memory page allocation and controlled distribution of memory resources between parent and child processes.

***

#### Function: `alloc_page`

```c
unsigned int alloc_page(unsigned int proc_index, unsigned int vaddr,
                        unsigned int perm)
{
    unsigned int page_index;
    unsigned int ptbl;

    page_index = container_alloc(proc_index);
    if (page_index == 0)
    {
        return MagicNumber;
    }

    ptbl = map_page(proc_index, vaddr, page_index, perm);

    return ptbl;
}
```

* **Purpose**: Allocates a physical page and maps it to a given virtual address with specified permissions. This function is typically called by the page fault handler when a process accesses an unmapped virtual address.
* **Mechanism**:
  1. **Allocate Physical Page**:
     * Calls `container_alloc(proc_index)` to allocate a physical page from the process’s memory container, storing the result in `page_index`.
     * If `container_alloc` returns `0`, it indicates a failure (no physical page is available), so the function returns `MagicNumber` to signal an error.
  2. **Map the Physical Page**:
     * Calls `map_page(proc_index, vaddr, page_index, perm)` to map the newly allocated physical page (`page_index`) to the specified virtual address (`vaddr`) with the permissions given by `perm`.
     * The `map_page` function registers the page in the page directory and returns the page directory entry.
  3. **Return Page Directory Entry**:
     * Returns the page directory entry (from `map_page`) which points to the newly allocated physical page.
* **Use Case**: This function is essential for **demand paging**. When a process accesses a page that isn’t mapped, causing a page fault, the page fault handler can call `alloc_page` to dynamically allocate and map a new page, allowing the process to continue without interruption. It’s a critical part of supporting dynamic memory allocation and expanding process address spaces as needed.

***

#### Function: `alloc_mem_quota`

```c
unsigned int alloc_mem_quota(unsigned int id, unsigned int quota)
{
    unsigned int child;
    child = container_split(id, quota);
    return child;
}
```

* **Purpose**: Allocates a memory quota for a child process by splitting the parent process’s quota, creating a new container for the child with the specified amount of memory.
* **Mechanism**:
  1. **Create Child Container**:
     * Calls `container_split(id, quota)`, which designates a portion of the parent’s memory quota (identified by `id`) to the new child process.
     * `container_split` returns the ID of the newly created child process container, which is stored in `child`.
  2. **Return Child ID**:
     * Returns the `child` ID, which identifies the container with the allocated memory quota.
* **Use Case**: This function is used during **process creation** to allocate memory resources for new processes. When a parent process spawns a child, it uses `alloc_mem_quota` to reserve a portion of its own memory quota for the child. This allows the OS to manage memory resources hierarchically, with each process controlling how much memory is available to its children.

***

#### Summary

The `MPTNew.c` file provides two essential functions for dynamic memory management in mCertiKOS:

1. **`alloc_page`**: Supports demand paging by allocating and mapping physical memory in response to page faults.
2. **`alloc_mem_quota`**: Enables hierarchical memory allocation by allowing parent processes to allocate memory quotas for child processes.

Together, these functions provide flexible and controlled memory allocation, ensuring efficient resource management and supporting dynamic growth of process memory spaces as needed.

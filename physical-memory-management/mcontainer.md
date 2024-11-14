# MContainer

The `MContainer` layer in mCertiKOS provides essential container-based memory management functionality for processes in a simulated operating system environment. It introduces a container structure (`SContainer`) that manages each process's memory quota, usage, and parent-child relationships. Containers allow the system to allocate and track memory resources efficiently, supporting multiple processes while preventing any single process from monopolizing system memory.

#### Structure of `SContainer`

Each container (`SContainer`) stores crucial data for a process:

* **`quota`**: The maximum memory the process is allowed to use.
* **`usage`**: The current memory usage of the process.
* **`parent`**: The ID of the parent process.
* **`nchildren`**: Number of child processes spawned by this container.
* **`used`**: Indicates if the container is currently active.

This structure allows hierarchical memory allocation, where each process is assigned memory by its parent, leading to a controlled resource distribution.

#### Core Functions in the `MContainer` Layer

1. **`container_init`**:
   * **Purpose**: Initializes the root container (process with ID 0) to control the entire available memory.
   * **Mechanism**:
     * Uses the physical memory table to calculate the total available memory pages that aren’t allocated (`real_quota`).
     * Assigns the calculated quota to the root process and initializes its usage, children, and status.
   * **Coding Techniques**:
     * Loops through available physical memory pages, using `at_is_norm` and `at_is_allocated` to filter pages that are normal and free.
   * **Use Case**: Essential during OS startup to set up the main memory container for the root process.
2. **`container_get_parent`**:
   * **Purpose**: Retrieves the parent ID of a specified process.
   * **Mechanism**:
     * Accesses the `parent` field of the container associated with the given process ID.
   * **Use Case**: Used by the system to maintain and traverse the process hierarchy.
3. **`container_get_nchildren`**:
   * **Purpose**: Returns the number of children spawned by a specified process.
   * **Mechanism**:
     * Simply retrieves the `nchildren` value from the container structure.
   * **Use Case**: Useful for memory management when determining if a process has dependent processes before deallocation.
4. **`container_get_quota`**:
   * **Purpose**: Retrieves the maximum allowable memory quota for a specified process.
   * **Mechanism**:
     * Directly accesses the `quota` field in the container.
   * **Use Case**: Often called before memory allocation to check if the process has sufficient memory quota.
5. **`container_get_usage`**:
   * **Purpose**: Returns the current memory usage for a specified process.
   * **Mechanism**:
     * Directly accesses the `usage` field in the container.
   * **Use Case**: Useful for tracking and debugging memory consumption by processes.
6. **`container_can_consume`**:
   * **Purpose**: Determines if a process can allocate additional memory pages without exceeding its quota.
   * **Mechanism**:
     * Checks if `quota - usage` is greater than or equal to the requested pages.
   * **Coding Techniques**:
     * Uses simple arithmetic for a quick validation of memory consumption limits.
   * **Use Case**: Frequently used before `container_alloc` to ensure a safe allocation.
7. **`container_split`**:
   * **Purpose**: Allocates memory quota for a new child process under a parent process.
   * **Mechanism**:
     * Computes the child’s container index using a formula based on `id`, `MAX_CHILDREN`, and `nchildren`.
     * Updates the parent’s `usage` and `nchildren` fields and initializes the child’s container fields.
   * **Coding Techniques**:
     * Implements hierarchical memory management, where a child process inherits a portion of its parent’s quota.
   * **Use Case**: Called when spawning new processes that require a portion of the parent’s resources.
8. **`container_alloc`**:
   * **Purpose**: Allocates a single page of memory to the specified process if it doesn’t exceed the quota.
   * **Mechanism**:
     * Calls `palloc` to obtain a physical page.
     * Updates the process’s `usage` upon successful allocation.
   * **Use Case**: Crucial in memory allocation, ensuring that each process only consumes its allowable memory.
9. **`container_free`**:
   * **Purpose**: Frees a physical page for a specified process, reducing its memory usage by one page.
   * **Mechanism**:
     * Calls `pfree` to release the memory and decrements the `usage`.
   * **Use Case**: Called during process termination or when memory is explicitly freed.

#### Summary

The `MContainer` layer forms the foundation of memory management in mCertiKOS, implementing container-based memory tracking and allocation. Each function encapsulates specific responsibilities in memory tracking and resource distribution, supporting hierarchical process management and safe memory allocation.

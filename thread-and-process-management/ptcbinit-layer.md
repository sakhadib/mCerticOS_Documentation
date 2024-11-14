# PTCBInit Layer

#### Function: `tcb_init`

```c
void tcb_init(unsigned int mbi_addr)
{
    paging_init(mbi_addr);

    int id;
    for (id = 0; id < NUM_IDS; id++) {
        tcb_init_at_id(id);
    }
}
```

* **Purpose**: Initializes the TCB for all threads (up to `NUM_IDS`) by setting their initial state and properties. Each TCB entry starts in the `TSTATE_DEAD` state, indicating that the thread is not yet active.
* **Mechanism**:
  1. **Paging Initialization**:
     * Calls `paging_init(mbi_addr)`, which initializes the paging system using the `mbi_addr` (Multiboot Information Address) provided. This ensures that memory management is correctly set up before thread control blocks are initialized.
  2. **Initialize Each TCB**:
     * Loops over all possible thread IDs (`0` to `NUM_IDS - 1`).
     * Calls `tcb_init_at_id(id)` for each ID, which sets the state of the TCB at that ID to `TSTATE_DEAD` and initializes the TCB fields (e.g., setting indices to `NUM_IDS` to represent NULL pointers).
     * By marking each TCB as `TSTATE_DEAD`, this setup allows the system to differentiate between active and inactive threads, readying the TCB for use when threads are created.
* **Use Case**:
  * Called during system startup or thread management initialization to ensure all TCBs are correctly set to an inactive state. This initialization ensures that no threads are marked as active, preventing accidental use of uninitialized thread entries.

#### Summary

The `tcb_init` function sets up the initial state for all thread control blocks, marking them as dead and unallocated. This step is crucial for managing process/thread lifecycle in mCertiKOS, as it establishes a clean starting state for thread tracking and management. By initializing each TCB entry to a known state, the kernel can safely handle thread creation and termination.

# Pthread Layer

`pthread.c` is responsible for thread initialization, spawning new threads, and yielding control between threads in the mCertiKOS environment. This file sets up a basic thread management system, allowing threads to be created, queued, and scheduled using a round-robin approach.

This file defines three core functions:

1. **`thread_init`**: Initializes the thread subsystem.
2. **`thread_spawn`**: Creates a new thread, setting up its initial context and placing it in the ready queue.
3. **`thread_yield`**: Switches the CPU’s control from the currently running thread to the next ready thread in the queue.

Together, these functions form the foundation for process scheduling in mCertiKOS by managing thread lifecycles and facilitating context switching.

***

#### Function: `thread_init`

```c
void thread_init(unsigned int mbi_addr)
{
    tqueue_init(mbi_addr);
    set_curid(0);
    tcb_set_state(0, TSTATE_RUN);
}
```

* **Purpose**: Initializes the thread management system by setting up the queue system, setting the first thread as the current thread, and marking it as running.
* **Mechanism**:
  1. **Initialize Thread Queues**:
     * Calls `tqueue_init(mbi_addr)` to initialize all thread queues, setting them to an empty state and setting up Thread Control Blocks (TCBs).
  2. **Set Initial Thread**:
     * Calls `set_curid(0)` to set the first thread (ID `0`) as the current thread.
  3. **Set Initial State**:
     * Marks the initial thread (ID `0`) as `TSTATE_RUN`, indicating that this thread is running.
* **Use Case**: This function is called during system startup to initialize the first thread and prepare the thread management system for operation.

***

#### Function: `thread_spawn`

```c
unsigned int thread_spawn(void *entry, unsigned int id, unsigned int quota)
{
    int child = kctx_new(entry, id, quota);
    tcb_set_state(child, TSTATE_READY);
    tqueue_enqueue(NUM_IDS, child);
    return child;
}
```

* **Purpose**: Creates a new thread by setting up a context for it, marking it as ready, and adding it to the ready queue. The function returns the ID of the newly created child thread.
* **Mechanism**:
  1. **Allocate a New Thread Context**:
     * Calls `kctx_new(entry, id, quota)` to allocate a new kernel context for the child thread. `kctx_new` sets the entry point and stack pointer for the new thread based on the `entry` function, `id`, and memory quota.
     * The function returns the ID of the new child thread if successful, which is stored in `child`.
  2. **Set Child Thread State**:
     * Sets the child’s state to `TSTATE_READY` using `tcb_set_state(child, TSTATE_READY)`. This marks the thread as ready to run.
  3. **Add Child to Ready Queue**:
     * Calls `tqueue_enqueue(NUM_IDS, child)` to add the child thread to the ready queue (identified by `NUM_IDS`).
  4. **Return Child ID**:
     * Returns the `child` thread ID, allowing the system to track the new thread.
* **Use Case**: This function is used whenever a new thread needs to be created, such as in response to a `fork` or `spawn` operation. By setting the thread’s initial context, state, and queue placement, this function prepares it for execution.

***

#### Function: `thread_yield`

```c
void thread_yield(void)
{
    unsigned int curid = get_curid();
    unsigned int next_ready = tqueue_dequeue(NUM_IDS);

    if (next_ready == NUM_IDS)
    {
        //* No other thread to run
        return;
    }

    tcb_set_state(curid, TSTATE_READY);                     //* Set the currently running thread state as ready
    tqueue_enqueue(NUM_IDS, curid);                         //* Push it back to the ready queue
    tcb_set_state(next_ready, TSTATE_RUN);                  //* Set the state of the popped thread as running
    set_curid(next_ready);                                  //* Set the current thread id
    kctx_switch(curid, next_ready);                         //* Switch to the new kernel context
}
```

* **Purpose**: Switches control from the currently running thread to the next ready thread in the queue, updating their states and performing a context switch if another thread is available.
* **Mechanism**:
  1. **Identify Current Thread**:
     * Calls `get_curid()` to retrieve the ID of the currently running thread, storing it in `curid`.
  2. **Get Next Ready Thread**:
     * Calls `tqueue_dequeue(NUM_IDS)` to dequeue the next ready thread from the ready queue. If no other thread is ready (i.e., `next_ready == NUM_IDS`), the function returns immediately without yielding.
  3. **Set Current Thread to Ready**:
     * Sets the state of the current thread (`curid`) to `TSTATE_READY` using `tcb_set_state(curid, TSTATE_READY)`.
     * Adds the current thread back to the ready queue with `tqueue_enqueue(NUM_IDS, curid)`.
  4. **Switch to Next Ready Thread**:
     * Sets the state of `next_ready` to `TSTATE_RUN`, marking it as the running thread.
     * Updates the current thread ID to `next_ready` using `set_curid(next_ready)`.
     * Calls `kctx_switch(curid, next_ready)` to perform the actual context switch, transitioning CPU control to the `next_ready` thread.
* **Use Case**: This function is used to yield the CPU from one thread to another, allowing a round-robin scheduling approach. It’s typically called when a thread finishes its time slice, voluntarily yields control, or another scheduling event occurs.

***

#### Summary

The `pthread.c` file provides core functionality for managing threads in mCertiKOS, enabling the creation, initialization, and context switching of threads. Here’s a summary of each function:

1. **`thread_init`**: Initializes the thread subsystem and sets the first thread to a running state.
2. **`thread_spawn`**: Allocates a new thread, sets its initial context and state, and places it in the ready queue.
3. **`thread_yield`**: Switches from the current thread to the next ready thread, facilitating round-robin scheduling and controlled execution.

These functions allow mCertiKOS to support multiple threads, enabling effective process management and scheduling.

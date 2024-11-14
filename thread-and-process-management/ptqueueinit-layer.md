# PTQueueInit Layer

The `PTQueueInit.c` file manages thread queues in mCertiKOS. The queues enable controlled scheduling and management of thread states. mCertiKOS employs a doubly linked list structure for thread queues, with each thread control block (TCB) linked to the next and previous threads in the queue. The primary functions in this file are:

1. **`tqueue_init`**: Initializes all thread queues.
2. **`tqueue_enqueue`**: Adds a thread to the end of a specified queue.
3. **`tqueue_dequeue`**: Removes and returns the first thread from a specified queue.
4. **`tqueue_remove`**: Removes a specified thread from any position in the queue.

These functions are essential for managing threads as they enter, wait, or exit scheduling queues.

***

#### Function: `tqueue_init`

```c
void tqueue_init(unsigned int mbi_addr)
{
    tcb_init(mbi_addr);

    unsigned int id;
    for (id = 0; id < NUM_IDS; id++) {
        //* NumIds + 1 queue in total
        tqueue_init_at_id(id);
    }
}
```

* **Purpose**: Initializes each thread queue, setting up a clear starting state where all queues are empty. It first initializes TCBs, then initializes each thread queue using `tqueue_init_at_id`.
* **Mechanism**:
  1. **Initialize TCBs**:
     * Calls `tcb_init(mbi_addr)` to initialize all thread control blocks with `TSTATE_DEAD` state and set up initial properties for each TCB.
  2. **Initialize Each Queue**:
     * Loops through each queue ID (`0` to `NUM_IDS - 1`) and calls `tqueue_init_at_id(id)` to set both `head` and `tail` of the queue to `NUM_IDS`, indicating that the queue is empty.
* **Use Case**: Called at system startup to set up a clean, empty state for all thread queues. This step is essential for ensuring proper management of thread queues when the system begins to operate.

***

#### Function: `tqueue_enqueue`

```c
void tqueue_enqueue(unsigned int chid, unsigned int pid)
{
    unsigned int tail_pid;

    tail_pid = tqueue_get_tail(chid);

    if (tail_pid != NUM_IDS) 
    {
        //* queue is not empty, link the current tail to pid
        tcb_set_next(tail_pid, pid);
    } 
    else 
    {
        //* queue is empty, set head to pid
        tqueue_set_head(chid, pid);
    }

    //* link pid to the previous tail and set as the new tail
    tcb_set_prev(pid, tail_pid);
    tqueue_set_tail(chid, pid);
}
```

* **Purpose**: Adds a thread (TCB) with ID `pid` to the end of the specified queue (`chid`). This function ensures that each thread enters the queue in the correct position and updates the queue’s tail pointer to maintain the linked list structure.
* **Mechanism**:
  1. **Get the Current Tail**:
     * Calls `tqueue_get_tail(chid)` to retrieve the current tail of the queue.
  2. **Check if Queue is Empty**:
     * If `tail_pid` is `NUM_IDS`, the queue is empty. In this case, the new thread (`pid`) becomes the `head` of the queue, so `tqueue_set_head(chid, pid)` is called to set `pid` as the head.
     * If the queue is not empty, it links the current tail to the new thread by calling `tcb_set_next(tail_pid, pid)`.
  3. **Update the Tail**:
     * `tcb_set_prev(pid, tail_pid)` sets the previous pointer of the new thread (`pid`) to the former tail.
     * `tqueue_set_tail(chid, pid)` sets the new thread as the tail of the queue, completing the enqueue operation.
* **Use Case**: Used whenever a new thread becomes ready for scheduling or needs to be added to any queue (such as the ready queue). This function allows the system to maintain a linked list of threads waiting in each queue.

***

#### Function: `tqueue_dequeue`

```c
unsigned int tqueue_dequeue(unsigned int chid)
{
    unsigned int head_id, head_next_id;

    head_id = tqueue_get_head(chid);

    if (head_id == NUM_IDS) 
    {
        //* queue is empty
        return NUM_IDS;
    }

    head_next_id = tcb_get_next(head_id);

    if (head_next_id != NUM_IDS)
    {
        //* head has a next element, unlink the previous pointer
        tcb_set_prev(head_next_id, NUM_IDS);
    }
    else
    {
        //* no next element, set tail to NUM_IDS (empty queue)
        tqueue_set_tail(chid, NUM_IDS);
    }

    tcb_set_next(head_id, NUM_IDS);
    tqueue_set_head(chid, head_next_id);

    return head_id;
}
```

* **Purpose**: Removes the thread at the head of the specified queue (`chid`) and returns its ID. This operation supports removing the next thread that is ready to run or has completed its task.
* **Mechanism**:
  1. **Get the Head of the Queue**:
     * Calls `tqueue_get_head(chid)` to retrieve the ID of the current head of the queue.
     * If the queue is empty (i.e., `head_id == NUM_IDS`), it returns `NUM_IDS`, indicating no thread to dequeue.
  2. **Update the Queue Head**:
     * Retrieves the ID of the thread following the head by calling `tcb_get_next(head_id)`.
     * If there is a next thread (`head_next_id != NUM_IDS`), it sets its `prev` pointer to `NUM_IDS`, effectively removing the head.
     * If there is no next thread, it sets the queue’s tail to `NUM_IDS`, indicating an empty queue.
  3. **Detach Head and Update**:
     * Sets the `next` pointer of the removed head to `NUM_IDS` to clear its link.
     * Updates the queue’s head to `head_next_id`, completing the dequeue process.
* **Use Case**: Used to remove and return the next thread ready to run, typically by the scheduler when selecting the next thread to execute. This function manages the linked list structure and ensures a clean head removal.

***

#### Function: `tqueue_remove`

```c
void tqueue_remove(unsigned int chid, unsigned int pid)
{
    unsigned int head_pid, tail_pid, prev_pid, next_pid;

    head_pid = tqueue_get_head(chid);
    tail_pid = tqueue_get_tail(chid);
    prev_pid = tcb_get_prev(pid);
    next_pid = tcb_get_next(pid);

    if (head_pid == pid)
    {
        //* removing the head, move head to next
        tqueue_set_head(chid, next_pid);
    }

    if (tail_pid == pid)
    {
        //* removing the tail, move tail to prev
        tqueue_set_tail(chid, prev_pid);
    }

    if (prev_pid != NUM_IDS)
    {
        //* unlink previous node from pid
        tcb_set_next(prev_pid, next_pid);
    }

    if (next_pid != NUM_IDS)
    {
        //* unlink next node from pid
        tcb_set_prev(next_pid, prev_pid);
    }

    tcb_set_prev(pid, NUM_IDS);
    tcb_set_next(pid, NUM_IDS);
}
```

* **Purpose**: Removes a specified thread (`pid`) from any position in the specified queue (`chid`). This function handles all possible cases: removing from the head, tail, or a middle position within the queue.
* **Mechanism**:
  1. **Identify Queue Position**:
     * Retrieves `head_pid` and `tail_pid` to identify if the thread to be removed is at the head, tail, or a middle position.
     * Retrieves `prev_pid` and `next_pid` pointers for the thread being removed to manage its links.
  2. **Remove from Head or Tail**:
     * If `pid` is the head, `tqueue_set_head(chid, next_pid)` updates the queue’s head to the next thread.
     * If `pid` is the tail, `tqueue_set_tail(chid, prev_pid)` updates the queue’s tail to the previous thread.
  3. **Update Adjacent Links**:
     * If `pid` has a previous thread (`prev_pid != NUM_IDS`), it updates `prev_pid` to link directly to `next_pid`, bypassing `pid`.
     * If `pid` has a next thread (`next_pid != NUM_IDS`), it updates `next_pid` to link directly to `prev_pid`, bypassing `pid`.
  4. **Clear Links of Removed Thread**:
     * Sets `tcb_set_prev(pid, NUM_IDS)` and `tcb_set_next(pid, NUM_IDS)`, effectively detaching the removed thread from the queue.

# PTQueueIntro Layer

The `MPTQueueIntro.c` file introduces the concept of **thread queues** in mCertiKOS. These queues manage threads that are either waiting on another thread or are ready to be scheduled. The code defines a queue structure and includes functions for initializing and accessing the queue’s head and tail.

#### Structure: `TQueue`

```c
struct TQueue {
    unsigned int head;
    unsigned int tail;
};
```

* **Purpose**: Represents a thread queue in mCertiKOS. Each queue keeps track of:
  * **`head`**: The index of the first thread in the queue.
  * **`tail`**: The index of the last thread in the queue.
* **Design**: The queue structure is simplified by storing only the head and tail, as the actual linking is managed by the thread control block (TCB), which uses a doubly linked list.

#### Array: `TQueuePool`

```c
struct TQueue TQueuePool[NUM_IDS + 1];
```

* **Purpose**: Contains all thread queues required by mCertiKOS.
* **Design**:
  * The first `NUM_IDS` queues are **sleep queues** for each thread, allowing threads to wait on other threads. Although these sleep queues are not needed in this lab, they are crucial for implementing inter-process communication (IPC).
  * The last queue (`NUM_IDS` index) is the **ready queue**. Threads that are ready to run are added to this queue and scheduled in a round-robin fashion.

#### Functions for Queue Management

**`tqueue_get_head`**

```c
unsigned int tqueue_get_head(unsigned int chid)
{
    return TQueuePool[chid].head;
}
```

* **Purpose**: Returns the head (first thread) of the specified queue.
* **Mechanism**: Retrieves `TQueuePool[chid].head` to identify the first thread in the queue.
* **Use Case**: Used by the scheduler or queue management routines to access the starting thread in a specific queue.

**`tqueue_set_head`**

```c
void tqueue_set_head(unsigned int chid, unsigned int head)
{
    TQueuePool[chid].head = head;
}
```

* **Purpose**: Sets the head of the specified queue to a given thread ID.
* **Mechanism**: Updates `TQueuePool[chid].head` with the `head` parameter to mark the new beginning of the queue.
* **Use Case**: Called when modifying the head of a queue, such as when a thread is added or removed.

**`tqueue_get_tail`**

```c
unsigned int tqueue_get_tail(unsigned int chid)
{
    return TQueuePool[chid].tail;
}
```

* **Purpose**: Returns the tail (last thread) of the specified queue.
* **Mechanism**: Accesses `TQueuePool[chid].tail` to retrieve the ID of the last thread in the queue.
* **Use Case**: Useful when adding threads to the end of a queue or determining the queue’s last element.

**`tqueue_set_tail`**

```c
void tqueue_set_tail(unsigned int chid, unsigned int tail)
{
    TQueuePool[chid].tail = tail;
}
```

* **Purpose**: Sets the tail of the specified queue to a given thread ID.
* **Mechanism**: Updates `TQueuePool[chid].tail` with the `tail` parameter, marking the new end of the queue.
* **Use Case**: Called when modifying the queue’s tail, such as when enqueuing a new thread.

**`tqueue_init_at_id`**

```c
void tqueue_init_at_id(unsigned int chid)
{
    TQueuePool[chid].head = NUM_IDS;
    TQueuePool[chid].tail = NUM_IDS;
}
```

* **Purpose**: Initializes the queue at the specified index by setting both `head` and `tail` to `NUM_IDS`, which represents `NULL`.
* **Mechanism**: Sets `TQueuePool[chid].head` and `TQueuePool[chid].tail` to `NUM_IDS`, indicating that the queue is empty.
* **Use Case**: Called during system initialization to ensure that all queues are properly set to an empty state.

#### Summary

This code establishes the basic structure and access functions for managing thread queues in mCertiKOS. The `TQueue` structure holds only the head and tail of each queue, leveraging the TCB’s doubly linked list implementation to manage queue entries. These functions facilitate queue management and will be essential in subsequent labs when working with scheduling and inter-process communication.

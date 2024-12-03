There are various implementations of **BlockingQueue** like **ArrayBlockingQueue**, **LinkedBlockingQueue**, **SynchronousQueue**, **PriorityBlockingQueue**.

## _ArrayBlockingQueue_
An _ArrayBlockingqueue_ is a bounded queue. It internally uses an array. We can mention the size of the array when creating its instance.

```
int INIT_CAPACITY = 10;

BlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(INIT_CAPACITY, true);
```

If we insert elements in the queue beyond the defined capacity and the queue is full, then the add operation throws _IllegalStateException_. Also, if we set the initial size to less than _1_, we’ll get _IllegalArgumentException_.

**Here the second parameter represents the fairness policy.** We can optionally set the fairness policy to preserve the order of blocked producer and consumer threads. It allows queue access for blocked threads in FIFO order. Thus, the thread that went into the waiting state first will first get the chance to access the queue. This helps to avoid thread starvation.

## _LinkedBlockingQueue_
A _LinkedBlockingQueue_ is an optionally bounded implementation of _BlockingQueue_. It’s backed by linked nodes.

**We can also specify the capacity while creating its instance. If not specified, then _Integer.MAX_VALUE_ is set as capacity.**
```
BlockingQueue<String> linkedBlockingQueue = new LinkedBlockingQueue<>();
```


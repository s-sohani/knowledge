There are various implementations of **BlockingQueue** like **ArrayBlockingQueue**, **LinkedBlockingQueue**, **SynchronousQueue**, **PriorityBlockingQueue**.

## _ArrayBlockingQueue_
An _ArrayBlockingqueue_ is a bounded queue. It internally uses an array. We can mention the size of the array when creating its instance.

```
int INIT_CAPACITY = 10;

BlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(INIT_CAPACITY, true);
```
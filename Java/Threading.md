**Java thread pool** manages the pool of worker threads. It contains a queue that keeps tasks waiting to get executed. We can use `ThreadPoolExecutor` to create thread pool in Java. Java thread pool manages the collection of Runnable threads. The worker threads execute Runnable threads from the queue. **java.util.concurrent.Executors** provide factory and support methods for **java.util.concurrent.Executor** interface to create the thread pool in java.

```
package com.journaldev.threadpool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SimpleThreadPool {

    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++) {
            Runnable worker = new WorkerThread("" + i);
            executor.execute(worker);
          }
        executor.shutdown();
        while (!executor.isTerminated()) {
        }
        System.out.println("Finished all threads");
    }
}
```

**Executors** class provide simple implementation of **ExecutorService** using **ThreadPoolExecutor** but ThreadPoolExecutor provides much more feature than that. We can specify the number of threads that will be alive when we create ThreadPoolExecutor instance and we can limit the size of thread pool and create our own **RejectedExecutionHandler** implementation to handle the jobs that can’t fit in the worker queue.


```
package com.journaldev.threadpool;

import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;

public class RejectedExecutionHandlerImpl implements RejectedExecutionHandler {

    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println(r.toString() + " is rejected");
    }

}
```

```
import java.util.concurrent.ThreadPoolExecutor;

public class MyMonitorThread implements Runnable
{
    private ThreadPoolExecutor executor;
    private int seconds;
    private boolean run=true;

    public MyMonitorThread(ThreadPoolExecutor executor, int delay)
    {
        this.executor = executor;
        this.seconds=delay;
    }
    public void shutdown(){
        this.run=false;
    }
    @Override
    public void run()
    {
        while(run){
                System.out.println(
                    String.format("[monitor] [%d/%d] Active: %d, Completed: %d, Task: %d, isShutdown: %s, isTerminated: %s",
                        this.executor.getPoolSize(),
                        this.executor.getCorePoolSize(),
                        this.executor.getActiveCount(),
                        this.executor.getCompletedTaskCount(),
                        this.executor.getTaskCount(),
                        this.executor.isShutdown(),
                        this.executor.isTerminated()));
                try {
                    Thread.sleep(seconds*1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        }
            
    }
}
```


```
package com.journaldev.threadpool;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class WorkerPool {

    public static void main(String args[]) throws InterruptedException{
        //RejectedExecutionHandler implementation
        RejectedExecutionHandlerImpl rejectionHandler = new RejectedExecutionHandlerImpl();
        //Get the ThreadFactory implementation to use
        ThreadFactory threadFactory = Executors.defaultThreadFactory();
        //creating the ThreadPoolExecutor
        ThreadPoolExecutor executorPool = new ThreadPoolExecutor(2, 4, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2), threadFactory, rejectionHandler);
        //start the monitoring thread
        MyMonitorThread monitor = new MyMonitorThread(executorPool, 3);
        Thread monitorThread = new Thread(monitor);
        monitorThread.start();
        //submit work to the thread pool
        for(int i=0; i<10; i++){
            executorPool.execute(new WorkerThread("cmd"+i));
        }
        
        Thread.sleep(30000);
        //shut down the pool
        executorPool.shutdown();
        //shut down the monitor thread
        Thread.sleep(5000);
        monitor.shutdown();
        
    }
}
```


## CompletableFuture
Asynchronous computation is difficult to reason about. Usually, we want to think of any computation as a series of steps, but in the case of asynchronous computation, **actions represented as callbacks tend to be either scattered across the code or deeply nested inside each other**. Things get even worse when we need to handle errors that might occur during one of the steps.

The _Future_ interface was added in Java 5 to serve as a result of an asynchronous computation, but it did not have any methods to combine these computations or handle possible errors.

**Java 8 introduced the _CompletableFuture_ class.** Along with the _Future_ interface, it also implemented the _CompletionStage_ interface. This interface defines the contract for an asynchronous computation step that we can combine with other steps.

_CompletableFuture_ is at the same time, a building block and a framework, with **about 50 different methods for composing, combining, and executing asynchronous computation steps and handling errors**.

## **Using _CompletableFuture_ as a Simple _Future_

First, the _CompletableFuture_ class implements the _Future_ interface so that we can **use it as a _Future_ implementation but with additional completion logic**.

For example, we can create an instance of this class with a no-arg constructor to represent some future result, hand it out to the consumers, and complete it at some time in the future using the _complete_ method. The consumers may use the _get_ method to block the current thread until this result is provided.

In the example below, we have a method that creates a _CompletableFuture_ instance, then spins off some computation in another thread and returns the _Future_ immediately.

When the computation is done, the method completes the _Future_ by providing the result to the _complete_ method:

```java
public Future<String> calculateAsync() throws InterruptedException {
    CompletableFuture<String> completableFuture = new CompletableFuture<>();

    Executors.newCachedThreadPool().submit(() -> {
        Thread.sleep(500);
        completableFuture.complete("Hello");
        return null;
    });

    return completableFuture;
}
```

To spin off the computation, we use the _Executor_ API. This method of creating and completing a _CompletableFuture_ can be used with any concurrency mechanism or API, including raw threads.

Notice that **the _calculateAsync_ method returns a _Future_ instance**.

We simply call the method, receive the _Future_ instance, and call the _get_ method on it when we’re ready to block for the result.

Also, observe that the _get_ method throws some checked exceptions, namely _ExecutionException_ (encapsulating an exception that occurred during a computation) and _InterruptedException_ (an exception signifying that a thread was interrupted either before or during an activity):

```java
Future<String> completableFuture = calculateAsync();

// ... 

String result = completableFuture.get();
assertEquals("Hello", result);
```

**If we already know the result of a computation**, we can use the static _completedFuture_ method with an argument that represents the result of this computation. Consequently, the _get_ method of the _Future_ will never block, immediately returning this result instead:

```java
Future<String> completableFuture = 
  CompletableFuture.completedFuture("Hello");

// ...

String result = completableFuture.get();
assertEquals("Hello", result);
```

As an alternative scenario, we may want to [**cancel the execution of a _Future_**](https://www.baeldung.com/java-future#2-canceling-a-future-with-cancel).

##  _CompletableFuture_ With Encapsulated Computation Logic

The code above allows us to pick any mechanism of concurrent execution, but what if we want to skip this boilerplate and execute some code asynchronously?

Static methods _runAsync_ and _supplyAsync_ allow us to create a _CompletableFuture_ instance out of _Runnable_ and _Supplier_ functional types correspondingly.

_Runnable_ and _Supplier_ are functional interfaces that allow passing their instances as lambda expressions thanks to the new Java 8 feature.

The _Runnable_ interface is the same old interface used in threads and does not allow to return a value.

The _Supplier_ interface is a generic functional interface with a single method that has no arguments and returns a value of a parameterized type.

This allows us to **provide an instance of the _Supplier_ as a lambda expression that does the calculation and returns the result**. It is as simple as:

```java
CompletableFuture<String> future
  = CompletableFuture.supplyAsync(() -> "Hello");

// ...

assertEquals("Hello", future.get());
```

## **5. Processing Results of Asynchronous Computations

The most generic way to process the result of a computation is to feed it to a function. The _thenApply_ method does exactly that; it accepts a _Function_ instance, uses it to process the result, and returns a _Future_ that holds a value returned by a function:

```java
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<String> future = completableFuture
  .thenApply(s -> s + " World");

assertEquals("Hello World", future.get());
```

If we don’t need to return a value down the _Future_ chain, we can use an instance of the _Consumer_ functional interface. Its single method takes a parameter and returns _void_.

There’s a method for this use case in the _CompletableFuture._ The _thenAccept_ method receives a _Consumer_ and passes it the result of the computation. Then the final _future.get()_ call returns an instance of the _Void_ type:

```java
CompletableFuture<String> completableFuture
  = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<Void> future = completableFuture
  .thenAccept(s -> System.out.println("Computation returned: " + s));

future.get();
```

Finally, if we neither need the value of the computation nor want to return some value at the end of the chain, then we can pass a _Runnable_ lambda to the _thenRun_ method. In the following example, we simply print a line in the console after calling the _future.get():_

```java
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<Void> future = completableFuture
  .thenRun(() -> System.out.println("Computation finished."));

future.get();
```

## **6. Combining Futures**

The best part of the _CompletableFuture_ API is the **ability to combine _CompletableFuture_ instances in a chain of computation steps**.

The result of this chaining is itself a _CompletableFuture_ that allows further chaining and combining. This approach is ubiquitous in functional languages and is often referred to as a monadic design pattern.

**In the following example, we use the _thenCompose_ method to chain two _Futures_ sequentially.**

Notice that this method takes a function that returns a _CompletableFuture_ instance. The argument of this function is the result of the previous computation step. This allows us to use this value inside the next _CompletableFuture_‘s lambda:

```java
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " World"));

assertEquals("Hello World", completableFuture.get());
```

The _thenCompose_ method, together with _thenApply,_ implements the basic building blocks of the monadic pattern. They closely relate to the _map_ and _flatMap_ methods of _Stream_ and _Optional_ classes, also available in Java 8.

Both methods receive a function and apply it to the computation result, but the _thenCompose_ (_flatMap_) method **receives a function that returns another object of the same type**. This functional structure allows composing the instances of these classes as building blocks.

If we want to execute two independent _Futures_ and do something with their results, we can use the _thenCombine_ method that accepts a _Future_ and a _Function_ with two arguments to process both results:

```java
CompletableFuture<String> completableFuture 
  = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCombine(CompletableFuture.supplyAsync(
      () -> " World"), (s1, s2) -> s1 + s2);

assertEquals("Hello World", completableFuture.get());
```

A simpler case is when we want to do something with two _Futures_‘ results but don’t need to pass any resulting value down a _Future_ chain. The _thenAcceptBoth_ method is there to help:

```java
CompletableFuture future = CompletableFuture.supplyAsync(() -> "Hello")
  .thenAcceptBoth(CompletableFuture.supplyAsync(() -> " World"),
    (s1, s2) -> System.out.println(s1 + s2));
```

## **7. Difference Between _thenApply()_ and _thenCompose()_**

In our previous sections, we’ve shown examples regarding _thenApply()_ and _thenCompose()_. Both APIs help chain different _CompletableFuture_ calls, but the usage of these two functions are different.

### **7.1. _thenApply()_**[](https://www.baeldung.com/java-completablefuture#1-thenapply)

**We can use this method to work with the result of the previous call.** However, a key point to remember is that the return type will be combined of all calls.

So this method is useful when we want to transform the result of a _CompletableFuture_ call:

```java
CompletableFuture<Integer> finalResult = compute().thenApply(s-> s + 1);
```

### **7.2. _thenCompose()_**

The _thenCompose()_ is similar to _thenApply()_ in that both return a new CompletionStage. However, **_thenCompose()_ uses the previous stage as the argument**. It will flatten and return a _Future_ with the result directly, rather than a nested future as we observed in _thenApply():_

```java
CompletableFuture<Integer> computeAnother(Integer i){
    return CompletableFuture.supplyAsync(() -> 10 + i);
}
CompletableFuture<Integer> finalResult = compute().thenCompose(this::computeAnother);
```

So if the idea is to chain _CompletableFuture_ methods, then it’s better to use _thenCompose()_.

Also, note that the difference between these two methods is analogous to [the difference between _map()_ and _flatMap()_](https://www.baeldung.com/java-difference-map-and-flatmap)_._

## **8. Running Multiple _Future_s in Parallel**

When we need to execute multiple _Futures_ in parallel, we usually want to wait for all of them to execute and then process their combined results.

The _CompletableFuture.allOf_ static method allows to wait for the completion of all of the _Futures_ provided as a var-arg:

```java
CompletableFuture<String> future1  
  = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2  
  = CompletableFuture.supplyAsync(() -> "Beautiful");
CompletableFuture<String> future3  
  = CompletableFuture.supplyAsync(() -> "World");

CompletableFuture<Void> combinedFuture 
  = CompletableFuture.allOf(future1, future2, future3);

// ...

combinedFuture.get();

assertTrue(future1.isDone());
assertTrue(future2.isDone());
assertTrue(future3.isDone());
```

Notice that the return type of the _CompletableFuture.allOf()_ is a _CompletableFuture<Void>_. The limitation of this method is that it does not return the combined results of all _Futures_. Instead, we have to get results from _Futures_ manually. Fortunately, _CompletableFuture.join()_ method and Java 8 Streams API makes it simple:


``` 
String combined = Stream.of(future1, future2, future3)
  .map(CompletableFuture::join)
  .collect(Collectors.joining(" "));

assertEquals("Hello Beautiful World", combined);

```

The _CompletableFuture.join()_ method is similar to the _get_ method, but it throws an unchecked exception in case the _Future_ does not complete normally. This makes it possible to use it as a method reference in the _Stream.map()_ method.

## **9. Handling Errors**[](https://www.baeldung.com/java-completablefuture#Handling)

For error handling in a chain of asynchronous computation steps, we have to adapt the _throw/catch_ idiom in a similar fashion.

Instead of catching an exception in a syntactic block, the _CompletableFuture_ class allows us to handle it in a special _handle_ method. This method receives two parameters: a result of a computation (if it finished successfully) and the exception thrown (if some computation step did not complete normally).

In the following example, we use the _handle_ method to provide a default value when the asynchronous computation of a greeting was finished with an error because no name was provided:

```java
String name = null;

// ...

CompletableFuture<String> completableFuture  
  =  CompletableFuture.supplyAsync(() -> {
      if (name == null) {
          throw new RuntimeException("Computation error!");
      }
      return "Hello, " + name;
  }).handle((s, t) -> s != null ? s : "Hello, Stranger!");

assertEquals("Hello, Stranger!", completableFuture.get());
```

As an alternative scenario, suppose we want to manually complete the _Future_ with a value, as in the first example, but also have the ability to complete it with an exception. The _completeExceptionally_ method is intended for just that. The _completableFuture.get()_ method in the following example throws an _ExecutionException_ with a _RuntimeException_ as its cause:

```java
CompletableFuture<String> completableFuture = new CompletableFuture<>();

// ...

completableFuture.completeExceptionally(
  new RuntimeException("Calculation failed!"));

// ...

completableFuture.get(); // ExecutionException
```

In the example above, we could have handled the exception with the _handle_ method asynchronously, but with the _get_ method, we can use the more typical approach of synchronous exception processing.

## **10. Async Methods**[](https://www.baeldung.com/java-completablefuture#Async)

Most methods of the fluent API in the _CompletableFuture_ class have two additional variants with the _Async_ postfix. These methods are usually intended for **running a corresponding execution step in another thread**.

The methods without the _Async_ postfix run the next execution stage using a calling thread. In contrast, the _Async_ method without the _Executor_ argument runs a step using the common _fork/join_ pool implementation of _Executor_ that is accessed with the _ForkJoinPool.commonPool()_, as long as [parallelism > 1](https://www.baeldung.com/java-when-to-use-parallel-stream#2-common-thread-pool). Finally, the _Async_ method with an _Executor_ argument runs a step using the passed _Executor_.

Here’s a modified example that processes the result of a computation with a _Function_ instance. The only visible difference is the _thenApplyAsync_ method, but under the hood, the application of a function is wrapped into a _ForkJoinTask_ instance (for more information on the _fork/join_ framework, see the article [“Guide to the Fork/Join Framework in Java”](https://www.baeldung.com/java-fork-join)). This allows us to parallelize our computation even more and use system resources more efficiently:

```java
CompletableFuture<String> completableFuture  
  = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<String> future = completableFuture
  .thenApplyAsync(s -> s + " World");

assertEquals("Hello World", future.get());
```

## 11. JDK 9 _CompletableFuture_ API[](https://www.baeldung.com/java-completablefuture#11-jdk-9-completablefuture-api)

Java 9 introduced new instance methods that improve flexibility and ease of use when working with asynchronous computing:

- _Executor defaultExecutor()_
- _CompletableFuture<U> newIncompleteFuture()_
- _CompletableFuture<T> copy()_
- _CompletionStage<T> minimalCompletionStage()_
- _CompletableFuture<T> completeAsync(Supplier<? extends T> supplier, Executor executor)_
- _CompletableFuture<T> completeAsync(Supplier<? extends T> supplier)_
- _CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)_
- _CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)_

Java 9 enhancements also added support for creating and managing instances of  _CompletableFuture_ with static utility methods:

- _Executor delayedExecutor(long delay, TimeUnit unit, Executor executor)_
- _Executor delayedExecutor(long delay, TimeUnit unit)_
- _<U> CompletionStage<U> completedStage(U value)_
- _<U> CompletionStage<U> failedStage(Throwable ex)_
- _<U> CompletableFuture<U> failedFuture(Throwable ex)_

### **11.1. _defaultExecutor()_**[](https://www.baeldung.com/java-completablefuture#1-method-defaultexecutor)

The _defaultExecutor() r_eturns the default _Executor_ used for async methods that don’t specify an _Executor_:

```java
new CompletableFuture().defaultExecutor()
```

This can be overridden by subclasses returning an executor providing, at least, one independent thread.

### **11.2. _newIncompleteFuture()_**[](https://www.baeldung.com/java-completablefuture#1-method-defaultexecutor)

The _newIncompleteFuture()_, also known as the “virtual constructor”, is used to get a new completable future instance of the same type:

```java
new CompletableFuture().newIncompleteFuture()
```

This method is especially useful when subclassing _CompletableFuture_, mainly because it’s used internally in almost all methods returning a new _CompletionStage_, allowing subclasses to control what subtype gets returned by such methods.

### 11.3. **_copy()_**[](https://www.baeldung.com/java-completablefuture#113copy)

The _copy()_ method creates a new _CompletableFuture_ that reflects the completion state of the original _CompletableFuture_:

```java
new CompletableFuture().copy()
```

This method is useful as a form of “defensive copying”, to prevent clients from completing, while still being able to arrange dependent actions on a specific instance of _CompletableFuture_.

The _copy()_ method returns a new _CompletableFuture._ This new _CompletableFuture_ completes normally if the original _CompletableFuture_ completes normally. If the original completes with an exception, then the new one also completes with an exception. In this case, it completes with a _CompletionException_ that contains the original exception as its cause.

### 11.4. **_minimalCompletionStage()_**[](https://www.baeldung.com/java-completablefuture#114minimalcompletionstage)

The _minimalCompletionStage()_ method returns a new _CompletionStage_ which behaves in the same way as described by the copy method, however, such a new instance throws _UnsupportedOperationException_ in every attempt to retrieve or set the resolved value:

```java
new CompletableFuture().minimalCompletionStage()
```

This method is useful for scenarios where we want to expose a restricted view of a _CompletableFuture_ that prevents clients from modifying its resolved value while still allowing them to arrange dependent actions.

### **11.5. _completeAsync()_**[](https://www.baeldung.com/java-completablefuture#5-methods-completeasync)

The _completeAsync()_ method should be used to complete the _CompletableFuture_ asynchronously using the value given by the _Supplier_ provided :

```java
CompletableFuture<T> completeAsync(Supplier<? extends T> supplier, Executor executor)
CompletableFuture<T> completeAsync(Supplier<? extends T> supplier)
```

The difference between these two overloaded methods is the existence of the second argument, where the _Executor_ running the task can be specified. The default executor (returned by the _defaultExecutor_ method) will be used if none is provided.

### 11.6. **_orTimeout()_**[](https://www.baeldung.com/java-completablefuture#116ortimeout)

The _orTimeout()_  method is used to automatically complete the _CompletableFuture_ with a _TimeoutException_ if not completed with a specified timeout period:

```java
new CompletableFuture().orTimeout(1, TimeUnit.SECONDS)
```

### **11.7. _completeOnTimeout()_**[](https://www.baeldung.com/java-completablefuture#7-method-completeontimeout)

The _completeOnTimeout() c_ompletes the CompletableFuture normally with the specified value unless it’s completed before the specified timeout :

```java
new CompletableFuture().completeOnTimeout(value, 1, TimeUnit.SECONDS)
```

### **11.8. _delayedExecutor()_**[](https://www.baeldung.com/java-completablefuture#118-delayedexecutor)

The _delayedExecutor()_ returns a new _Executor_ that submits a task to the given base executor after the given delay (or no delay if non-positive) :

```java
Executor delayedExecutor(long delay, TimeUnit unit, Executor executor)
Executor delayedExecutor(long delay, TimeUnit unit)
```

Each delay commences upon invocation of the returned executor’s execute method. If no executor is specified the default executor (_ForkJoinPool.commonPool()_) will be used.

### **11.9. _completedStage()_ and _failedStage()_**[](https://www.baeldung.com/java-completablefuture#119-completedstage-and-failedstage)

The _completedStage()_ and _failedStage()_ utility methods return already resolved _CompletionStage_ instances:

```java
<U> CompletionStage<U> completedStage(U value)
<U> CompletionStage<U> failedStage(Throwable ex)
```

The _completedStage(_) returns normal completion and _failedStage()_ returns completion with an exception.

### **11.10.  _failedFuture()_**[](https://www.baeldung.com/java-completablefuture#1110-failedfuture)

The _failedFuture()_ method adds the ability to specify an already completed exceptionally _CompletableFuture_ instance:

```java
<U> CompletableFuture<U> failedFuture(Throwable ex)
```

This method can be useful in testing or simulating failure conditions in asynchronous workflows.
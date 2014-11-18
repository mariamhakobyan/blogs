##Chapter 10 - Concurrency
  
### Item 66: Synchronize access to shared mutable data

* Synchronized keyword combines the functionality of  mutual exclusion and the volatile keyword.  Not only does synchronization prevent a thread from observing an object in an inconsistent state, it also ensures that each thread entering the block sees the effects of all previous modifications.
* You cannot avoid synchronization just because you are dealing with atomic data.  Atomic merely guarantees that a thread would not see an arbitrary value when reading the field; it does not guarantee that a value written by one thread would be visible to another.
* The language specification guarantees that reading or writing a variable is atomic unless the variable is of type long or double.
* If you are using a Boolean variable to signal to another thread to stop, then mark it as volatile. Else the hoisting optimization by some VMs would result in a liveness failure.
* Synchronization has no effect unless both read and write operations are synchronized.
* Increment operator (++) is not atomic.
    * It performs two operations on the field: first it reads the value, then it writes back a new value, equal to the old value plus one. If a second thread reads the field between the time a thread reads the old value and writes back a new one, the second thread will see the same value as the first and return the same serial number. This is a safety failure: the program computes the wrong results.
* Do not use  Thread.stop() as it is depreciated.
* Volatile and Atomic types should be carefully used. 
    * While the volatile modifier performs no mutual exclusion, it guarantees that any thread that reads the field will see the most recently written value.
    * Use the class AtomicLong instead of normal primitive, which is part of java.util.concurrent.atomic.
* Confine mutable data to a single thread as far as possible.
* In summary, when multiple threads share mutable data, each thread that reads or writes the data must perform synchronization.


### Item 67: Avoid excessive synchronization 

* To avoid liveness and safety failures, never cede control to the client within a synchronized method or block. 
    * The class has no knowledge of what the method does and has no control over it. Depending on what an alien method does, calling it from a synchronized region can cause exceptions, deadlocks, or data corruption.
* As a rule, you should do as little work as possible inside synchronized regions.
    * Do not call alien methods in synchronized block. Doing this is uncontrollable and may cause exceptions and deadlock.
    * If your array is traversed frequently, but modified rarely, then consider using a CopyOnWriteArrayList. Use concurrent containers like CopyOnWriteArrayList to separate the data writing and reading. This is particularly useful to the situation in which writing data is happened occasionally.
    * In a multicore world, the real cost of excessive synchronization is not the CPU time spent obtaining locks; it is the lost opportunities for parallelism and the delays imposed by the need to ensure that every core has a consistent view of memory.
    * Do not synchronize the class internally unless you have a good reason.
    

### Item 68: Prefer executors and tasks to threads

* Executor framework separates the task and mechanism of executing. The Executor framework is a flexible interface-based task execution facility. So use executors prior to  Thread. See java.util.concurrent.Executors.
    * Creating a work queue is a single line of code:
            ExecutorService executor = Executors.newSingleThreadExecutor();
    * Here is how to submit a runnable for execution:
            executor.execute(runnable);
    * Here is how to tell the executor to terminate gracefully (if you fail to do this, it is likely that your VM will not exit):
            executor.shutdown();

* If you want more than one thread to process requests from the queue, simply call a different static factory that creates a different kind of executor service called a *thread pool*.
* For a lightly loaded server, use CachedThreadPool. For a heavily loaded server, use FixedThreadPool. Using ScheduledThreadPoolExecutor prior to Timer when multiple timing tasked are required.
* The notion of thread combines the unit of work with the mechanism for executing it.  Now the unit of work is called a task.  The mechanism for executing it is in the executor service.
* The key abstraction is the unit of work, which is called a task. The tasks are either Runnable or Callable (which is Runnable with a return value).
* Use *ScheduledThreadPoolExecutor* as a replacement for java.util.Timer.
    * A timer uses only a single thread for task execution, which can hurt timing accuracy in the presence of long- running tasks. If a timer’s sole thread throws an uncaught exception, the timer ceases to operate. A scheduled thread pool executor supports multiple recovers gracefully from tasks that throw unchecked exceptions.
* Read Goetz’s Java Concurrency in Practice.

### Item 69: Prefer concurrency utilities to wait() and notify()

* The higher-level utilities in java.util.concurrent fall into three categories: the Executor Framework, concurrent collections and synchronizers.
    * The concurrent collections provide high-performance concurrent implementations of standard collection interfaces such as List, Queue, and Map.
* Given the difficulty of using wait and notify correctly, you should always use the higher-level concurrency utilities instead.
* Use  ConcurrentHashMap in preference to  Collections.synchronizedMap  or  Hashtable.
* Some of the collection interfaces have been extended with blocking operations, which wait (or block) until they can be successfully performed. 
    * BlockingQueue extends Queue and adds several methods, including take, which removes and returns the head element from the queue, waiting if the queue is empty. This allows blocking queues to be used for work queues (also known as producer-consumer queues).
* Synchronizers are objects that enable threads to wait for one another, allowing them to coordinate their activities. Common synchronizers include CyclicBarrier, CountdownLatch and Semaphore.
    * Countdown latches are single-use barriers that allow one or more threads to wait for one or more other threads to do something.
* For interval timing, always use System.nanoTime() in preference to System.currentTimeMillis().
    * System.nanoTime is both more accurate and more precise, and it is not affected by adjustments to the system’s real-time clock.
* Always use the wait loop idiom to invoke the wait() method; never invoke it outside of a loop.
```java
    //The standard idiom for using the wait method  
    synchronized (obj) {  
        while (<condition does not hold>)  
        obj.wait(); // (Releases lock, and reacquires on wakeup)  
    
        // Perform action appropriate to condition  
    }
```

* You should always use notifyAll. 
    * It will always yield correct results because it guarantees that you’ll wake the threads that need to be awakened. You may wake some other threads, too, but this won’t affect the correctness of your program. These threads will check the condition for which they’re waiting and, finding it false, will continue waiting.
    * As an optimization, you may choose to invoke notify instead of notifyAll if all threads that could be in the wait-set are waiting for the same condition and only one thread at a time can benefit from the condition becoming true.
* There is seldom, if ever, a reason to use wait and notify in new code.

### Item 70: Document thread safety

* The presence of synchronized modifier in a method declaration is an implementation detail, not a part of its exported API. One cannot conclude on its basis that the method is thread-safe.
* To enable safe concurrent use, a class must clearly document what level of thread safety it supports. Methods with special thread safety properties should describe these properties in their own documentation comments.
* There are 5 levels of thread-safety.
    * **immutable** like String, Long and BigInteger.
    * **unconditionally thread-safe** like Random and ConcurrentHashMap. They have internally enough synchronization that they can be used without external synchronization.
    * **conditionally thread-safe** wherein certain methods need synchronization. Example is collection returned by Collections.synchronized wrappers whose iterators need synchronization.
    * **not thread-safe** like ArrayList.  Clients here must surround each method invocation with synchronization.
    * **thread-hostile**.  If a class modifies internally its static members without synchronization, then do not use the class in multithreaded applications.
* If an object represents a view on another object, you must synchronize on the backing object to prevent its direct modification.
* If you have a public lock, you expose yourself to a denial of service attack by a client who holds on to the lock for long. To prevent this, have a private final lock object and synchronize on it. This private lock idiom is to used only on unconditionally thread-safe classes.
* The private lock idiom is useful for classes designed for inheritance.


### Item 71: Use lazy initialization judiciously

* Lazy initialization is the act of delaying the initialization of a field until its value is needed. If the value is never needed, the field is never initialized.
    * The best advice for lazy initialization is “don’t do it unless you need to”.
* Under most circumstances, normal initialization is preferable to lazy initialization.
* Lazy initialization decreases the cost of initializing a class or creating an instance, at the expense of increasing the cost of accessing the lazily initialized field. It may be worthwhile when only part of instances will be initialized in practice.
* Use a synchronized accessor to the getfield method to ensure the concurrency.
* If you use lazy initialization to break an initialization circularity, use a synchronized accessor. Here, put the synchronized modifier before the accessor. Inside the method, check if the field is null, and if so, call the initializing method.
* If you need to use lazy initialization for performance reasons on a static field, then use the so-called lazy initialization holder class idiom. In the definition of the static final variable, equate it to the initializing method.
```java
    // Lazy initialization holder class idiom for static fields
    private static class FieldHolder {
        static final FieldType field = computeFieldValue();
    }
    
    static FieldType getField() { return FieldHolder.field; }
```
* If you need to use lazy initialization for performance reasons on an instance field, then use the so-called double check idiom. First check if the field is null. If not, return the value you have.  If it is null, synchronize on object; again check for null and if found to be null, initialize. The instance field should be declared as volatile.
```java
    // Double-check idiom for lazy initialization of instance fields
    private volatile FieldType field;
    FieldType getField() {
        FieldType result = field;
        if (result == null) { // First check (no locking)
            synchronized(this) {
                result = field;
                if (result == null) // Second check (with locking)
                field = result = computeFieldValue();
            }
        }
        return result;
    }
```
* If you can tolerate repeated initialization, then remove the second check from the double-check idiom. This is called the single-check idiom.
* If you further do not care if each thread re-calculates the value, then remove the volatile modifier from the single-check idiom. This is then called the racy single-check idiom. This is used internally by the String class to cache their hash codes.


### Item 72: Don't depend on the thread scheduler

* Any program that relies on the thread scheduler for correctness or performance is likely to be non portable.
* Threads should not run if they are not doing useful work. Threads should not busy-wait.
* Do not fix bugs by calling Thread.yield. It has no testable semantics.
* Do not fix bugs by changing thread priorities. Thread priorities are among the least portable features of the Java platform.
* The best way to write a robust, responsive, portable program is to ensure that the average number of runnable threads is not significantly greater than the number of processors. 
* Use Thread.sleep(1) instead of Thread.yield() for increasing concurrency.

#### Item 73: Avoid thread groups

* They are a deprecated feature of the language.
* Use thread pool executors instead.
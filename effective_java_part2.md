##Chapter 7 - Methods

### Item 38: Check parameters for validity 

* At the start of methods, check parameters for validity.
* For public methods, document the exceptions thrown using the @throws javadoc tag. The exception would typically be an IllegalArgumentException, IndexOutOfBoundsException or NullPointerException.
* For non-public methods, check parameters using a series of assertions. Unlike normal validity checks, asserts throw an AssertionError if they fail. Further, they have no cost and no effect unless you enable them by passing -ea to the java interpreter.
* In constructors in particular, one must always check the validity of the parameters passed.


### Item 39: Make defensive copies when needed

* You must program defensively, with the assumption that the clients of your class will do their best to destroy its invariants.
    * This may actually be true if someone tries to break the security of your system, but more likely your class will have to cope with unexpected behavior resulting from honest mistakes on the part of programmers using your API. Either way, it is worth taking the time to write classes that are robust in the face of ill-behaved clients.
* If you want to create an immutable class, make sure to make a defensive copy of each mutable parameter in the constructor and set it then only to the field declared as final.
* Defensive copies are made before checking the validity of parameters and further, the validity check is performed on the copies rather than the originals.  This protects the class against changes to the parameters from another thread during the window of vulnerability between the time of parameters being checked and parameters being copied. Violating this principle leads to the so-called TOCTOU or time-of-check/time-of-use attack.


Defensive copy usually implemented by:

1. clone() method
1. copy constructor
1. static factory

This is not mandatory. If the class trusts the clients, then defensive copy could be replaced by documentation explanation to save the cost.

* Do not use the clone method to make a defensive copy of a parameter whose type is subclassable by untrusted parties.
* In your accessors, return defensive copies of mutable internal fields.  You can use clone here however.
* To avoid thinking about defensive copying, consider using immutable objects wherever possible.


### Item 40: Design method signatures carefully

* Choose method names carefully.  Follow naming convention of package over general naming convention picked up elsewhere.
* Don't go overboard in providing convenience methods.
    * Too many methods make a class difficult to learn, use, document, test, and maintain. 
* Avoid long parameter lists, especially if they are identically typed. To ameliorate the problem, there are three ways:
    * Aim for four parameters or fewer. Most programmers can’t remember longer parameter lists. If many of your methods exceed this limit, your API won’t be usable without constant reference to its documentation.
    * Break up method into multiple methods.
    * Create helper classes to hold groups of parameters.
    * Adapt the Builder pattern from object construction to method invocation.
* For parameter types, favor interfaces over classes.
* Prefer two-element enum types to boolean parameters for clearness and extendability.


### Item 41: Use overloading judiciously

* The choice of which overloading to invoke is made at compile time.
* Selection among overloaded methods is static, while selection among overridden methods is dynamic.
    * The correct version of an overridden method is chosen at runtime, based on the runtime type of the object on which the method is invoked.
    * Avoid confusing uses of overloading.
* Never export two overloadings with the same number of parameters. Do not overload a method with varargs.
* For constructors, the option of not overloading is often not present.  There, consider using static factories.
* In the List implementation, there are two overloading of the remove method. remove(int i) removes the element from position i.  remove(E e) removes the element e of type E with which the generic List was instantiated.  Now when E is Integer (ie boxed version of int), there is possibility of confusion. remove((Integer) i) and remove(i) behave differently for int i.
* In the String class, valueOf(char[]) and valueOf(Object) do completely different things.  This is a mistake in design.


### Item 42: Use varargs judiciously

* Do not use varargs unless you really need it and benefit from it.
* Varargs is expensive in resources as it causes internally an array allocation and initialization.
* Arrays.asList() is tricky. It returns List<Integer> when the argument is Integer[] and returns List<int[]> when the argument is int[].
* If your method requires a minimum of one argument, use "int arg1, int.. args" instead of "int.. args".


### Item 43: Return empty arrays or collections, not nulls

* To return an array, use "return someList.toArray(new Type[])".  To return an empty list, use Collections.emptyList().
* There is no reason ever to return null from an array or collection-valued method instead of returning an empty array or collection.
* Returning null forces the client of the code to separately check for the null.

### Item 44: Write doc comments for all exposed API elements

* To document your API properly, you must precede every exported class, interface, constructor, method, and field declaration with a doc comment.
* The doc comment for a method must state what a method does, not how it does it. Mention *pre-conditions* (e.g. parameters, thrown exceptions), *post-conditions*, and *side-effects* (e.g. the method starts a background thread). Describe thread-safety of the class as well.
* The doc comment for a method should describe succinctly the contract between the method and its client.
* Use @param, @return and @throws tags.  You can use HTML tags in your comment.
* It is no longer necessary to use the HTML 'code' or 'tt' tags in doc comments. Use @code instead of 'code' because it eliminates the need to escape HTML metacharacters.
* The first sentence of each doc comment becomes the summary description of the element. 
* No two members or constructors should have the same summary description.
* When documenting a generic type or method, be sure to document all type parameters.
* When documenting an enum type, be sure to document the constants.
* When documenting an annotation type, be sure to document any members.


---
##Chapter 8 - General Programming

### Item 45: Minimize the scope of local variables

* The most powerful technique for minimizing the scope of a local variable is to declare it where it is first used.
* Nearly every local variable declaration should contain an initializer.
* Prefer for loops to while loops.
* Keep methods small and focused.
    * If you combine two activities in the same method, local vari- ables relevant to one activity may be in the scope of the code performing the other activity. To prevent this from happening, simply separate the method into two: one for each activity.


### Item 46: Prefer for-each loops to traditional for loops

The for-each loop provides compelling advantages over the traditional for loop in clarity and bug prevention, with no performance penalty.

There are three common situations where you can’t use a for-each loop:

* filtering (remove list elements), 
* transforming (replacing list elements),
* parallel iteration (traversing multiple collections).

For any group of elements, consider implementing Iterable.

### Item 47: Know and use the libraries

* By using a standard library, you take advantage of the knowledge of the experts who wrote it and the experience of those who used it before you.
* For example, if you want a random number, use Random.nextInt() rather than trying to cook up a pseudo-random generator.
* Every programmer must know the contents of three packages - java.lang, java.util and java.io.
* Within java.util, the Collections framework and concurrent subpackage are of particular importance.

### Item 48: Avoid float and double if exact answers are required

* The float and double types are particularly ill-suited for monetary calculations, because it is impossible to represent 0.1 (or any other negative power of ten) as a float or double exactly. 
* Instead use BigDecimal, int or long for monetary calculations.
* Use int for number with less than 9 digits and long for 19 digits.

### Item 49: Prefer primitive types to boxed primitives

* There are three differences between the primitives and the boxed primitives.
    * Primitives only have their values, boxed primitives have their identities distinct from values.
    * Boxed primitive have only non-functional value - null.
    * Primitives are generally time and space efficient as compared to boxed primitives.
* Applying the == operator to boxed primitives is almost always wrong.
* When you mix primitives and boxed primitives in a single operation, the boxed primitives is auto-unboxed. Beware when your program does unboxing, it can throw a NullPointerException.
* Autoboxing reduces the verbosity, but not the danger, of using boxed primitives.


### Item 50: Avoid strings where other types are more appropriate

* Strings are more cumbersome, less flexible, slower and error-prone than other types, if they are used inappropriately.  Do not use strings in place of primitive types, enums and aggregate types.
* Capability refers to an unforgeable key, such as the key used internally in ThreadLocal to store your object against each thread. Strings should not be used for capabilities.


### Item 51: Beware the performance of string concatenation

* Since strings are immutable, concatenating two strings requires the contents of both strings to be copied. This implies that using the string concatenation operator repeatedly to concatenate n strings requires time quadratic in n.
* To achieve acceptable performance, use a StringBuilder in place of a String to store the statement under construction.
* Use the unsynchronized StringBuilder instead. StringBuffer is now deprecated.

### Item 52: Refer to objects by their interfaces

* If appropriate interface types exist, then parameters, return values, variables, and fields should all be declared using interface types.
* If you get into the habit of using interfaces as types, your program will be much more flexible.
    * If you decide that you want to switch implementations, all you have to do is change the class name in the constructor (or use a different static factory).
* There are three scenarios in which you would not use an interface.
    * For Value classes like String, Random, there is no interface.
    * For Class-based framework, use the base class as there may be no interface available.
    * If the class (for e.g. LinkedHashMap) provides methods not found in the interface, then use the class instead of the interface.

### Item 53: Prefer interfaces to reflection

There are three disadvantages of using reflection.

*  You lose all the benefits of compile-time type checking.
    * If a program attempts to invoke a non existent or inaccessible method reflectively, it will fail at runtime unless you’ve taken special precautions.
*  The code required to perform reflective access is clumsy and verbose.
*  Performance suffers.
    * Reflective method invocation is much slower than normal method invocation.
* As a rule, objects should not be accessed reflectively in normal applications in runtime.  
    * Exceptions to the rule include class browsers, object inspectors, code analysis tools, interpretive embedded systems, and remote procedure call systems.

For many programs that must use a class that is unavailable at compile-time, there exists at compile time an appropriate interface or superclass by which to refer to the class. If so, you can create instances reflectively, and access them normally via their interface or superclass.

### Item 54: Use native methods judiciously

* The Java Native Interface (JNI) allows Java applications to call native methods, which are special methods written in native programming languages like C or C++.
* It is rarely advisable to use native methods for performance reasons.
* The use of native methods has the following disadvantages:
    * Native languages are not safe from memory corruption.
    * Applications are now more difficult to debug.
    * Performance cost is present in going into and out of native code.
    * Writing the glue code is tedious to write and read.

Avoid using native code unless you really need it.


### Item 55: Optimize judiciously

* Strive to write good programs rather than fast ones.
* Strive to avoid design decisions that would limit performance.
* Consider the performance consequences of your API design decisions. For example
    * Making a public type mutable entails a lot of defensive  copying.
    * Using inheritance in a public class where composition should have been used places artificial limits on the performance of the subclass.
    * Using concrete implementations instead of interfaces can prevent you from taking advantages of future improvements to the library.
* It is a very bad idea to manipulate an API to achieve good performance.
* Measure performance before and after each optimization.
* It is extremely important to run a profiler to understand which part of the code is slow.
* Java does not have a strong performance model.  Performance varies greatly across JVMs.

### Item 56: Adhere to generally accepted naming conventions

See Java Language Specification Section 6.8 or see Pg 237 of the text.

Package                          com.google.inject, org.joda.time.format
Class or Interface         	Timer, FutureTask, LinkedHashMap, HttpServlet
Method or Field            	remove, ensureCapacity, getCrc
Constant Field               	MIN_VALUE, NEGATIVE_INFINITY
Local Variable                	i, xref, houseNumber
Type Parameter            	T, E, K, V, X, T1, T2

  
---
  
##Chapter 9 - Exceptions

### Item 57: Use exceptions only for exceptional conditions

* Exceptions are, as their name implies, to be used only for exceptional conditions; they should never be used for ordinary control flow.
* A well-designed API must not force its clients to use exceptions for ordinary control flow.
* Placing code inside try-catch block inhibits JVM optimizations.
* If you have a state-dependent method, then use either a distinguished return value (such as null) or a state-testing method instead of choosing to use exceptions for control flow.

### Item 58: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors

* Use checked exceptions for conditions from which the caller can reasonably be expected to recover.
* Use runtime exceptions to indicate programming errors.
* All of the unchecked throwables you implement should subclass RuntimeException.
* Besides checked and unchecked exceptions, there is a category of Error that can be thrown.  There is a strong convention against using it outside of the JVM. So do not create any subclasses of Error. Instead sub-class RuntimeException.
* While it is possible to define other categories of throwable objects, do not do so.

### Item 59: Avoid unnecessary use of checked exceptions

Checked exceptions are a wonderful feature of the Java programming language. Unlike return codes, checked exception force the programmer to deal with exceptional conditions, greatly enhancing reliability. That said, overuse of checked exceptions can make an API far less pleasant to use.

Avoid using the checked exception unless:

* the exception can not avoided by proper use of API.
* Handling exception brings benefits.


### Item 60: Favor the use of standard exceptions

One of the attributes that most strongly distinguishes expert programmers from less experienced ones is that experts strive for and usually achieve a high degree of code reuse. Exceptions are no exception to the general rule that code reuse is good.

There are three reasons for favoring standard exceptions, namely

* It makes the API easier to learn and use.
* It makes the API easier to read later.
* Fewer exception classes imply a smaller memory footprint and less time spent loading classes.

Use *IllegalArgumentException* when you are passed an argument whose value is inappropriate. Use IllegalStateException when you the caller attempted to use an object before it was properly initialized.

Use a *NullPointerException* when a caller passes null for a parameter for which null values are prohibited. Use IndexOutOfBoundsException on receiving an out-of-range value for a parameter representing an index into a sequence.

Use *ConcurrentModificationException* if an object designed for use b a single thread (or with external synchronization) is being concurrently modified. Use UnSupportedOperationException if an object does not support an attempted operation.


### Item 61: Throw exceptions appropriate to the abstraction

* If it is impossible to prevent the occurring of exception in lower level, propagate it to higher level by catching and rethrowing.
* Higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstraction. 
    * This idiom is known as *exception translation*.

```java
	// Exception Translation
    try {
        // Use lower-level abstraction to do our bidding
        . . .
    } catch(LowerLevelException e) {
        throw new HigherLevelException(. . .);
    }
```
* A special form of exception translation called *exception* chaining is appropriate in cases where the lower-level exception might be helpful to someone debugging the problem that caused the higher-level exception.
```java
	// Exception Chaining
    try {
        . . . // Use lower-level abstraction to do our bidding
    } catch (LowerLevelException cause) {   
        throw new HigherLevelException(cause);
    }
```
* Most exceptions have chaining-aware constructor; but if they don't, then use the Throwable interface's initCause method.


### Item 62: Document all exceptions thrown by each method

* Always declare checked exceptions individually, and document precisely the conditions under which each one is thrown using the Javadoc @throws tag.
* Never declare that a method “throws Exception” or, worse yet, “throws Throwable.” Specify the particular exceptions that you expect.
* Use the Javadoc @throws tag to document each unchecked exception that a method can throw, but do not use the throws keyword to include unchecked exceptions in the method declaration.
* If an exception is thrown by many methods in a class for the same reason, it is acceptable to document the exception in the class’s documentation comment rather than documenting it individually for each method.
* A well-documented list of unchecked exceptions a method can throw acts as a list of preconditions for its successful execution.  It is important that you do this for all interfaces; this specifies their general contract.

### Item 63: Include failure-capture information in detail messages

* Printing out the stack trace of an exception prints out its toString representation which in turn consists of the exception-class name along with its detail message. So ensure that the detail message of an exception contains the values of all the parameters and fields that contributed to the exception in the first place.
* Do not include unnecessary prose in exceptions.
* Instead of a string constructor, it would have been better if Java had constructor for exceptions which required users to fill in all the required information for its detail message.
* It is rare that the programmer would want programmatic access to the details of an unchecked exception.

### Item 64: Strive for failure atomicity

* Generally speaking, a failed method invocation should leave the object in the state that it was in prior to the invocation. A method with this property is said to be failure atomic.
```java
    Example:  
    public Object pop() {
        if (size == 0)  throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
```
For arguments passed to a method, failure atomicity is ensured by these approaches:

* Failure atomicity is guaranteed if the object you send is immutable.
* Check parameters for validity at start of method itself.
* Check parameters for validity before making any change to the object.
* Write recovery code to return object to original state on encountering an error.
* Make a temporary copy of the object received; operate on it and copy back to the original object only if you are successful.

Do not try to maintain the failure atomicity when throwing errors.

Failure Atomicity refers to a property of a method that ensures that a failed invocation of it would always leave objects in the state they were prior to the method invocation.

Failure atomicity is not achievable (e.g. in some ConcurrentModificationException cases) or desirable in a very small minority of cases. In all other scenarios, strive for it.

As a rule, any generated exception that is part of a method’s specification should leave the object in the same state it was in prior to the method invocation. Where this rule is violated, the API documentation should clearly indicate what state the object will be left in.
  

### Item 65: Don't ignore exceptions

* Do not use empty catch block to ignore the exception. Throw it outward at least if you do not know how to handle it.
* At the very least, the catch block should contain a comment explaining why it is appropriate to ignore the exception.


----

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


----
  
##Chapter 11 - Serialization

### Item 74: Implement Serializable judiciously

* Implementing Serializable decreases the flexibility to change the class's implementation after release.
    * When a class implements Serializable, its byte-stream encoding (or serialized form) becomes part of its exported API.
    * If you do not make the effort to design a custom serialized form, but merely accept the default, the serialized form will forever be tied to the class’s original internal representation. In other words, the class’s private and package-private instance fields become part of its exported API, and the practice of minimizing access to fields loses its effectiveness as a tool for information hiding.
    * If you accept the default serialized form and later change the class’s internal representation, an incompatible change in the serialized form might result. Clients attempting to serialize an instance using an old version of the class and deserialize it using the new version will experience program failures.
* Implementing Serializable increases the likelihood of bugs and security holes.
* Have a custom serialized form.  Have your own stream-unique identifiers, also referred to as serial version UIDs.
    * If you do not specify this number explicitly by declaring a static final long field named serialVersionUID, the system automatically generates it at runtime by applying a complex procedure to the class.
* Implementing Serializable enables an extra-linguistic mechanism for creating objects - creating a security hole.
    * Because there is no explicit constructor associated with deserialization, it is easy to forget that you must ensure that it guarantees all of the invariants established by the constructors and that it does not allow an attacker to gain access to the internals of the object under construction.
* Implementing Serializable increases the testing burden since on every new release, you must try to deserialize with older versions of the class and then test its semantics.
    * You must ensure both that the serialization-deserialization process succeeds and that it results in a faithful replica of the original object.
* Think long and hard before implementing Serializable.
* Classes designed for inheritance should rarely implement Serializable, and interfaces should rarely extend it.
    * If a class that is designed for inheritance is not serializable, it may be impossible to write a serializable subclass.
* You should consider providing a parameterless constructor on non-serializable classes designed for inheritance.
* Inner classes should not implement Serializable,  static classes can.

### Item 75: Consider using a custom serialized form

* Do not accept the default serialized form without first considering whether it is appropriate.
* The default serialized form is likely to be appropriate if an object’s physical representation is identical to its logical content.
* Even if you decide that the default serialized form is appropriate, you often must provide a readObject() method to ensure invariants and security.
* Provide writeObject and readObject methods implementing this serialized form.  The transient modifier indicates that an instance field is to be omitted from a class’s default serialized form.
* Using the default serialized form has four disadvantages:
    * ties the exported API to the internal representation permanently.
    * consumes excessive space.
    * consumes excessive time.
    * causes stack overflows.
* Even if all fields are transient, do not dispense with the defaultWriteObject and defaultReadObject invocation.
* Declare a field as non-transient if it is part of the logical state of the object.
* You must impose any synchronization on object serialization that you would impose on any other method that reads the entire state of the object.
* Regardless of what serialized form you choose, declare an explicit serial version UID in every serializable class you write.
* If no serial version UID is provided, an expensive computation is required to generate one at runtime.

private static final long serial VersionUID = randomLongValue ;


### Item 76: Write readObject() methods defensively

* Defensively copy mutable components of immutable classes and check for invariants in your readObject method. 
* Think of readObject like a public constructor; do not invoke any overridable methods in the readObject() method.
* Do not use the writeUnshared and readUnshared methods.
    * They are typically faster than defensive copying, but they don’t provide the necessary safety guarantee.
    * A simple test for deciding whether the default readObject method is acceptable for a class: would you feel comfortable adding a public constructor that took as parameters the values for each non transient field in the object and stored the values in the fields with no validation whatsoever? If not, you must provide a readObject method, and it must perform all the validity checking and defensive copying that would be required of a constructor. Alternatively, you can use the serialization proxy pattern.
* When an object is deserialized, it is critical to defensively copy any field containing an object reference that a client must not possess.
* Check any invariants and throw an  InvalidObjectException if a check fails.

### Item 77: For instance control, prefer enum types to readResolve()

* For a singleton class implementing Serializable, declare all fields as transient and put in a readResolve method to return the one true reference.
    * If the Elvis class is made to implement Serializable, the following readResolve method suffices to guarantee the singleton property: 
```java
    // readResolve for instance control - you can do better! 
    private Object readResolve() { 
        // Return the one true Elvis and let the garbage collector 
        // take care of the Elvis impersonator. 
        return INSTANCE; 
    }
```
* If you depend on readResolve() for instance control, all instance fields with object reference types must be declared transient.
* Prefer the enum singleton to the class-based approach for security reasons. Instance control through Enum is preferred to the readResolve().


### Item 78: Consider serialization proxies instead of serialized instances

* If performance is not a big concern, use the the serialization proxy pattern. Serialization proxy pattern is implemented based on an inner static class with a single constructor.
* Make both your class and its proxy class implement Serializable. 
* In the writeReplace method of your main class, return a proxy object constructed from itself. 
* In the readObject method of the proxy class, return the main class object. 
* Use readResolve() method to return an instance of enclosing class at the time of deserialization, since the method in the inner class is able to call the constructor of enclosing class.
* In both cases, use the public constructors of the classes involved. Further block the readObject method of the proxy class by making it throw an exception whenever invoked.
* Two limitations of the serialization proxy pattern:
    * It is not compatible with classes that are extendable by their clients (in that time, the enclosing class has not been initialized).
    * It is not compatible with some classes whose object graphs contain circularities.
* Serialization proxy pattern is more secure, but with higher expense.
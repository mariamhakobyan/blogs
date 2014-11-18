Summary of a must read book for each Java engineer - [Effective Java 2nd Edition by Joshua Bloch](http://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) (Chapter 1-6).  

----

#Chapter 1 - Introduction

While the rules of this book do not apply 100 percent of the time, they do characterize best programming practices in the great majority of cases. You should not slavishly follow these rules, but violate them only occasionally and with good reason. Learning the art of programming, like most other disciplines, consists of first learning the rules and then learning when to break them. 

The book is not about performance. It’s about writing programs, that are clear, correct, usable, robust, flexible, and maintainable.If you can do that, it’s usually a relatively simple matter to get the performance you need.

----

#Chapter 2 - Creating and Destroying Objects


### Item 1: Consider static factory methods instead of constructors
**Advantages:**

* Unlike constructors, they have names.
    * The constructor might not describe the object being returned, but the static factory method with a well-chosen name is easier to use and the resulting code is easier to read.
    * Unlike constructors, they are not required to create a new object each time they are invoked.
    * This allows immutable classes (Item 15) to use preconstructed instances, or to cache instances as they’re constructed, and dispense them repeatedly to avoid creating unnecessary duplicate objects. The Boolean.valueOf(boolean) method illustrates this technique: it never creates an object. 
* Unlike constructors, they can return an object of any subtype of their return type.
    * This gives you great flexibility in choosing the class of the returned object.
* Reduces the verbosity of creating parameterized type instances (Obviated by diamond in Java 7).

**Disadvantages:**


* Classes with only static factory methods without public or protected constructors cannot be subclassed.
* Not readily distinguishable from other static methods. They do not stand out in API documentation in the way that constructors do, so it can be difficult to figure out how to instantiate a class that provides static factory methods instead of constructors.
Here are some common names for static factory methods:
* ***valueOf*** — Returns an instance that has, loosely speaking, the same value as its parameters. Such static factories are effectively type-conversion methods.
* ***of*** — A concise alternative to valueOf, popularized by EnumSet (Item 32).
* ***getInstance*** — Returns an instance that is described by the parameters but cannot be said to have the same value. In the case of a singleton, getInstance takes no parameters and returns the sole instance.
* ***newInstance*** — Like getInstance, except that newInstance guarantees that each instance returned is distinct from all others.
* ***getType*** — Like getInstance, but used when the factory method is in a different class. Type indicates the type of object returned by the factory method.
* ***newType*** — Like newInstance, but used when the factory method is in a different class. Type indicates the type of object returned by the factory method.


**Summary:** Static factory methods and public constructors both have their uses, and it pays to understand their relative merits. Often static factories are preferable, so avoid the reflex to provide public constructors without first considering static factories.


### Item 2: Consider a builder when faced with many constructor parameters
Static factories and constructors share a limitation: they do not scale well to large numbers of optional parameters. The client is left wondering what all those values mean and must carefully count parameters to find out. Long sequences of identically typed parameters can cause subtle bugs. If the client accidentally reverses two such parameters, the compiler won’t complain, but the program will misbehave at runtime (Item 40).

Traditional solutions to these are - *telescoping constructor* pattern and *JavaBeans* pattern.

The telescoping constructor pattern is where you provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters.

In JavaBeans pattern you call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest.

Both work, but are not efficient and readable, and have many disadvantages.


* Consider creation of a class with a number of optional parameters. There are two traditional approaches for this task. The first involves *telescoping constructors*, in which you provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters. It’s hard to read and follow as the number of parameters increase.  The second approach is to use the *Javabeans* pattern of a nullary constructor followed by a series of setter methods being called.  This has three disadvantages, namely:
    * Excessively verbose
    * Since construction is now spread across several calls, construction is in a inconsistent state in between.
    * Object cannot be made immutable anymore.
* To overcome the disadvantages of both approaches above, we should use the Builder pattern with the following three steps.
    * The client calls a factory to get a builder object first. 
    * Then all setter methods are called on this builder object to set each optional parameter in a fluent fashion.
    * Finally the client calls the parameter-less build() method on the builder to get an immutable instance.



### Item 3: Enforce the singleton property with a private constructor or an enum type

Making a class a singleton can make it difficult to test its clients as it is not possible to substitute a mock implementation for a singleton unless it implements an interface to serve as its type.

There have two traditional approaches for declaring a singleton. In both, we rely on a private constructor and exporting a public static member.

* Public field - the member is a final field accessed directly as className.fieldName. 

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { … }
    public void leaveTheBuilding() { … }
}
```

* Static factory method - the member is obtained through a static factory. 
```java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { … }
    public static Elvis getInstance() { return INSTANCE; }
    public void leaveTheBuilding() { … }
}
```

Given that modern JVM would inline the call to the static factory, the two approaches are equally efficient. The second approach however is more amenable to change to say threadlocal behavior rather than singleton.

The traditional approaches above suffer from two deficiencies. 

* A privileged client can reflectively invoke the private constructor with the aid of setAccessible method.  So you must ensure that the constructor throws an exception if a second instance is created.
* To make the class serializable, it is not sufficient to merely state “implements Serializable”. Declare all instance fields transient and provide a readResolve() method returning the same instance each time.

The best way to implement a singleton property is use a single-element enum. This solves the serialization problem for free.

```java
 //Enum singleton - the preferred approach  
 public enum Elvis {  
    INSTANCE;  
    public void leaveTheBuilding() { … }  
 }
```

### Item 4: Enforce non instantiability with a private constructor
For utility classes, it is desirable to enforce non-instantiability. Marking it as abstract allows the creation of a subclass that can be instantiated. So it is preferable to put in a private constructor. To prevent a method within the class from calling the constructor, throw an assertion error inside the private constructor.

Some utility classes like java.lang.Math or java.util.Arrays are not designed to be instantiated.


### Item 5: Avoid creating unnecessary objects

* Since you can always re-use immutable objects, prefer ```java String s = “dummy” to String s  = new String(“dummy”);```
* Boolean.valueOf(String) is preferable to Boolean(String) constructor. Prefer static factories to enable re-use.
* Use static initializers to set mutable objects that would not be changed later.
* Prefer primitives to boxed primitives, and watch out for unintentional autoboxing.

### Item 6: Eliminate obsolete object references
Although Java has garbage collector, memory leak can still happen. GC reclaims the memory by inspecting the references of objects.

* Nulling out object references should be the exception rather than the norm.
* Whenever a class manages its own memory, the programmer should be alert for memory leaks.
* One common source of memory leaks is caches. If the desired lifetime of a cache is determined by external references to the key, consider using a WeakHashMap.
* Another common source of memory leaks is listeners and other callbacks. If you have an API where clients can register themselves and do not de-register themselves, then you have a potential for a memory leak.
* Use a heap profiler to catch memory leaks.

### Item 7: Avoid finalizers

* Finalizers are unpredictable, dangerous and generally unnecessary.
* Never do anything time critical in a finalizer as the JVM does not guarantee when a finalizer would be called.
* Never depend on a finalizer to update critical persistent state as the JVM does not guarantee that a finalizer would be called.
* There is a *severe* performance penalty for using finalizers.
* Provide an explicit termination method and ask clients to invoke it. Use this in conjunction with try-finally blocks.
* The finalizer can act as a safety net in case the owner forgets to call the explicit termination method. In this case, the finalizer must log a warning if it finds that the resource has not yet been terminated.
* If you need to associate a finalizer with a public, non-final class, then use a finalizer guardian, so that finalization can take place even if a subclass finalizer fails to invoke super.finalize().
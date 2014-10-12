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

----

# Chapter 3 - Methods Common to All Objects

### Item 8: Obey the general contract when overriding equals()
Do not override equals if any of the following conditions apply:

* Each instance of the class in inherently unique.
    * This is true for classes such as Thread that represent active entities rather than values.
* You don’t care whether the class provides a logical equality test.
    * For example, java.util.Random could have overridden equals to check whether two Random instances would produce the same sequence of random numbers going forward, but the designers didn’t think that clients would need or want this functionality.
* A superclass has overridden equals and the super class behavior is appropriate for this class as well.
* The class is private or package-private and you are certain that its equals method will never be invoked.

When you override the equals method, you must adhere to its general contract. The equals method implements an equivalence relation. It is:

* **Reflexive:** For any non-null reference value x, x.equals(x) must return true.
* **Symmetric:** For any non-null reference values x and y, x.equals(y) must return true if and only if y.equals(x) returns true.
* **Transitive:** For any non-null reference values x, y, z, if x.equals(y) returns true and y.equals(z) returns true, then x.equals(z) must return true.
* **Consistent:** For any non-null reference values x and y, multiple invocations of x.equals(y) consistently return true or consistently return false, provided no information used in equals comparisons on the objects is modified.
* For any non-null reference value x, x.equals(null) must return false.

There is no way to extend an instantiable class and add a value component while preserving the equals contract.
Here is a recipe for a high-quality equals() method:

* Do not write an equals() method that depends on unreliable resources. 
    * For example, java.net.URL’s equals method relies on comparison of the IP addresses of the hosts associated with the URLs. Translating a hostname to an IP address can require network access, and it isn’t guaranteed to yield the same results over time.
* Use the == operator to check if the argument is a reference to this object.
    * If so, return true. This is just a performance optimization, but one that is worth doing if the comparison is potentially expensive.
* Use the instanceof operator to check if the argument has the correct type.
    * If not, return false. Typically, the correct type is the class in which the method occurs. Occasionally, it is some interface implemented by this class.
* Cast the argument to the correct type.
* For each significant field in the class, check if that field of the argument matches the corresponding field of the object.
    * The special treatment of float and double fields is made necessary by the existence of Float.NaN, -0.0f and the analogous double constants; see the Float.equals documentation for details.
* When you are finished writing your equals() method, ask yourself three questions: Is it symmetric? Is it transitive? Is it consistent?
    * And don’t just ask yourself; write unit tests to check that these properties hold!
* Always override hashCode when you override equals.
* Don’t substitute another type for Object in the equals declaration.
```java
  public boolean equals(MyClass o) {…}
```

### Item 9: Always override hashCode() when you override equals()

* Equal objects must have equal hashcodes. Hence you must override hashcode in every class that overrides equals.

```java
    // The worst possible legal hash function - never use!
    @Override public int hashCode() { return 42; }
```

* It’s legal because it ensures that equal objects have the same hash code. It’s atrocious because it ensures that every object has the same hash code. Therefore, every object hashes to the same bucket, and hash tables degenerate to linked lists. Programs that should run in linear time instead run in quadratic time.


### Item 10: Always override toString()

* Providing a good toString implementation makes your class pleasant to use.
* When practical, the toString() method should return all of the interesting information contained in the object.
* Document the format of your toString() representation or mention that clients should not rely on the format remaining unchanged.
* Provide programmatic access to all the information contained in the value returned by toString().

### Item 11: Override clone() judiciously

* Cloneable represents a highly atypical use of an interface.  If a class implements Cloneable, we are modifying the behavior of a protected method (ie clone) on a superclass (ie Object).
* If you override the clone method in a non-final class, then you should return an object obtained by invoking super.clone.
    * If all of a class’s super-classes obey this rule, then invoking super.clone will eventually invoke Object’s clone method, creating an instance of the right class.
* A class that implements Cloneable is expected to provide a public clone method.
* In order to make a class cloneable, it may be necessary to remove final modifiers from some fields, because clone would be prohibited from assigning a new value to those fields.
* Make sure you do a deep copy of your mutable fields.
* In effect, the clone()method functions as another constructor; you must ensure that it does not harm to the original object and that it properly establishes invariants on the clone.
* Public clone methods should omit it because methods that don’t throw checked exceptions are easier to use (Item 59).
* If you extend a class that implements Cloneable, you have little choice but to implement a well-behaved clone method. Otherwise, you are better off providing an alternative means of object copying, or simply not providing the capability (e.g. immutable classes). 
*  A fine approach to object copying is to provide a copy constructor or copy factory.

### Item 12: Consider implementing Comparable

* The Comparable interface has a sole method named compareTo.
* Implementing Comparable allows operating your class within the scope of many Collections in a useful manner.
* Try to make your compareTo consistent with equals.
* There is no way to extend an instantiable class with a value component while preserving the compareTo contract.
* Use Double.compare and Float.compare instead of the signs < and >.
* Remember if A is a large positive int and B is a large negative int, then A-B would overflow and return a negative integer.

----

#Chapter 4 -Classes and Interfaces

### Item 13: Minimize the accessibility of classes and members
The rule of thumb is simple: make each class or member as inaccessible as possible. In other words, use the lowest possible access level you can.


* Instance fields should never be public.
    * If an instance field is non-final, or is a final reference to a mutable object, then by making the field public, you give up the ability to limit the values that can be stored in the field.
* Classes with public mutable fields are not thread-safe.
    * Instance field is public, you give up the ability to take any action when the field is modified.
* Public static final fields should be only used for constants.
* Ensure that objects referenced by public static final fields are immutable.
* Never have a static final array as public since non-empty arrays are always mutable. Either clone the array or use Collections.unmodifiableList(Arrays.asList()) method.


### Item 14: In public classes, use accessor methods, not public fields

* Provide accessor (getter) and mutator (setter) methods for all public classes.
* For private or package-private classes you may or may not provide accessors/mutators.

### Item 15: Minimize mutability
Immutable classes are easier to design, implement, and use than mutable classes. They are less prone to error and are more secure.

To make a class immutable, follow the following five rules


1. Do not provide any methods to change the object’s state.
1. Ensure that the class cannot be extended.
    * Preventing subclassing is generally accomplished by making the class final, but there are alternatives such as putting a static factory instead of a public constructor etc.
1. Make all fields final.
1. Make all fields private.
1. Ensure exclusive access to any mutable components received in constructor through defensive copying.
    * If your class has any fields that refer to mutable objects, ensure that clients of the class cannot obtain references to these objects. Never initialize such a field to a client-provided object reference or return the object reference from an accessor. Make defensive copies (Item 39) in constructors, accessors, and readObject methods.

---
* Make the immutable class to be final.
    * The alternative to making an immutable class final is to make all of its constructors private or package-private, and to add public static factories in place of the public constructors.
* Immutable objects are inherently thread-safe; they require no synchronization./
* So immutable objects can be shared freely.
* Static factory is a good way for immutable class.
* The only real disadvantage of immutable classes is that they require a separate object for each distinct value. 
* Classes should be immutable unless there is a very good reason to make them mutable.
    * Resist the urge to write a set method for every get method.
* Immutable classes provide many advantages, and their only disadvantage is the potential for performance problems under certain circumstances.
* If a class cannot be made immutable, limit its mutability as much as possible.
* Make every field final unless there is a compelling reason to make it non final.
* Constructors should create fully initialized objects with all of their invariants established.
    * Don’t provide a public initialization method separate from the constructor or static factory unless there is a compelling reason to do so.


### Item 16: Favor composition over inheritance

Inheritance is a powerful way to achieve code reuse, but it is not always the best tool for the job. Used inappropriately, it leads to fragile software. It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension. Inheriting from ordinary concrete classes across package boundaries, however, is dangerous.

* Unlike method invocation, inheritance violates encapsulation.
    * In other words, a subclass depends on the implementation details of its superclass for its proper function. The superclass’s implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched.
    * A related cause of fragility in subclasses is that their superclass can acquire new methods in subsequent releases. This works fine until a new method capable of inserting an element is added to the superclass in a subsequent release. Once this happens, it becomes possible to add an “illegal” element merely by invoking the new method, which is not overridden in the subclass.
    * If the superclass acquires a new method in a subsequent release and you have the bad luck to have given the subclass a method with the same signature and a different return type, your subclass will no longer compile.
* To avoid all of the problems described earlier, you should use composition design. Instead of extending an existing class, give your new class a private field that references an instance of the existing class.
* The disadvantages of wrapper classes are few. One caveat is that wrapper classes are not suited for use in callback frameworks, where in objects pass self references to other objects for subsequent invocations (“callbacks”). Because a wrapped object doesn’t know of its wrapper, it passes a reference to itself (this) and callbacks elude the wrapper. This is known as the *SELF problem*.
* Inheritance is appropriate only in circumstances where the subclass really is a *subtype* of the superclass. 
    * If you are tempted to have a class B extend a class A, ask yourself the question: Is every B really an A? If you cannot truthfully answer yes to this question, B should not extend A.
    * There are a number of obvious violations of this principle in the Java platform libraries. For example, a stack is not a vector, so Stack should not extend Vector. Similarly, a property list is not a hash table, so Properties should not extend Hashtable. In both cases, composition would have been preferable.
* Inheritance propagates any flaws in the superclass’s API, while composition lets you design a new API that hides these flaws.

### Item 17: Design and document for inheritance or else prohibit it 

* The class must document or simply prohibit its *self-use* of overridable methods.
* The only way to test a class designed for inheritance is to write subclasses.
    * that three subclasses are usually sufficient to test an extendable class. One or more of these subclasses should be written by someone other than the superclass author.
* You must test your class by writing subclasses before you release it.
* Constructors (as well as clone and readObject methods) must not invoke overridable methods, directly or indirectly.
    * The superclass constructor runs before the subclass constructor, so the overriding method in the subclass will get invoked before the subclass constructor has run. If the overriding method depends on any initialization performed by the subclass constructor, the method will not behave as expected.
* Neither clone() nor readObject() may invoke an overridable method, directly or indirectly, as they behave like constructors.
* The Cloneable and Serializable interfaces present special difficulties when designing for inheritance. It is generally not a good idea for a class designed for inheritance to implement either of these interfaces, as they place a substantial burden on programmers who extend the class.
* Designing a class for inheritance places substantial limitations on the class.
* The best solution to this problem is to prohibit subclassing in classes that are not designed and documented to be safely subclassed. Prohibit subclassing in classes that are not designed and documented to be safely subclassed by “final” or making the constructor “private” or “package private” and to add public *static* factories in place of the constructors.
* If you feel that you must allow inheritance from such a class, one reasonable approach is to ensure that the class never invokes any of its overridable methods and to document this fact. In other words, eliminate the class’s self-use of overridable methods entirely.
    * To eliminate a class’s self-use of overridable methods, move the body of each overridable method to a private “helper method” and have each overridable method invoke its private helper method. Then replace each self-use of an overridable method with a direct invocation of the overridable method’s private helper method.

### Item 18: Prefer interfaces to abstract classes

Interfaces are usually the best way to implement a type. Their advantages are as follows:

1. Existing classes can easily be retrofitted to implement a new interface.
    * All you have to do is add the required methods if they don’t yet exist and add an implements clause to the class declaration.
1. Interfaces are ideal for defining mixins. 
    * Mixin is a type that a class can implement in addition to its primary type to show that it can provide additional behaviour (e.g. Comparable interface).
1. Interfaces allow the construction of non-hierarchical type frameworks. If you use abstract classes, you risk a combinatorial explosion of classes to take care of each choice.
1. Interfaces enable safe, powerful functionality enhancements via the *wrapper class* idiom, described in Item 16.
    * If you use abstract classes to define types, you leave the programmer who wants to add functionality with no alternative but to use inheritance. The resulting classes are less powerful and more fragile than wrapper classes.

To benefit from both options:

* You can combine the virtues of interfaces and abstract classes by providing an abstract skeletal implementation class to go with each nontrivial interface that you export.
    * The interface still defines the type, but the skeletal implementation takes all of the work out of implementing it.
    * A minor variant on the skeletal implementation is the simple implementation, which is like a skeletal implementation in that it implements an interface and is designed for inheritance, but it differs in that it isn’t abstract: it is the simplest possible working implementation.
* Check the Adapter pattern in GoF book using an anonymous inner class to convert an int array to an Integer list.
* The class implementing an interface can forward invocations of interface methods to a contained instance of a private inner class that extends the skeletal implementation.  This technique is called simulated multiple inheritance.
* Abstract classes the only advantage over interfaces in that they are easier to evolve.  In general, once an interface is released and widely implemented, it is almost impossible to change.
    * If, in a subsequent release, you want to add a new method to an abstract class, you can always add a concrete method containing a reasonable default implementation. All existing implementations of the abstract class will then provide the new method. This does not work for interfaces. It is, generally speaking, impossible to add a method to a public interface without breaking all existing classes that implement the interface.


### Item 19: Use interfaces only to define types

* The constant interface pattern is a poor use of interfaces.
    * To avoid writing out the constants, some programmers put in their constants in interfaces. This is the so-called constant interface anti-pattern. An interface should say something about what a client can do with instances of the class. It is inappropriate to define an interface for any other purpose.
    * Enum and Utility class are more appropriate for defining constants and using the *static import* feature.

### Item 20: Prefer class hierarchies to tagged classes
One sometimes sees a class whose instances come in two or more flavors.  These are handled by multiple switch statements and by having data corresponding to each flavor within the same class.

Such classes are called tagged classes. These are verbose, error-prone and inefficient.

* Replace tagged classes with a single data type capable of representing objects of multiple flavors: sub-typed classes.

### Item 21: Use function objects to represent strategies
Features like function pointers, delegates, lambda expressions call functions in functions.

Strategy pattern is built on these features typically on function pointers.

A stateless class with only one method can represent a function object.

The functionality of function-pointers is replicated via the Strategy interface in Java. 

Declare an interface to represent a strategy and a class that implements this interface for each concrete strategy. 

If the strategy is used once, use an anonymous inner class. Otherwise put it as a private static member in a class and export via a static final field whose type is the Strategy interface.

### Item 22: Favor static member classes over nonstatic
A nested class (a class defined inside another class) is classified into static member, non-static member, anonymous and local classes.

* Static member classes are used as public helper classes.
* If you declare a member class that does not require access to an enclosing instance, *always* put the static modifier.
* Anonymous classes are used in function objects, process objects, and in static factory methods.
    * Three common uses of anonymous classes:
        1. function objects
        1. process objects, such as Runnable, Thread
        1. static factory methods
* Local classes are used in place of anonymous classes, if the class instantiation is done from multiple locations.

----

#Chapter 5 - Generics


### Item 23: Don’t use raw types in new code

* Do not use raw type to define variable, as the compiler will not check the type of parameters and it loses the type safety and may generate run-time error.
* If you use raw types, you lose all the safety and expressiveness benefits of generics.
    * You lose type safety if you use a raw type like List, but not if you use a parameterized type like List < Object >.
* Suppose the element is unknown and does not matter. Even then do not use raw types. Instead use the wildcard ? as in List<?>. Unbounded wildcard types like List<?> is capable to hold one of any type of element with type safety.
* You cannot put any element other than null into a Collection<?>.
* Generic type information is erased at runtime.
* Since generic type information is erased at run-time, there are two places where raw types must be used.
    1. Use raw types in class literals, such as List.class.
    1. Use raw types while using the instanceOf operator.

To reiterate, Set< Object > is a parameterized type representing a set that can contain objects of any type. Set<?> is a wildcard type representing a set that can contain only objects of unknown type. Set is a raw type, which opts out of the generic type system. The first two are safe; the third is not.

### Item 24: Eliminate unchecked warnings

* Eliminate every unchecked warning you can.
* If you can’t eliminate a warning, and you can prove that the code that provoked the warning is type safe, then (and only then) suppress the warning with an @SuppressWarnings(“unchecked”) annotation.
* Always use the @SuppressWarnings annotation on the smallest scope possible.
* Every time you use an @SuppressWarnings(“unchecked”) - annotation, add a comment saying why its safe to do so.
    * If you find it hard to write such a comment, keep thinking. You may end up figuring out that the unchecked operation isn’t safe after all.
* It is illegal to put a SuppressWarnings annotation on a return statement. Create a local variable to hold the return value and annotate that with the @SuppressWarnings annotation.

### Item 25: Prefer lists to arrays
Arrays are covariant, i.e. if Sub is a subtype of Super, then Sub[] is also a subtype of Super[]. This leads to run-time errors if one is not careful.

Arrays provide runtime type safety but not compile-time type safety and vice versa for generics.

```java
    // Fails at runtime!
    Object[] objectArray = new Long[1];
    objectArray[0] = “I don’t fit in”; // Throws ArrayStoreException
    
    // Won’t compile!
    List<Object> ol = new ArrayList<Long>(); // Incompatible types
    ol.add(“I don’t fit in”);
```

Generics enforce their type constraints only at compile time and discard (or erase) their element type information at runtime. An implication of this is that arrays and generics do not mix well.

It is illegal to create an array of a generic type such as “new List< String >[]” and “new List<E>[]“. Technically, we say that List< String >, List< E > and E are non-reifiable since their run-time representation carries less information than their compile-time representation. The only parameterized types that are reifiable are unbounded wildcard types, such as List<?> and Map<?,?>.

To remove errors/warnings coming about by mixing arrays and generics, consider replacing arrays with lists.

### Item 26: Favor generic types

Generics are safer. Suppress the warning when casting.
```java
    @SuppressWarnings(“unchecked”)
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
```

### Item 27: Favor generic methods
Remember that the order in which the set’s elements are printed out is implementation-dependent.

To ease writing  of parameterized type instance creation code, the author recommends a static factory. We get the same benefit by invoking Lists or other appropriate class from org.apache.commons. or org.google.commons library.

Recursive type bound refers to code like “T extends Comparable<T>”.  This refers to all types that can be compared to another instance of their own type.

Generic singleton factory:
```java  
    // Generic singleton factory pattern
    private static UnaryFunction<Object> IDENTITY_FUNCTION = new UnaryFunction<Object>()
    {
        public Object apply(Object arg) { return arg; }
    };
    
    // IDENTITY_FUNCTION is stateless and its type parameter is
    // unbounded so it’s safe to share one instance across all types.
    @SuppressWarnings(“unchecked”)
    public static <T> UnaryFunction<T> identityFunction() {
        return (UnaryFunction<T>) IDENTITY_FUNCTION;
    }
```
Recursive type bound is usually used for Comparable<T>.
```java
    public interface Comparable<T>; {
        int compareTo(T o);
    }
```

```java
    // Using a recursive type bound to express mutual comparability
    public static <T extends Comparable<T> T max(List<T> list) {…}
```

### Item 28: Use bounded wildcards to increase API flexibility
A common mnemonic is PECS  or producer-extends, consumer-super.

* A type does not extend itself; yet it is a subtype of itself.
* For maximum flexibility, use wildcard types on input parameters that represent producers or consumers.
* Do not use wildcard types as return types. Rather than providing additional flexibility for your users, it would force them to use wildcard types in client code.
* If the user of a class has to think about wildcard types, there is probably something wrong with the class’s API.
* When invoking a method, you might have to something explicitly mention the type parameter. Example, Union.<Number>union(integers, doubles).
* Always use Comparable<? super T> in preference to Comparable<T>.
* Always use Comparator<? super T> in preference to Comparator<T>.
* If a type parameter appears only once in a method declaration, replace it with a wildcard.
* Can’t put any value except null into a List<?>, which means List of some particular type.

### Item 29: Consider type safe heterogeneous containers
Given Class< T > type as an input argument to a method, the method can return type.cast(someMap.get(type)).  The someMap is defined as a Map< Class < ? >, Object >.  Such a map is called a type-safe heterogeneous container.

When a class literal is passed among methods to communicate both compile-time and runtime type information, it is called a type token. The use of type.cast above is called dynamic casting.

Place the type parameter on the key rather than the container to enable the container to include different types.

```java
    // Typesafe heterogeneous container pattern - implementation
    public class Favorites {
    
        private Map<Class<?>, Object> favorites = new HashMap<Class<?>, Object>();
        
        public <T> void putFavorite(Class<T> type, T instance) {
            if (type == null)
                throw new NullPointerException(“Type is null”);
            
            favorites.put(type, instance);
        }
        
        public <T> T getFavorite(Class<T> type) {
            return type.cast(favorites.get(type));
        }
    }
```

If you are using either third-party code or legacy code that you suspect of not using generics, consider wrapping your collections with Collection.checked methods. Example would be:


```java
    Set<Integer> setOfInts = new HashSet<Integer>();
    setOfInts = Collections.checkedSet(setOfInts, Integer.class);
```

Now even if someone copied a reference of setOfInts to a plain Set-instance, they would not be able to insert non-integer items into it.

----

# Chapter 6 - Enums and Annotations

### Item 30: Use enums instead of int constants

* Using int/string to simulate enums is called the int/string enum pattern. It is always a bad idea.
* int enums are compile-time constants; and it is not very helpful to have numbers printed out while debugging.
* In other languages like C/C++, enums are essentially int values. But in Java, they are full-fledged classes.
* Enum types provide compile-time type safety. Each type has its own namespace.
* To associate data with enum constants, declare instance fields and write a constructor that takes the data and stores it in the fields.
* If you want to associate a different behavior with each instance, then declare an abstract method in the enum type, and then override it with a concrete method for each constant in a constant-specific class body.  Such methods are called constant-specific method implementation.
* Consider including a *fromString()* method on your enum types.
* *valueOf(String)* method can translate a constant’s name into the constant itself.
* Enums should not switch on their values to share code. Instead use the strategy enum method.
* Switches on enums are good for augmenting external enums with constant-specific behaviour.
* Consider the strategy enum pattern if multiple enum constants share common behaviors.

```java
Enums can also add methods or fields.

    // Enum type with constant-specific method implementations
    public enum Operation {
        PLUS { double apply(double x, double y){return x + y;} },
        MINUS { double apply(double x, double y){return x - y;} },
        TIMES { double apply(double x, double y){return x * y;} },
        DIVIDE { double apply(double x, double y){return x / y;} };
    
        abstract double apply(double x, double y);
    }
```

### Item 31: Use instance fields instead of ordinals

* Never derive a value associated with an enum from its ordinal; store it in an instance field instead.
* Ordinal would change if you re-order your instances or include additional ones in the middle.

### Item 32: Use EnumSet instead of bit fields

* The EnumSet class can efficiently represent sets of values drawn from a single enum type. Use EnumSet.of(YourEnumName.A, YourEnumName.B).
* Just because an enumerated type is used in sets, there is no reason to represent it in bit fields.
* If you wrap it in Collections.unmodifiableSet, it would become immutable but would result in a noticeable performance cost.

```java
    // EnumSet - a modern replacement for bit fields
    public class Text {
    
        public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
        
        // Any Set could be passed in, but EnumSet is clearly best
        public void applyStyles(Set<Style> styles) { … }
    }
```

### Item 33: Use EnumMap instead of ordinal indexing

* It is rarely appropriate to use ordinal() to index arrays, use an EnumMap or a nested EnumMap.
    * If the relationship that you are representing is multidimensional, use EnumMap<…, EnumMap<…>. This is a special case of the general principle that application programmers should rarely, if ever, use Enum.ordinal.

```java
        // Using an EnumMap to associate data with an enum
        
        Map<Herb.Type, Set<Herb> herbsByType = new EnumMap<Herb.Type, Set<Herb>(Herb.Type.class);
        
        for (Herb.Type t : Herb.Type.values()) {
            herbsByType.put(t, new HashSet<Herb>());
        }
        
        for (Herb h : garden) {
            herbsByType.get(h.type).add(h);
        }
        
        System.out.println(herbsByType);
```

### Item 34: Emulate extensible enums with interfaces

* While you can not write an extensible enum type, you can emulate it by writing an interface to go with a basic enum type that implements the interface.

*< T extends Enum < T > & InterfaceName >* refers to a type that is both an enum-type as well as implementing an interface. To access the enumConstants from a class-name, use the getEnumConstants() method.


### Item 35: Prefer annotations to naming patterns

* There is simply no reason to use naming patterns now that we have annotations.
* All programmers should, however, use the predefined annotation types provided by the Java platform.
* Annotations do not directly affect program semantics, but they do affect the way programs are treated by tools and libraries, which can in turn affect the semantics of the running program. Annotations can be read from source files, class files, or reflectively at run time.
* Annotations complement javadoc tags. In general, if the markup is intended to affect or produce documentation, it should probably be a javadoc tag; otherwise, it should be an annotation.
* Annotations often need meta-annotations like @Retention and @Target.
* A marker annotation has no parameters.

### Item 36: Consistently use the @Override annotation

* Use the @Override annotation on every method declaration that you believe to override a superclass declaration, except for the abstract methods or methods of interfaces.
    * The compiler can protect you from a great many errors if you use the Override annotation on every method declaration that you believe to override a supertype declaration, with one exception. In concrete classes, you need not annotate methods that you believe to override abstract method declarations (though it is not harmful to do so).

### Item 37: Use marker interfaces to define types

* If you want to define a type that that does not have any new methods associated with it, then use a marker interface.
* Marker interfaces define a type that is implemented by instances of the marked class; marker annotations do not.
    * The existence of this type allows you to catch errors at compile time that you couldn’t catch until runtime if you used a marker annotation.
* If you want to mark program elements other than classes/interfaces, or allow for the possibility of adding more information to the marker in the future, or fit the marker into a framework making heavy use of annotations, then use a marker annotation.
* If you find yourself writing a marker annotation type whose target is ElementType.TYPE, take the time to figure out whether it really should be an annotation type, or whether a marker interface would be more appropriate.

The typical marker interface is Serialize interface.
## Chapter 4 - Classes and Interfaces

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
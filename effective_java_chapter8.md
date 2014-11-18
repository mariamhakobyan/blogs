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
## Chapter 3 - Methods Common to All Objects

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
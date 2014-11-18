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
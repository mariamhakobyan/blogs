## Chapter 6 - Enums and Annotations

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
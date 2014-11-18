## Chapter 5 - Generics


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
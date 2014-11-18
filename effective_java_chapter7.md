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
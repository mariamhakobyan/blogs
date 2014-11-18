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
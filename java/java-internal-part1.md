# Java Core Fundamentals - Complete Study Guide

# Table of Contents

1. Structure of Main Method
2. JVM Startup Flow
3. Why Main Method is Static
4. Why Main Method Returns Void
5. Command Line Arguments
6. Abstraction in Java
7. Abstract Classes
8. Interfaces in Java
9. Interface vs Abstract Class
10. Functional Interfaces
11. String Immutability
12. String Pool Internals
13. Why String is Immutable
14. String vs StringBuilder vs StringBuffer
15. Interview Questions
16. Cheat Sheet

---

# 1. Structure of Main Method

Every Java application starts execution from:

```java
public static void main(String[] args)
{
    // code
}
```

This is known as:

```text
Entry Point of Java Application
```

---

# Complete Syntax Breakdown

```java
public static void main(String[] args)
```

Contains:

```text
public
static
void
main
String[] args
```

---

# 2. JVM Startup Flow

Suppose:

```java
public class Application {

    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

Execution Flow:

```text
javac Application.java
       ↓
Application.class
       ↓
JVM Starts
       ↓
Class Loader Loads Class
       ↓
JVM Searches
       ↓
public static void main()
       ↓
Invokes Main Method
       ↓
Program Starts
```

---

# 3. Why Main Method is Static

Main method executes before any object exists.

---

If main were non-static:

```java
public void main(String[] args)
```

JVM would need:

```java
new Application()
```

before calling main.

Problem:

```text
Who creates the first object?
```

Circular dependency.

---

Because of this:

```java
public static void main(...)
```

allows JVM to call:

```java
Application.main(...)
```

directly.

---

Without object creation.

---

# 4. Why Main Method is Public

JVM exists outside the class.

To invoke main:

```java
Application.main()
```

method must be accessible.

---

If private:

```java
private static void main(...)
```

JVM cannot call it.

Compilation succeeds.

Execution fails.

---

# 5. Why Main Method Returns Void

JVM does not expect any value back.

---

Correct:

```java
public static void main(String[] args)
```

---

Invalid:

```java
public static int main(String[] args)
```

JVM won't recognize it.

---

# 6. Why String[] args?

Used to receive command-line arguments.

Example:

```bash
java Application Anshi Java SDE2
```

---

Received as:

```java
args[0] = "Anshi"
args[1] = "Java"
args[2] = "SDE2"
```

---

Example:

```java
public class Test {

    public static void main(String[] args) {

        for(String arg : args)
        {
            System.out.println(arg);
        }
    }
}
```

Output:

```text
Anshi
Java
SDE2
```

---

# Alternative Syntaxes

All valid:

```java
String args[]
```

```java
String... args
```

```java
String[] x
```

Only method signature matters.

---

# 7. Abstraction in Java

Abstraction means:

```text
Showing essential details
while hiding implementation details.
```

---

Example

Driving a car.

You know:

```text
Accelerator
Brake
Steering
```

You don't know:

```text
Fuel Injection Logic
Engine Timing
Combustion Process
```

Hidden.

---

This is:

```text
Abstraction
```

---

Java provides abstraction through:

```text
Abstract Classes
Interfaces
```

---

# 8. Abstract Class

Declared using:

```java
abstract class
```

---

Example

```java
abstract class Vehicle {

    abstract void start();

    void stop() {
        System.out.println("Vehicle Stopped");
    }
}
```

---

Child Class

```java
class Car extends Vehicle {

    @Override
    void start() {
        System.out.println("Car Started");
    }
}
```

---

Usage

```java
Vehicle v = new Car();
v.start();
```

Output:

```text
Car Started
```

---

# Characteristics

Abstract class can have:

```text
Abstract Methods
Concrete Methods
Constructors
Instance Variables
Static Methods
```

---

Cannot instantiate directly.

```java
new Vehicle(); // Error
```

---

# 9. Interfaces in Java

Interface defines:

```text
Contract
```

---

Example

```java
interface Payment {

    void pay();
}
```

---

Implementation

```java
class CreditCardPayment
implements Payment {

    public void pay() {
        System.out.println("Paid");
    }
}
```

---

Usage

```java
Payment p =
new CreditCardPayment();

p.pay();
```

---

# Why Interfaces?

Loose Coupling.

Programming to abstraction.

---

Bad

```java
CreditCardPayment payment =
new CreditCardPayment();
```

---

Good

```java
Payment payment =
new CreditCardPayment();
```

Tomorrow:

```java
Payment payment =
new UpiPayment();
```

No code changes elsewhere.

---

# 10. Interface Evolution (Java 8+)

Before Java 8:

```text
Only Abstract Methods
```

---

After Java 8

Interfaces can contain:

```java
default methods
```

```java
interface Payment {

    default void validate() {
        System.out.println("Validate");
    }
}
```

---

Static Methods

```java
interface Payment {

    static void helper() {
        System.out.println("Helper");
    }
}
```

---

Java 9+

Private methods also supported.

---

# 11. Interface vs Abstract Class

| Feature | Interface | Abstract Class |
|----------|-----------|-----------|
| Multiple Inheritance | Yes | No |
| Constructors | No | Yes |
| Instance Variables | No | Yes |
| Abstract Methods | Yes | Yes |
| Concrete Methods | Yes (default) | Yes |
| State | No | Yes |
| Use Case | Contract | Shared Behavior |

---

# Interview Answer

Use Interface when:

```text
Need Contract
Need Multiple Inheritance
Need Loose Coupling
```

Use Abstract Class when:

```text
Need Shared State
Need Common Logic
Need Partial Implementation
```

---

# 12. String Immutability

One of the most important Java interview topics.

---

Definition

```text
Once a String object is created,
its value cannot be changed.
```

---

Example

```java
String s = "Java";

s.concat(" Programming");

System.out.println(s);
```

Output:

```text
Java
```

Not:

```text
Java Programming
```

---

Why?

Because:

```java
concat()
```

creates a new object.

---

Example

```java
String s = "Java";

s = s.concat(" Programming");
```

Output

```text
Java Programming
```

---

# Internal Working

```java
String s = "Java";
```

Creates:

```text
String Object
```

Memory:

```text
"Java"
```

---

Now:

```java
s.concat(" Programming");
```

Creates:

```text
"Java Programming"
```

New object.

Original remains unchanged.

---

# 13. String Pool Internals

Special memory area:

```text
String Constant Pool (SCP)
```

---

Example

```java
String s1 = "Java";
String s2 = "Java";
```

Memory:

```text
Pool

"Java"
```

Only one object.

Both references point to same object.

---

Verification

```java
System.out.println(s1 == s2);
```

Output:

```text
true
```

---

Now:

```java
String s3 = new String("Java");
```

Creates:

```text
Heap Object
```

Outside pool.

---

Comparison

```java
s1 == s3
```

Output:

```text
false
```

Different objects.

---

# 14. Why String is Immutable

Most asked interview question.

---

Reason 1: Security

Used heavily in:

```text
Database URLs
File Paths
Network Connections
Class Loading
```

---

Imagine:

```java
String url =
"jdbc:mysql://prod-db";
```

If mutable:

```java
"HackedDB"
```

could replace it.

Security risk.

---

Reason 2: String Pool

Pooling works only because Strings never change.

---

Example

```java
String s1 = "Java";
String s2 = "Java";
```

Same object shared safely.

---

If mutable:

```java
s1 changes
```

then:

```java
s2 changes
```

Unexpected behavior.

---

Reason 3: Thread Safety

Immutable objects are inherently:

```text
Thread Safe
```

Multiple threads can read simultaneously.

No synchronization required.

---

Reason 4: HashMap Performance

Strings frequently used as keys.

Example

```java
Map<String,Integer>
```

HashCode cached.

---

If String mutable:

```java
key changes
```

HashMap lookup breaks.

---

# 15. String vs StringBuilder vs StringBuffer

---

## String

Immutable.

```java
String s = "Java";
```

---

Every modification:

```text
Creates New Object
```

---

## StringBuilder

Mutable.

Not Thread Safe.

Fast.

```java
StringBuilder sb =
new StringBuilder();
```

---

Append:

```java
sb.append("Java");
```

Same object modified.

---

## StringBuffer

Mutable.

Thread Safe.

Slower.

```java
StringBuffer sb =
new StringBuffer();
```

Uses synchronization.

---

# Comparison

| Feature | String | StringBuilder | StringBuffer |
|----------|----------|----------|----------|
| Mutable | No | Yes | Yes |
| Thread Safe | Yes | No | Yes |
| Performance | Slowest | Fastest | Moderate |
| Memory Efficient | No | Yes | Yes |

---

# Interview Rule

Single Threaded

```java
StringBuilder
```

---

Multi Threaded

```java
StringBuffer
```

---

Constant Text

```java
String
```

---

# 16. Common Interview Questions

---

## Why Main Method Static?

```text
JVM must invoke it without
creating an object.
```

---

## Can We Overload Main Method?

Yes.

```java
main()
main(int x)
```

Allowed.

---

But JVM always calls:

```java
main(String[] args)
```

---

## Can Interface Have Methods?

Java 8+

```text
Abstract
Default
Static
Private
```

---

## Why String Immutable?

```text
Security
String Pool
Thread Safety
HashMap Performance
```

---

## Difference Between == and equals()

```java
==
```

compares:

```text
References
```

---

```java
equals()
```

compares:

```text
Content
```

---

# 17. Cheat Sheet

## Main Method

```java
public static void main(String[] args)
```

---

## Why Static?

```text
JVM invokes without object creation.
```

---

## Abstraction

```text
Hide implementation.
Expose behavior.
```

---

## Interface

```text
Contract
Multiple Inheritance
Loose Coupling
```

---

## Abstract Class

```text
Partial Abstraction
Shared State
Common Logic
```

---

## String

```text
Immutable
Thread Safe
Pool Friendly
```

---

## StringBuilder

```text
Mutable
Fast
Not Thread Safe
```

---

## StringBuffer

```text
Mutable
Thread Safe
Synchronized
```

---

# Golden Interview Answer

> Java achieves abstraction primarily through interfaces and abstract classes. Interfaces define contracts and enable loose coupling, while abstract classes provide shared behavior and state. Strings are immutable to ensure security, enable String Pool optimization, guarantee thread safety, and allow efficient usage as HashMap keys. The JVM starts execution through the `public static void main(String[] args)` method because it can be invoked without creating an object instance.

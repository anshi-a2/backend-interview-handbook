# Java OOP (Object-Oriented Programming) - Complete Senior/SDE-2 Study Guide

Level: SDE-2 / Senior Backend Engineer

---

# Table of Contents

1. What is OOP?
2. Why OOP Exists
3. Class vs Object
4. Four Pillars of OOP
   - Encapsulation
   - Abstraction
   - Inheritance
   - Polymorphism
5. Association
6. Aggregation
7. Composition
8. Dependency
9. IS-A vs HAS-A Relationship
10. Method Overloading
11. Method Overriding
12. Dynamic Method Dispatch
13. Runtime Polymorphism Internals
14. Abstract Class Internals
15. Interface Internals
16. Multiple Inheritance Problem
17. Diamond Problem
18. Java 8 Interface Enhancements
19. SOLID Principles
20. OOP Design Trade-offs
21. Common Interview Questions
22. Senior-Level Discussion Points
23. Ultimate Cheat Sheet

---

# 1. What is OOP?

Object-Oriented Programming is a programming paradigm where software is modeled as:

```text
Objects
```

that contain:

```text
State + Behavior
```

---

Example

Car

State:

```text
color
speed
brand
```

Behavior:

```text
start()
stop()
accelerate()
```

---

Java is primarily an:

```text
Object-Oriented Language
```

---

# Why OOP?

Before OOP:

```text
Functions
Global Variables
```

Problems:

```text
Tight Coupling
Poor Reusability
Poor Maintainability
```

---

OOP provides:

```text
Encapsulation
Abstraction
Reusability
Extensibility
Maintainability
```

---

# 2. Class vs Object

---

## Class

Blueprint.

Example

```java
class Employee {

    int id;

    String name;
}
```

---

Class itself occupies memory only for:

```text
Metadata
```

inside JVM.

---

No actual employee exists yet.

---

## Object

Instance of class.

```java
Employee emp =
new Employee();
```

---

Memory allocated in:

```text
Heap
```

---

Visualization

```text
Class
  ↓
Blueprint

Object
  ↓
Real Entity
```

---

# 3. Memory Representation

```java
Employee emp =
new Employee();
```

---

Stack

```text
emp
 |
 ↓
0x101
```

---

Heap

```text
0x101

id = 0
name = null
```

---

Object reference stored in stack.

Actual object stored in heap.

---

# 4. Four Pillars of OOP

---

# Encapsulation

---

Definition

```text
Binding Data and Methods Together
```

and

```text
Hiding Internal State
```

---

Example

Bad

```java
public int balance;
```

Anyone can modify.

---

Good

```java
private int balance;
```

---

Expose controlled access.

```java
public void deposit()
```

---

Example

```java
class BankAccount {

    private double balance;

    public void deposit(double amount){

        if(amount > 0)
            balance += amount;
    }
}
```

---

Benefits

```text
Data Protection
Validation
Loose Coupling
Maintainability
```

---

Senior Answer

Encapsulation is not merely making fields private.

It is:

```text
Protecting Invariants
```

of the object.

---

Example

Account balance should never become negative.

Encapsulation guarantees this rule.

---

# Abstraction

---

Definition

```text
Hide Implementation
Expose Behavior
```

---

Example

Car

You know:

```text
start()
stop()
```

You don't know:

```text
Fuel Injection
Engine Timing
Spark Mechanism
```

---

Java achieves abstraction using:

```java
Abstract Class
Interface
```

---

Example

```java
interface PaymentService {

    void pay();
}
```

---

Implementation hidden.

```java
class CreditCardPayment
```

---

Senior Perspective

Abstraction reduces:

```text
Coupling
```

between:

```text
Caller
and
Implementation
```

---

Allows implementation changes without affecting clients.

---

# Inheritance

---

Definition

```text
Acquire Properties
from Parent Class
```

---

Example

```java
class Animal {

    void eat(){}
}
```

---

```java
class Dog extends Animal {

}
```

---

Dog gets:

```java
eat()
```

without rewriting.

---

Relationship

```text
IS-A
```

---

Example

```text
Dog IS-A Animal
```

---

Advantages

```text
Code Reuse
Polymorphism
Hierarchy Modeling
```

---

Problems

Excessive inheritance causes:

```text
Tight Coupling
Fragile Hierarchies
```

---

Senior Principle

Prefer:

```text
Composition
```

over

```text
Inheritance
```

when possible.

---

# Polymorphism

---

Definition

```text
One Interface
Multiple Behaviors
```

---

Example

```java
PaymentService service;
```

Can point to:

```java
UPIPayment
```

or

```java
CardPayment
```

---

Same method.

```java
service.pay();
```

Different behavior.

---

Types

```text
Compile-Time Polymorphism
Runtime Polymorphism
```

---

# 5. Method Overloading

Compile-time polymorphism.

---

Example

```java
add(int a,int b)
```

---

```java
add(double a,double b)
```

---

Compiler decides.

---

Based on:

```text
Method Signature
```

---

Not Return Type.

---

Invalid

```java
int test()
```

and

```java
String test()
```

---

Compiler Error.

---

# 6. Method Overriding

Runtime polymorphism.

---

Parent

```java
class Animal {

   void sound(){
      System.out.println("Animal");
   }
}
```

---

Child

```java
class Dog extends Animal {

   @Override
   void sound(){
      System.out.println("Bark");
   }
}
```

---

Execution

```java
Animal a = new Dog();

a.sound();
```

Output

```text
Bark
```

---

Runtime decides.

---

# 7. Dynamic Method Dispatch

Most important OOP internal.

---

Code

```java
Animal a = new Dog();
```

---

Compile Time

Reference Type:

```text
Animal
```

---

Runtime

Actual Object:

```text
Dog
```

---

JVM checks:

```text
Actual Object Type
```

not

```text
Reference Type
```

---

Calls

```java
Dog.sound()
```

---

This is:

```text
Dynamic Dispatch
```

---

Foundation of:

```text
Spring Framework
Strategy Pattern
Dependency Injection
```

---

# 8. Association

Relationship between objects.

---

Example

```text
Teacher ↔ Student
```

---

Independent lifecycles.

---

Visualization

```text
Teacher
   |
Student
```

---

# 9. Aggregation

Weak HAS-A relationship.

---

Example

```text
Department HAS Employees
```

---

Employees can exist independently.

---

Diagram

```text
Department
   ◇
Employee
```

---

If department removed:

```text
Employee survives
```

---

# 10. Composition

Strong HAS-A relationship.

---

Example

```text
House HAS Rooms
```

---

If house destroyed:

```text
Rooms destroyed
```

---

Diagram

```text
House
   ◆
Room
```

---

Most preferred design.

---

Example

```java
class Engine {}

class Car {

    private Engine engine;
}
```

---

# Aggregation vs Composition

| Feature | Aggregation | Composition |
|----------|-------------|-------------|
| Ownership | Weak | Strong |
| Lifecycle | Independent | Dependent |
| Coupling | Lower | Higher |
| Example | Team-Player | House-Room |

---

# 11. Dependency

Temporary relationship.

---

Example

```java
void send(EmailService service)
```

---

Method depends on service.

---

No ownership.

---

Used heavily in:

```text
Spring Dependency Injection
```

---

# 12. IS-A vs HAS-A

---

IS-A

Inheritance.

```text
Dog IS-A Animal
```

---

HAS-A

Composition.

```text
Car HAS-A Engine
```

---

Interview Rule

Prefer:

```text
HAS-A
```

over

```text
IS-A
```

unless true hierarchy exists.

---

# 13. Abstract Class Internals

---

Cannot instantiate.

```java
abstract class Payment
```

---

Can contain

```java
Abstract Methods
Concrete Methods
Fields
Constructors
```

---

Example

```java
abstract class Payment {

    abstract void pay();

    void validate(){
    }
}
```

---

Purpose

```text
Shared State + Shared Behavior
```

---

# 14. Interface Internals

Pure contract.

---

Before Java 8

```text
Only Abstract Methods
```

---

Java 8+

```text
Default Methods
Static Methods
```

---

Java 9+

```text
Private Methods
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
class UPI
implements Payment
```

---

# Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|----------|----------------|------------|
| State | Yes | Constants Only |
| Constructor | Yes | No |
| Multiple Inheritance | No | Yes |
| Use Case | Shared Logic | Contract |

---

# 15. Diamond Problem

Example

```java
A
| \
B  C
 \/
 D
```

---

If

```java
B.test()
```

and

```java
C.test()
```

exist

Which method should D inherit?

---

Ambiguous.

---

Java solves by:

```text
No Multiple Class Inheritance
```

---

Allowed

```java
implements Interface1,
           Interface2
```

---

# Java 8 Default Method Conflict

```java
interface A {
   default test(){}
}
```

---

```java
interface B {
   default test(){}
}
```

---

Class

```java
class C
implements A,B
```

Must override.

---

Reason:

Avoid ambiguity.

---

# 16. SOLID Principles

Most important senior topic.

---

## S

Single Responsibility Principle

One class.

One reason to change.

---

Bad

```java
UserService

Save User
Generate PDF
Send Email
```

---

Good

Separate services.

---

## O

Open Closed Principle

Open for:

```text
Extension
```

Closed for:

```text
Modification
```

---

Use interfaces.

---

## L

Liskov Substitution

Child must replace parent safely.

---

Bad Example

```java
Bird
```

```java
Penguin extends Bird
```

but cannot fly.

Violation.

---

## I

Interface Segregation

Don't force unused methods.

---

Bad

```java
Worker {

  code()
  cook()
}
```

---

Good

Separate interfaces.

---

## D

Dependency Inversion

Depend on:

```text
Abstractions
```

not

```text
Concrete Classes
```

---

Foundation of Spring.

---

# 17. OOP Design Trade-offs

Junior View

```text
Inheritance Everywhere
```

---

Senior View

Prefer

```text
Composition
```

because:

```text
More Flexible
Less Coupling
Easy Testing
```

---

Example

Bad

```java
Car extends Engine
```

---

Good

```java
Car has Engine
```

---

# 18. Common Interview Questions

---

### Why encapsulation?

```text
Protect object state and invariants.
```

---

### Why abstraction?

```text
Reduce coupling.
```

---

### Difference between abstraction and encapsulation?

Encapsulation

```text
Hide Data
```

Abstraction

```text
Hide Complexity
```

---

### Why interfaces preferred?

```text
Loose coupling.
```

---

### Composition vs Inheritance?

Composition preferred.

---

### How runtime polymorphism works?

```text
Dynamic Method Dispatch
```

using actual object type.

---

### Why no multiple inheritance in Java?

```text
Diamond Problem
```

---

# 19. Senior-Level OOP Discussion

A senior engineer should describe OOP as:

```text
A mechanism for managing complexity
through abstraction boundaries.
```

Not merely:

```text
Classes and Objects
```

---

Good OOP design should:

```text
Reduce Coupling
Increase Cohesion
Enable Extensibility
Improve Testability
```

---

In large systems:

```text
Abstraction
Composition
Polymorphism
```

matter significantly more than inheritance.

---

# Ultimate Cheat Sheet

## Encapsulation

```text
Hide Data
Protect Invariants
```

---

## Abstraction

```text
Hide Implementation
Expose Contract
```

---

## Inheritance

```text
IS-A Relationship
Code Reuse
```

---

## Polymorphism

```text
One Interface
Multiple Behaviors
```

---

## Overloading

```text
Compile Time
```

---

## Overriding

```text
Runtime
```

---

## Dynamic Dispatch

```text
Actual Object Decides Method
```

---

## Composition

```text
HAS-A Relationship
Preferred Design
```

---

## Interface

```text
Contract
Loose Coupling
```

---

## Abstract Class

```text
Shared State + Shared Logic
```

---

## SOLID

```text
SRP
OCP
LSP
ISP
DIP
```

---

# Golden SDE-2 Interview Answer

> OOP is not just about classes and objects; it is about managing complexity through abstraction. Encapsulation protects object invariants, abstraction hides implementation details, inheritance models true IS-A relationships, and polymorphism enables extensible behavior through dynamic dispatch. In modern enterprise systems, composition is generally preferred over inheritance because it reduces coupling and improves flexibility. Frameworks like Spring heavily leverage interfaces, dependency inversion, and runtime polymorphism to build scalable and maintainable systems. A senior engineer focuses less on syntax and more on how OOP principles improve extensibility, testability, maintainability, and system evolution over time.

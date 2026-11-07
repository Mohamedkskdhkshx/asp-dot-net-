# C# OOP Interview Questions and Answers

A revision sheet for core Object-Oriented Programming questions in C#
and ASP.NET Core interviews.

## 1. Why use an abstract class instead of creating an object directly from it?

**Answer:**

An abstract class cannot be instantiated directly. We use it to provide
a common base for related classes and define common behavior or abstract
methods that derived classes must implement.

It helps us achieve abstraction and runtime polymorphism.

## 2. What is the difference between Method Overloading and Method Overriding?

**Answer:**

### Method Overloading

-   Same method name but different parameter signatures.
-   Usually happens within the same class, but can also occur across an
    inheritance hierarchy.
-   The compiler resolves the overload based on the arguments.

``` csharp
public class Calculator
{
    public int Add(int x, int y) => x + y;
    public double Add(double x, double y) => x + y;
    public int Add(int x, int y, int z) => x + y + z;
}
```

### Method Overriding

-   A derived class provides a new implementation of an inherited
    method.
-   The base method must be `virtual`, `abstract`, or `override`.
-   The derived method uses the `override` keyword.
-   It enables runtime polymorphism.

``` csharp
public class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Animal sound");
    }
}

public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}
```

## 3. Can we achieve polymorphism using interfaces instead of abstract classes?

**Answer:**

Yes. We can achieve runtime polymorphism using interfaces. An interface
defines a contract that different classes can implement, and each class
can provide its own implementation.

``` csharp
public interface IPayment
{
    void Pay(decimal amount);
}

public class CashPayment : IPayment
{
    public void Pay(decimal amount)
    {
        Console.WriteLine("Cash payment");
    }
}

public class CreditCardPayment : IPayment
{
    public void Pay(decimal amount)
    {
        Console.WriteLine("Credit card payment");
    }
}
```

``` csharp
IPayment payment = new CreditCardPayment();
payment.Pay(500);

payment = new CashPayment();
payment.Pay(500);
```

The implementation that runs depends on the actual object at runtime.

## 4. What is the difference between an Abstract Class and an Interface?

**Answer:**

An abstract class is useful when related classes share common behavior
or state. An interface defines a contract or capability that different
classes can implement.

  -----------------------------------------------------------------------
  Feature                 Abstract class          Interface
  ----------------------- ----------------------- -----------------------
  Implemented methods     Yes                     Yes, through default
                                                  interface
                                                  implementations in
                                                  modern C#

  Abstract members        Yes                     Yes; interface members
                                                  are generally
                                                  implicitly abstract
                                                  unless implemented

  Instance fields         Yes                     No

  Constructors            Yes                     No instance
                                                  constructors

  Instance state          Yes                     No instance fields to
                                                  hold state

  Multiple inheritance    A class can inherit     A class can implement
                          only one class          multiple interfaces
  -----------------------------------------------------------------------

**When should we use each?**

-   Use an abstract class when related classes need shared
    implementation or instance state.
-   Use an interface when different classes need to follow the same
    contract or expose the same capability.

## 5. What happens if we remove the `override` keyword from an abstract method implementation?

**Answer:**

The code will not compile because a non-abstract derived class must
implement all inherited abstract members.

The `override` keyword tells the compiler that we are providing an
implementation for an inherited abstract or virtual method.

``` csharp
public abstract class PaymentMethod
{
    public abstract void Pay(decimal amount);
}

public class CashPayment : PaymentMethod
{
    public override void Pay(decimal amount)
    {
        Console.WriteLine("Cash payment");
    }
}
```

## 6. What is the difference between `virtual`, `abstract`, and `override`?

**Answer:**

-   `virtual`: Provides a default implementation that derived classes
    may override.
-   `abstract`: Declares a member without an implementation; a concrete
    derived class must implement it.
-   `override`: Provides a new implementation of an inherited virtual or
    abstract member.

``` csharp
public class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("Animal sound");
    }
}

public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}
```

## 7. What are the four pillars of OOP?

**Answer:**

1.  **Encapsulation:** Hiding internal data and controlling access to
    it.
2.  **Inheritance:** Reusing and extending functionality from another
    class.
3.  **Polymorphism:** Allowing the same base type or interface to
    represent different implementations.
4.  **Abstraction:** Exposing essential functionality while hiding
    implementation details.

## 8. What is Runtime Polymorphism?

**Answer:**

Runtime polymorphism allows a base-class or interface reference to refer
to different concrete implementations. For virtual and abstract members,
the implementation that executes is selected at runtime based on the
actual object type.

## 9. What is Compile-Time Polymorphism?

**Answer:**

Compile-time polymorphism means the compiler determines which overloaded
method to call based on the arguments and available method signatures.
Method overloading is a common example.

## 10. What is the difference between an object and a class?

**Answer:**

A class is a blueprint that defines data and behavior. An object is an
instance of a class.

``` csharp
public class Car
{
    public string Model { get; set; }
}

Car car = new Car();
car.Model = "Sedan";
```

Here, `Car` is the class and `car` is a variable holding a reference to
the created object.

------------------------------------------------------------------------

## Additional OOP Questions to Prepare

  -----------------------------------------------------------------------
  \#                                  Interview question
  ----------------------------------- -----------------------------------
  11                                  What is the difference between
                                      `const` and `readonly`?

  12                                  What is the difference between
                                      `ref` and `out`?

  13                                  What is the difference between an
                                      abstract class and a sealed class?

  14                                  What is composition
                                      vs. inheritance?

  15                                  What is the difference between `==`
                                      and `.Equals()`?

  16                                  What are boxing and unboxing?

  17                                  What is the difference between a
                                      value type and a reference type?

  18                                  What is Dependency Injection?

  19                                  What is the difference between
                                      association, aggregation, and
                                      composition?

  20                                  What is the difference between
                                      `new` and `override` when dealing
                                      with inherited members?
  -----------------------------------------------------------------------

## Study Advice

-   Understand the idea before memorizing the answer.
-   Practice explaining each answer aloud in your own words.
-   Write and run the code examples.
-   Be ready to explain when you would use each concept in a real
    ASP.NET Core project.
-   Try answering the additional questions before checking documentation
    or examples.

**Next topic:** SOLID Principles, starting with the Single
Responsibility Principle (SRP), using practical C# and ASP.NET Core
examples.

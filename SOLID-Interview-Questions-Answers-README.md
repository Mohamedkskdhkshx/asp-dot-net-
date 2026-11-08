# SOLID Principles --- Interview Questions and Answers

A practical interview revision guide for C# and ASP.NET Core developers.

## 1. What are the SOLID principles?

**Answer:**

SOLID is a set of five object-oriented design principles that help
developers build software that is easier to maintain, test, extend, and
understand.

-   **S --- Single Responsibility Principle (SRP):** A class should have
    one responsibility and one reason to change.
-   **O --- Open/Closed Principle (OCP):** Software entities should be
    open for extension but closed for modification.
-   **L --- Liskov Substitution Principle (LSP):** Objects of a derived
    class should be usable wherever objects of the base class are
    expected without breaking expected behavior.
-   **I --- Interface Segregation Principle (ISP):** A class should not
    be forced to implement methods it does not need.
-   **D --- Dependency Inversion Principle (DIP):** High-level and
    low-level modules should depend on abstractions, not directly on
    each other.

------------------------------------------------------------------------

## 2. Single Responsibility Principle (SRP)

### Q1. What is the Single Responsibility Principle?

**Answer:**

SRP states that a class should have only one responsibility and one
reason to change. It helps make code easier to maintain, test, and
modify.

### Q2. Does SRP mean that a class should have only one method?

**Answer:**

No. A class can have multiple methods, as long as those methods are
related to the same responsibility. SRP is about responsibility, not the
number of methods.

### Q3. A `ProductService` creates, updates, and deletes products, sends emails, and generates PDF reports. Does it follow SRP?

**Answer:**

No. It has multiple responsibilities: product management, email
notifications, and PDF generation. We can separate these
responsibilities into classes such as `ProductService`, `EmailService`,
and `PdfService`.

------------------------------------------------------------------------

## 3. Open/Closed Principle (OCP)

### Q4. What is the Open/Closed Principle?

**Answer:**

OCP states that software entities should be open for extension but
closed for modification. We should be able to add new functionality
without repeatedly changing existing, tested code.

### Q5. A payment service uses `if/else` statements for Cash and Credit Card payments. Every time we add a payment method, we modify the service. Which principle is violated, and how can we improve the design?

**Answer:**

This can violate OCP. We can create an abstraction such as
`IPaymentMethod` and implement it in separate classes such as
`CashPayment`, `CreditCardPayment`, and `PayPalPayment`. The payment
processor can depend on the interface instead of checking every payment
type.

### Q6. Does OCP mean that we must never modify existing code?

**Answer:**

No. It means we should design suitable extension points so that adding
new behavior does not require unnecessary changes to existing code. Bug
fixes and changing requirements may still require modifications.

------------------------------------------------------------------------

## 4. Liskov Substitution Principle (LSP)

### Q7. What is the Liskov Substitution Principle?

**Answer:**

LSP states that objects of a derived class should be usable wherever
objects of the base class are expected without breaking the program's
expected behavior.

### Q8. A `Bird` base class has a `Fly()` method. `Eagle` can fly, but `Penguin` overrides `Fly()` and throws `NotSupportedException`. Which principle is violated, and how can we fix it?

**Answer:**

This violates LSP because a `Penguin` cannot fulfill the expected
behavior of a bird type that promises flying. We can move flying
behavior into a separate interface such as `IFlyingBird`, implemented
only by birds that can fly.

### Q9. Is inheritance alone enough to satisfy LSP?

**Answer:**

No. A derived class must preserve the expected behavior and contract of
its base class. Simply inheriting from a base class does not guarantee
that the design follows LSP.

------------------------------------------------------------------------

## 5. Interface Segregation Principle (ISP)

### Q10. What is the Interface Segregation Principle?

**Answer:**

ISP states that a class should not be forced to implement methods it
does not need. We should prefer small, focused interfaces over large
interfaces.

### Q11. An `IWorker` interface contains `Work()`, `Eat()`, and `Sleep()`. A `Robot` only needs `Work()`. Which principle is violated, and how can we fix it?

**Answer:**

This violates ISP. We can split `IWorker` into smaller interfaces such
as `IWorkable`, `IEatable`, and `ISleepable`. A robot can implement only
`IWorkable`, while a human can implement all three.

### Q12. Why are small interfaces useful?

**Answer:**

They reduce unnecessary dependencies, make implementations simpler, and
allow classes to implement only the capabilities they actually support.

------------------------------------------------------------------------

## 6. Dependency Inversion Principle (DIP)

### Q13. What is the Dependency Inversion Principle?

**Answer:**

DIP states that high-level modules should not depend directly on
low-level modules. Both should depend on abstractions, such as
interfaces or abstract classes.

### Q14. An `OrderService` directly creates an `EmailService` using `new EmailService()`. Which principle may be violated, and how can we improve the design?

**Answer:**

This violates DIP because the high-level `OrderService` depends directly
on the concrete low-level `EmailService`. We can introduce an
abstraction such as `INotificationService`, make `EmailService`
implement it, and inject the abstraction into `OrderService`.

### Q15. What is the difference between Dependency Inversion and Dependency Injection?

**Answer:**

-   **Dependency Inversion (DIP)** is a design principle that encourages
    depending on abstractions.
-   **Dependency Injection (DI)** is a technique for providing an
    object's dependencies from outside the object.
-   DI is commonly used to help implement DIP, but they are not the same
    thing.

### Q16. How do you register an implementation in ASP.NET Core Dependency Injection?

**Answer:**

For example, in `Program.cs`:

``` csharp
builder.Services.AddScoped<INotificationService, EmailService>();
```

ASP.NET Core's DI container can then provide an `EmailService` wherever
an `INotificationService` is required.

------------------------------------------------------------------------

## 7. General SOLID Interview Questions

### Q17. Why are SOLID principles important?

**Answer:**

They help reduce tight coupling, improve cohesion, make code easier to
test and maintain, and make it easier to add new features safely.

### Q18. Are SOLID principles strict rules that must always be applied?

**Answer:**

No. They are design guidelines. We should apply them when they solve a
real design problem and avoid unnecessary abstraction or
overengineering.

### Q19. Can one design violate more than one SOLID principle?

**Answer:**

Yes. For example, a large service class that handles many unrelated
tasks and directly creates concrete dependencies may violate both SRP
and DIP.

### Q20. How are SOLID principles related to ASP.NET Core?

**Answer:**

In ASP.NET Core, we can apply SOLID by keeping controllers focused,
separating business logic into services, using interfaces for
abstractions, injecting dependencies through the built-in DI container,
and designing small, focused components.

------------------------------------------------------------------------

## Quick Revision Table

  Principle   Remember
  ----------- -------------------------------------------------------
  SRP         One responsibility; one reason to change.
  OCP         Extend behavior without unnecessary modification.
  LSP         Derived types must preserve expected behavior.
  ISP         Do not force classes to implement unused methods.
  DIP         Depend on abstractions, not concrete implementations.

## Practice Challenge

Design an E-commerce Order System using SOLID principles.

Consider these responsibilities:

-   Create and manage orders.
-   Calculate order totals and discounts.
-   Process payments.
-   Send email or SMS notifications.
-   Generate invoices.

**Your tasks:**

1.  Identify the responsibilities that should be separated.
2.  Decide which interfaces are needed.
3.  Explain how the services should depend on abstractions.
4.  Describe how you would add a new payment method without changing the
    existing order-processing logic.
5.  Explain how your design follows each SOLID principle.

Try solving the challenge before looking for a complete implementation.
In an interview, explain the design decisions and the reasons behind
them, not just the definitions.

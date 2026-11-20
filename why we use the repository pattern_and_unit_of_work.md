# Architectural Guide: Repository & Unit of Work Patterns

## 1. The Starting Point: The 2-Layer Architecture

In a standard API application, we typically begin with two layers:

1. **The Top Layer (Controller):** Responsible for handling requests and containing both the business logic and the database queries.
2. **The Second Layer (Model Layer):** Contains the data models and the `DbContext`.

**Data Flow:**

```text
Controller ──► DbContext (Model Layer)
```

### The Problem

We run into issues with the top layer because it suffers from **repeated code**—the exact same database queries get duplicated across multiple controllers.

---

## 2. The Enhancement: Introducing the Repository Pattern (3-Layer Architecture)

To solve code duplication, we separate the query logic into a dedicated repository layer. Each entity gets its own repository containing the specific queries it needs.

After this enhancement, our architecture expands to **three layers**:

1. **Top Layer (Controller):** Contains the application logic only.
2. **Second Layer (Repositories):** Contains the queries and calls the model layer.
3. **Third Layer (Model Layer):** Contains the models and `DbContext`.

**Updated Data Flow:**

```text
Controller ──► Repository ──► DbContext (Model Layer)
```

---

## 3. Implementing Dependency Injection

To use the repository inside the controller without manually instantiating it (`new`), we use Dependency Injection via constructor injection:

```csharp
private readonly StudentRepository _studentRepository;

public StudentsController(StudentRepository studentRepository)
{
    _studentRepository = studentRepository;
}
```

### The Next Challenge

While constructor injection removes the anti-pattern of creating objects inside objects, we now face a new limitation: **tight coupling to a concrete implementation**.

If our application grows to include multiple data access layers, or if we want our controller to not be tied to just one concrete repository, depending directly on a concrete class creates rigidity.

---

## 4. Dependency Inversion & Abstractions

The Dependency Inversion Principle dictates that high-level modules should depend on **abstractions**, not concrete implementations.

To achieve this:

1. **Create an Interface:** Define an interface for the student repository (e.g., `IStudentRepository`) that the concrete repository must implement.
2. **Update the Controller:** Inject the **interface** into the controller instead of the concrete repository class:

```csharp
private readonly IStudentRepository _studentRepository;

public StudentsController(IStudentRepository studentRepository)
{
    _studentRepository = studentRepository;
}
```

3. **Configure in `Program.cs`:** Register the mapping in the DI container. This tells the application which concrete repository to use at runtime (provided the repository implements the interface):

```csharp
// Example registration in Program.cs
builder.Services.AddScoped<IStudentRepository, StudentRepository>();
```

---

## 5. The Multi-Repository Problem (DbContext Isolation)

As the application grows, a single controller often needs to work with multiple repositories at the same time (e.g., `StudentRepository` and `CourseRepository`).

### The Problem

If each repository creates its own separate instance of `DbContext` and calls `SaveChanges()` independently:

1. **Isolated Instances:** The object in the first repository cannot see the changes tracked by the object in the second repository.
2. **Separate Saves:** Saving changes independently breaks transactional safety—if one repository succeeds and the other fails, your database ends up in an inconsistent state.

---

## 6. The Solution: The Unit of Work Pattern

To ensure all repositories share the exact same `DbContext` instance and commit their changes together, we introduce the **Unit of Work Pattern**.

The Unit of Work coordinates multiple repositories under a single `DbContext` instance and exposes one unified method (such as `CompleteAsync()`) to save all changes at once.

**Final Data Flow:**

```text
Controller ──► Unit of Work ──► Repositories (Shared DbContext) ──► Database
```

### Implementation Overview

1. **Unit of Work Interface & Class:** Holds references to all repositories and passes the **same `DbContext` instance** into each of them:

```csharp
public interface IUnitOfWork : IDisposable
{
    IStudentRepository Students { get; }
    ICourseRepository Courses { get; }
    Task<int> CompleteAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;

    public UnitOfWork(AppDbContext context)
    {
        _context = context;
        Students = new StudentRepository(_context);
        Courses = new CourseRepository(_context);
    }

    public IStudentRepository Students { get; private set; }
    public ICourseRepository Courses { get; private set; }

    public async Task<int> CompleteAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public void Dispose()
    {
        _context.Dispose();
    }
}
```

2. **Usage in the Controller:** The controller injects `IUnitOfWork` instead of individual repositories, allowing multiple repositories to collaborate on the same context before saving once:

```csharp
private readonly IUnitOfWork _unitOfWork;

public EnrollmentsController(IUnitOfWork unitOfWork)
{
    _unitOfWork = unitOfWork;
}

public async Task<IActionResult> EnrollStudent(int studentId, int courseId)
{
    var student = await _unitOfWork.Students.GetByIdAsync(studentId);
    var course = await _unitOfWork.Courses.GetByIdAsync(courseId);

    // Both repositories operate on the exact same DbContext instance

    // Commit all changes together in a single transaction
    await _unitOfWork.CompleteAsync();

    return Ok();
}
```

3. **Register in `Program.cs`:**

```csharp
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

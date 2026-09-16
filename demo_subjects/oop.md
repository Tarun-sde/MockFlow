### Q1. What are the four fundamental pillars of Object-Oriented Programming?

**A.** 
The four foundational pillars of Object-Oriented Programming (OOP) are:

1. **Encapsulation**: Bundling data (attributes) and the methods that operate on that data into a single unit (class), while restricting direct external access to internal state using access modifiers (`private`, `protected`). Prevents unwanted side-effects and guarantees invariants.
2. **Abstraction**: Hiding internal implementation complexity and exposing only relevant, high-level interfaces to consumers. For example, a `DatabaseConnection` interface exposes `.connect()` and `.query()` while concealing low-level socket protocol handshakes.
3. **Inheritance**: The mechanism by which a derived class inherits attributes and behaviors from a base class, promoting code reuse and establishing an "is-a" relationship hierarchy.
4. **Polymorphism**: The ability of different classes to respond to the same interface or method call in their own specialized way. Divided into Compile-Time (static) polymorphism (method overloading) and Runtime (dynamic) polymorphism (method overriding via virtual method tables).

### Q2. Explain each of the SOLID principles with a concise real-world example.

**A.** 
SOLID represents five design principles for maintainable, decoupled object-oriented systems:

1. **Single Responsibility Principle (SRP)**: A class should have one, and only one, reason to change. Example: A `User` entity should not handle both profile data and sending password reset emails; extract email operations into an `EmailService`.
2. **Open/Closed Principle (OCP)**: Software entities should be open for extension, but closed for modification. Example: Adding a new payment method (e.g., Apple Pay) should be done by implementing a `PaymentGateway` interface rather than editing an existing `switch` statement in `PaymentProcessor`.
3. **Liskov Substitution Principle (LSP)**: Subtypes must be substitutable for their base types without altering system correctness. Example: If `Penguin` inherits from `Bird`, but calling `fly()` throws an exception, LSP is violated.
4. **Interface Segregation Principle (ISP)**: Clients should not be forced to depend upon interfaces they do not use. Example: Prefer granular interfaces (`Printer`, `Scanner`, `Fax`) over a single bloated `AllInOneMachine` interface.
5. **Dependency Inversion Principle (DIP)**: High-level modules should not depend on low-level modules; both should depend on abstractions. Example: An `OrderService` should depend on an `INotificationService` interface rather than a concrete `TwilioSmsClient` class.

### Q3. Why is Composition favored over Inheritance ("Favor composition over inheritance")?

**A.** 
Inheritance introduces tight coupling and fragile base class problems:

1. **White-Box Reuse vs Black-Box Reuse**: Inheritance exposes internal implementation details of parent classes to subclasses (white-box reuse). Changes in a parent class frequently break assumptions or invariants in derived classes unexpectedly.
2. **Compile-Time Static Binding**: Inheritance relationships are fixed at compile time and cannot be altered during program execution. Composition allows dynamic runtime swapping of behaviors (e.g., swapping a `ShippingStrategy` instance at runtime).
3. **Class Explosion**: Modeling combinatorial variations with inheritance (e.g., `FastCar`, `FlyingCar`, `FastFlyingCar`) leads to deeply nested, unmaintainable hierarchies. Composition ("has-a" relationship) allows combining small, focused collaborating components flexibly.

### Q4. What is the difference between Method Overloading and Method Overriding?

**A.** 
- **Method Overloading (Compile-Time / Static Polymorphism)**:
  Occurs within the **same class** when multiple methods share the identical name, but have different parameter signatures (different number of parameters, different parameter types, or different order of types). The compiler determines which method to invoke at compile time based on argument types. Return type alone cannot distinguish overloaded methods.
- **Method Overriding (Runtime / Dynamic Polymorphism)**:
  Occurs across an **inheritance relationship** when a derived subclass provides its own specific implementation of a method already defined in its superclass. The method signature (name, parameter list, and compatible return type) must match exactly. Method resolution occurs dynamically at runtime using the instance's actual runtime type (via a Virtual Method Table / vtable).

### Q5. What is the difference between an Abstract Class and an Interface?

**A.** 
- **Abstract Class**:
  Represents a partial blueprint ("is-a" identity). Cannot be instantiated directly. Can define concrete state (instance variables with state), constructors, concrete methods with full implementations, and abstract method signatures that subclasses must implement. Most languages (Java, C#, Python) restrict classes to single inheritance of abstract classes.
- **Interface**:
  Represents a contract or capability specification ("can-do" capability). In classical OOP, it defines only method signatures without state or implementation details (though modern Java supports default/static methods). A class can implement multiple interfaces, allowing flexible polymorphic behavior across unrelated inheritance trees.

### Q6. What is the Diamond Problem in multiple inheritance and how do languages resolve it?

**A.** 
The Diamond Problem arises in multiple inheritance when a class `D` inherits from two classes `B` and `C`, both of which inherit from a common base class `A`. If `B` and `C` override a method from `A`, an ambiguity arises: which implementation does `D` inherit when that method is invoked on an instance of `D`?

**Language Resolutions**:
1. **Disallowing Multiple Class Inheritance**: Languages like Java, C#, and Go forbid multiple class inheritance entirely, allowing multiple inheritance only of interfaces.
2. **Method Resolution Order (MRO / C3 Linearization)**: Python resolves the ambiguity by creating a deterministic, linear precedence list of ancestor classes, traversed left-to-right depth-first while preserving monotonicity.
3. **Virtual Inheritance**: C++ allows virtual base classes (`class B : virtual public A`), ensuring only a single shared instance of the base subobject `A` exists within `D`, requiring explicit qualification if overrides conflict.

### Q7. What is Dependency Inversion and how does Dependency Injection implement it?

**A.** 
- **Dependency Inversion Principle (DIP)**: A high-level design guideline stating that high-level business logic modules should depend on abstract contracts (interfaces) rather than direct, concrete low-level implementations.
- **Dependency Injection (DI)**: A concrete creational design pattern used to achieve Dependency Inversion. Instead of a class instantiating its own dependencies internally via `new ConcreteService()`, the dependencies are "injected" from the outside by a caller or an IoC (Inversion of Control) container.

**Injection Styles**:
1. **Constructor Injection**: Dependencies are passed as parameters into the class constructor (preferred for mandatory dependencies, ensuring instances are initialized in a valid state).
2. **Setter Injection**: Dependencies are assigned via property setter methods (useful for optional dependencies).
3. **Method Injection**: Dependencies are provided as parameters to the specific method call requiring them.

### Q8. What is the difference between Value Objects and Entities in Domain-Driven Design?

**A.** 
- **Entities**:
  Objects defined by their **identity**, which remains constant throughout their lifecycle regardless of state changes. Two entities with identical attributes are considered distinct if their IDs differ. Example: A `User` entity retains its identity across username, address, or email modifications.
- **Value Objects**:
  Objects defined exclusively by their **attributes** with no conceptual identity. They are immutable. Two value objects with identical properties are considered completely equal and interchangeable. Example: `Money(amount: 50, currency: "USD")` or `Address(street, city, zip)`. Modifying a value object means creating a new instance rather than mutating the existing one in place.

### Q9. Explain the Factory Method pattern vs the Abstract Factory pattern.

**A.** 
- **Factory Method**:
  A creational pattern that uses inheritance. A creator base class defines an abstract factory method (`createProduct()`), delegating the decision of which specific concrete class to instantiate to derived subclasses. Focuses on producing a single product.
- **Abstract Factory**:
  A creational pattern that uses composition. An interface declares a family of related or dependent product creation methods without specifying their concrete classes (e.g., `createButton()`, `createScrollbar()`, `createCheckbox()`). Concrete factory implementations (such as `MacUIFactory`, `WindowsUIFactory`) instantiate families of matching components that work harmoniously together.

### Q10. What is the Liskov Substitution Principle and how is it violated by a Square inheriting from Rectangle?

**A.** 
The Liskov Substitution Principle (LSP) states that objects of a superclass should be replaceable with objects of its subclasses without altering any of the desirable properties of the program (correctness, task performed, etc.).

**The Square / Rectangle Violation**:
In mathematics, a square is a rectangle with equal sides. In OOP, if `Square` inherits from `Rectangle`:
- `Rectangle` provides independent setters: `setWidth(w)` and `setHeight(h)`.
- To maintain square invariants, `Square` overrides both setters so modifying width also changes height, and vice-versa.
- A consumer function expecting a `Rectangle`:
  ```python
  def resize(rect: Rectangle):
      rect.setWidth(5)
      rect.setHeight(10)
      assert rect.getArea() == 50
  ```
Passing a `Square` causes `rect.getArea()` to return 100 instead of 50, breaking the caller's reasonable invariant expectations and violating LSP. A Square should not inherit from Rectangle; they should both implement a common `Shape` interface.

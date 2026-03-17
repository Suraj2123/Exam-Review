# Software Engineering — Comprehensive Exam Review

Based on Lectures 03, 05, 07, 08, 09, 10, 11 and the exam topic list.

---

# 1. SOLID PRINCIPLES (Lecture 03)

SOLID is a set of 5 design principles for writing maintainable, extensible object-oriented code. Each principle addresses a specific type of design problem.

---

## S — Single Responsibility Principle (SRP)

**Definition:** A class should have only ONE reason to change. Each class should do ONE thing.

**Why it matters:** If a class handles multiple responsibilities, changing one responsibility risks breaking the other. The class becomes fragile and hard to test.

**Violation example:**
```cpp
class UserManager {
    void saveToDatabase(User u) { /* DB logic */ }
    void sendEmail(User u) { /* email logic */ }
    void generateReport(User u) { /* PDF logic */ }
};
```
This class has THREE reasons to change: database schema changes, email format changes, report format changes.

**Solution:**
```cpp
class UserRepository { void save(User u); };
class EmailService { void sendEmail(User u); };
class ReportGenerator { void generate(User u); };
```
Each class has one job. Changing email logic can't break database logic.

**How to spot SRP violations:** If you describe a class and use the word "and" — it probably violates SRP.

---

## O — Open/Closed Principle (OCP)

**Definition:** Software entities should be **open for extension** but **closed for modification**. You should be able to add new behavior WITHOUT editing existing, working code.

**Why it matters:** Every time you modify existing code, you risk introducing bugs in something that already works.

**Violation example:**
```cpp
double calculateArea(Shape s) {
    if (s.type == "circle")
        return 3.14 * s.radius * s.radius;
    else if (s.type == "rectangle")
        return s.width * s.height;
    // Adding triangle? Must MODIFY this function
}
```
Every new shape requires editing this function.

**Solution — use polymorphism:**
```cpp
class Shape {
public:
    virtual double area() = 0;
};

class Circle : public Shape {
    double area() override { return 3.14 * radius * radius; }
};

class Triangle : public Shape {
    double area() override { return 0.5 * base * height; }
};
```
Adding a new shape = adding a new class. Existing code is never touched.

---

## L — Liskov Substitution Principle (LSP)

**Definition:** If class B is a subclass of class A, you should be able to replace A with B anywhere in the code without breaking anything.

**Why it matters:** Inheritance should represent a true "is-a" relationship. If a subclass changes the expected behavior, code that depends on the parent type will break.

**Classic violation — Square/Rectangle problem:**
```cpp
class Rectangle {
    virtual void setWidth(int w) { width = w; }
    virtual void setHeight(int h) { height = h; }
};

class Square : public Rectangle {
    void setWidth(int w) override { width = w; height = w; }
    void setHeight(int h) override { width = h; height = h; }
};
```
A Square overrides setWidth to also change height. Code that does `rect->setWidth(5); rect->setHeight(10);` expects width=5 and height=10, but with a Square it gets width=10 and height=10. **Behavior is broken** — LSP violated.

**How to spot LSP violations:** If a subclass:
- Throws exceptions the parent doesn't
- Ignores or overrides parent behavior in unexpected ways
- Changes the meaning of inherited methods

...then LSP is violated.

---

## I — Interface Segregation Principle (ISP)

**Definition:** No client should be forced to depend on methods it does not use. Prefer many small, focused interfaces over one large "fat" interface.

**Why it matters:** If a class implements an interface with 10 methods but only needs 3, it's forced to provide dummy implementations for the other 7. This creates confusion and fragile code.

**Violation example:**
```cpp
class Animal {
public:
    virtual void fly() = 0;
    virtual void swim() = 0;
    virtual void run() = 0;
};

class Dog : public Animal {
    void fly() override { /* Dogs can't fly! Forced to implement anyway */ }
    void swim() override { /* OK */ }
    void run() override { /* OK */ }
};
```

**Solution — split into focused interfaces:**
```cpp
class Flyable { virtual void fly() = 0; };
class Swimmable { virtual void swim() = 0; };
class Runnable { virtual void run() = 0; };

class Dog : public Swimmable, public Runnable { /* Only what it needs */ };
class Eagle : public Flyable, public Runnable { /* Only what it needs */ };
```

---

## D — Dependency Inversion Principle (DIP)

**Definition:** High-level modules should not depend on low-level modules. Both should depend on **abstractions** (interfaces). Abstractions should not depend on details — details should depend on abstractions.

**Why it matters:** If your business logic directly depends on a specific database or API, changing that database means rewriting your business logic.

**Violation example:**
```cpp
class OrderService {
    MySQLDatabase db;  // Directly depends on MySQL
public:
    void saveOrder(Order o) { db.insert(o); }
};
```
Switching to PostgreSQL means rewriting OrderService.

**Solution:**
```cpp
class Database {
public:
    virtual void insert(Order o) = 0;
};

class MySQLDatabase : public Database { /* MySQL implementation */ };
class PostgresDatabase : public Database { /* Postgres implementation */ };

class OrderService {
    Database* db;  // Depends on abstraction, not MySQL specifically
public:
    OrderService(Database* database) : db(database) {}
    void saveOrder(Order o) { db->insert(o); }
};
```
Now you can swap databases without touching OrderService.

---

## SOLID — Quick Reference

| Principle | One-liner | Violation smell |
|-----------|-----------|-----------------|
| **SRP** | One class = one reason to change | Class does too many things |
| **OCP** | Extend behavior without modifying existing code | Adding features = editing if/else chains |
| **LSP** | Subclass can replace parent without surprises | Subclass breaks parent's expected behavior |
| **ISP** | Small focused interfaces, not fat ones | Classes forced to implement methods they don't need |
| **DIP** | Depend on abstractions, not concrete classes | Business logic directly uses specific DB/API |

---
---

# 2. OBJECT-ORIENTED CONCEPTS & INTERFACES (Lecture 05)

---

## Encapsulation

**Definition:** Bundling data with the methods that operate on it, AND restricting direct access to some components.

In C++ this is done through access modifiers:
- **private** — not accessible outside the class, not even by subclasses
- **protected** — accessible by subclasses but not outside
- **public** — accessible by everyone

**Terminology:**
- **Member** (member variable / internal member variable) = a piece of data associated with an object
- **Method** (member function) = a function associated with the object

---

## Abstraction

**Definition:** Hiding unnecessary internal complexity from the user, allowing them to work with simpler interfaces.

Accomplished through **public methods**:
- **Accessors (getters)** — controlled read access to encapsulated data
- **Mutators (setters)** — controlled write access to encapsulated data
- Other public methods let users perform complex operations without understanding internals

---

## Interface

**Definition:** "A description of the actions that an object can do." All publicly accessible methods of an object = that object's interface.

Abstraction + Encapsulation = Interface
- Abstraction defines WHAT users can do
- Encapsulation prevents them from doing anything else
- Together they create and enforce the interface

Interfaces are NOT specific to OOP — they exist across many programming paradigms.

---

## Inheritance

**Definition:** A mechanism where one class acquires the properties and methods of another class.

**Terminology:**
- **Superclass** (parent class / base class) = the class being inherited from
- **Subclass** (child class / derived class) = the class that inherits

```cpp
class A { };
class B : public A { };    // B is subclass of A
class C : public B { };    // C is subclass of both A and B
                            // B is superclass of C, subclass of A
```

**Key C++ behavior:** A subclass can be referenced using a superclass pointer. This enables polymorphism — you can treat different subclasses as "the same type."

```cpp
class Animal {
public:
    virtual void sound() = 0;  // Pure virtual = abstract method
};

class Cat : public Animal {
    void sound() override { cout << "meow"; }
};

class Dog : public Animal {
    void sound() override { cout << "woof"; }
};

vector<Animal*> animals;
animals.push_back(new Cat());
animals.push_back(new Dog());

for (auto animal : animals) {
    animal->sound();  // Calls the correct version for each type
}
```

**Pure virtual function** (`= 0`): makes the class abstract — it CANNOT be instantiated directly. Subclasses MUST implement it.

---

## Why Interfaces + Inheritance Matter

**Extensibility:** Adding a new subclass (e.g., `Turtle`) works immediately with existing code that uses the `Animal*` pointer — no modifications needed.

**Maintainability:** Changing a subclass's implementation doesn't affect code that uses the interface.

This directly connects to **OCP** (open for extension, closed for modification) and **DIP** (depend on abstractions).

---

## Interfaces Across Languages

| Feature | C++ | Python | Go |
|---------|-----|--------|-----|
| Interface enforcement | Pure virtual functions (`= 0`) | Duck typing (no enforcement) | Explicit interface types |
| Inheritance | `class Dog : public Animal` | `class Dog(Animal)` | No classical inheritance |
| Polymorphism | Through base class pointers | Automatic (duck typing) | Anything matching interface = that type |

**Python duck typing:** If an object has the right methods, Python will call them regardless of whether the class inherits from the right parent. "If it walks like a duck and quacks like a duck, it's a duck."

---
---

# 3. EMERGENT DESIGN (Lecture 07)

Source: Clean Code Chapter 12

---

## Definition

Emergent design means the design **evolves** as the code and understanding evolve, rather than over-designing everything upfront. The design "emerges" from following simple rules consistently.

---

## The 4 Rules of Simple Design (in priority order)

### Rule 1: Run All the Tests
A design is wrong if it can't be verified. Tests prove the system works. Without tests, you can't refactor safely, and the design will rot.

### Rule 2: No Duplication
Duplication = extra work, extra risk, extra complexity. If two pieces of code do similar things, merge them.

**Example — before:**
```cpp
void scaleToOneDimension(float desiredDimension, float imageDimension) {
    // ... scaling logic for one dimension
    image = newImage;
}

void rotate(int degrees) {
    // ... same scaling logic duplicated here
    image = newImage;
}
```

**After — extract shared logic:**
```cpp
void replaceImage(RenderedOp newImage) {
    image = newImage;
}
```

### Template Method Pattern
When algorithms are similar but differ in specific steps:
1. Break the algorithm into steps
2. Turn steps into methods
3. Put them in an abstract class with a "template method"
4. Have specific algorithms be subclasses that override only the differing steps

### Rule 3: Expressive
Code should clearly communicate its intent. Good names, small functions, standard patterns — make it easy for the NEXT person to understand.

### Rule 4: Minimal Classes and Methods
Don't create classes and methods unnecessarily. Keep the design as small as possible while satisfying Rules 1–3.

---

## Advantages of Emergent Design
- Adapts to changing requirements
- Simpler initial design reduces wasted work
- Continuous refactoring keeps quality high

## Disadvantages
- No plan for changes → design can deteriorate
- No plan for growth → may limit future development
- Ignores crucial dependencies → wrong implementations
- Lack of coordination → development deadlocks

**Mitigation:** Clear and SOLID interface creation from the very beginning.

---
---

# 4. STRONG CODING (Lecture 08)

Source: Code Complete Chapters 10–11, Clean Code Chapter 2

---

## Starting Strong

- Turn off implicit declarations (enforced by g++)
- Declare all variables (enforced by g++)
- Use naming conventions
- Check variable names

---

## Variable Initialization

### When to Initialize
Initialize variables as close to their first use as possible.

### Preventing Unwanted Changes
- Use `const` when possible
- Reset counters and accumulators
- Initialize class member data in the constructor
- Use the destructor to free memory allocated in the constructor
- Keep track of variables that are reinitialized
- Don't ignore compiler warnings
- Always validate input before assigning to local variables

---

## Localizing References — Span and Live Time

### Span
The number of lines between consecutive references to a variable. Smaller span = better.

```
a = 0;        // a referenced
b = 0;        // b referenced
c = 0;        // c referenced
a = b + c;    // a referenced again — span of a = 2 (lines between)
```

**Average span:** compute span for each pair of consecutive references, then average them.

### Live Time
The total number of lines from the first reference to the last reference of a variable. Shorter live time = better.

### Why Localize References?
- Reduces the "window of vulnerability" where bugs can be introduced
- Makes code easier to read — you don't have to scroll far to understand a variable
- Makes refactoring safer — moving a block of code is easier when all variable references are close together

---

## Minimize Scope
- Declare variables in the smallest scope possible
- Don't declare variables at the top of a function if they're only used in one block
- Group related statements together

---

## Use Variables for One Purpose
Don't reuse a variable for a different meaning later in the code. Create a new variable instead.

---

## Naming Variables

### Intention-Revealing Names
Choose names that describe WHAT the variable is for:
```cpp
int elapsedTimeInDays;   // Good — clear purpose
int d;                    // Bad — what is d?
```

### Avoid Disinformation
Don't use names that could be confused with something else:
```
XYZControllerForEfficientHandlingOfStrings
XYZControllerForEfficientStorageOfStrings
// Too similar — easy to pick the wrong one
```

### C++ Naming Conventions
| What | Convention | Example |
|------|-----------|---------|
| Integer indexes | `i`, `j` | `for (int i = 0; ...)` |
| Pointers | prefix `p` | `pNode` |
| Constants, typedefs, macros | ALL_CAPS | `MAX_SIZE` |
| Class names / type names | MixedCase | `MyClassName` |
| Variables and functions | camelCase (lowercase first) | `variableName` |
| Underscore | NOT used as separator (except ALL_CAPS and certain prefixes) | |

### Differentiating Classes and Objects
Use clear naming to distinguish between a class (type) and its instances (objects).

---
---

# 5. DEFENSIVE PROGRAMMING (Lecture 09)

Source: Code Complete Chapter 8

---

## Definition

"Garbage in, nothing out." Defensive programming is about handling garbage input safely rather than blindly processing it.

---

## Basic Principles
1. Check values of all data from **external sources**
2. Check values of all **routine input parameters**
3. Decide **how to handle bad inputs** before they cause damage

---

## Assertions

Assertions are checks embedded in production code (different from unit tests) to catch things that should NEVER happen.

**Use assertions to:**
- Validate input data
- Verify a file is open/closed
- Check file privileges (read-only, write, etc.)
- Confirm a value hasn't been changed by a routine
- Verify a pointer is not null
- Check a container is empty or full

### C++ Assertions

**Static assert** (compile time):
```cpp
static_assert(sizeof(int) == 4, "int must be 4 bytes");
```
Checked at compile time. If the condition is false, compilation fails with the error message. Use for conditions that must be true on the target platform.

**Dynamic assert** (runtime):
```cpp
#include <cassert>
assert(ptr != nullptr);    // Program aborts if false
assert(index >= 0 && index < size);
```
Checked at runtime. If the condition is false, the program prints the failed assertion and aborts. Typically disabled in release builds.

### Guidelines for Assertions
- Use assertions for conditions that should NEVER occur (programmer errors, not user errors)
- Don't use assertions for expected error conditions — use error handling for those
- Assertions should not have side effects (the program should work the same with assertions disabled)

---

## Error-Handling Techniques

When an error occurs, you have several options (choose based on context):

| Technique | When to Use |
|-----------|-------------|
| Return a neutral value | System must keep running after error |
| Return next valid data | Can handle some data loss |
| Return same answer as last time | Real-time variance isn't important |
| Return closest legal value | Reading from calibrated instruments |
| Log a warning to a file | Error handling is done offline |
| Call an error-processing routine | Makes debugging easier |
| Display an error message | User-facing (sanitize the message!) |
| Shut down the system | Safety-critical systems |

---

## Barricading a System

When working with **dirty data** (e.g., data from the internet), separate the error-handling/validation logic from the core system. Create a "barricade" — a validation layer at the boundary:

```
External data → [BARRICADE: validate, sanitize] → Clean data → Core system
```

Inside the barricade: use error handling (expect bad data)
Inside the core: use assertions (data should already be clean)

---

## Offensive Programming

In development/debug mode, make errors as loud and visible as possible:
- Make asserts abort the program
- Fill allocated memory to detect allocation errors
- Design `default` cases in switch statements to fail hard
- Fill objects with junk data before deleting
- Have the system email you log messages

---

## How Much Defense in Production?

| Keep | Remove |
|------|--------|
| Code that checks for important errors | Code for trivial errors |
| Code that crashes gracefully | Code that causes hard crashes |
| Error logging for tech support | Overly detailed debug output |
| Friendly, clean error messages | Raw technical messages |

---
---

# 6. CODE TUNING TECHNIQUES (Lecture 10 Part 2)

---

## General Principles

### Stop Testing When You Know the Answer
Use short-circuit evaluation. Add `break` statements when you've found what you need.

```cpp
// Less efficient:
for (int i = 0; i < n; i++) {
    if (data[i] < 0) negativeFound = true;
    // Keeps looping even after finding negative!
}

// Better:
for (int i = 0; i < n; i++) {
    if (data[i] < 0) { negativeFound = true; break; }
}
```

### Order Tests by Frequency
Put the most common cases first in if/else chains and switch statements.

---

## Loop Optimizations

| Technique | What It Does | When to Use |
|-----------|-------------|-------------|
| **Loop fission** | Split one loop doing two things into two separate loops | When operations in the loop don't depend on each other and splitting improves cache performance |
| **Loop fusion** | Combine two loops with same bounds into one | Reduces loop overhead when iterating over same range |
| **Loop interchange** | Swap inner and outer loop order | Improves cache locality for multi-dimensional arrays (iterate row-major) |
| **Loop-invariant code motion** | Move computations that don't change per iteration OUTSIDE the loop | Any expression inside a loop that stays constant |
| **Loop splitting** | Handle the special first/last iteration separately outside the loop | When the first or last iteration has different logic |
| **Loop unrolling** | Do multiple iterations' work in one iteration | Reduces loop overhead for simple operations |
| **Loop unswitching** | Move an if-statement that doesn't change per iteration OUTSIDE the loop | Condition is loop-invariant |

### Loop Interchange Example (Cache Locality)
```cpp
// Bad — jumps across rows (cache misses):
for (j = 0; j <= 20; j++)
    for (i = 0; i <= 10; i++)
        a[i][j] = i + j;

// Good — sequential memory access (cache hits):
for (i = 0; i <= 10; i++)
    for (j = 0; j <= 20; j++)
        a[i][j] = i + j;
```

### Loop-Invariant Code Motion Example
```cpp
// Before — y+z and x*x computed EVERY iteration:
for (int i = 0; i < n; i++) {
    x = y + z;
    a[i] = 6 * i + x * x;
}

// After — computed ONCE:
x = y + z;
t1 = x * x;
for (int i = 0; i < n; i++) {
    a[i] = 6 * i + t1;
}
```

---

## Strength Reduction

Replace expensive operations with cheaper ones:

| Expensive | Cheaper Replacement |
|-----------|-------------------|
| Multiplication | Addition |
| Exponentiation | Multiplication |
| Trig functions | Trig identities |
| `long long` | `long` or `int` (if range allows) |
| `double` | `float` (if precision allows) |
| Multiply/divide by 2 | Bit shift (`<< 1`, `>> 1`) |

### Example
```cpp
// Before — multiplication in loop:
for (i = 0; i < n; i++) {
    sum += 2 * (i + 1);
}

// After — closed-form formula:
sum = 5 + (n * (n + 1));
```

---

## Initialize at Compile Time
If a value can be computed at compile time, do it there instead of at runtime.

---

## ALWAYS MEASURE

Never guess where the bottleneck is. Use profiling tools:
- **Valgrind** (memory + performance)
- **Intel VTune**
- **Perf** (Linux)
- **OProfile**

Optimize PROVEN hotspots, not guesses.

---
---

# 7. PERSONAL CHARACTER (Lecture 11)

---

## Core Message: Be Humble, Not Smart

The best programmer is not the most intelligent one, but the most humble one:
- **Discipline, curiosity, and experience** will transform you into a competent programmer
- **Creativity** is fueled by passion
- You can always improve, but you're not as bad as you think you are

---

## Imposter Syndrome

A common feeling among software developers where you feel like a "fake" despite evidence of your competence.

**How to fight it:**
- Recognize that everyone struggles — even senior developers
- Focus on growth, not comparison
- Share your knowledge — teaching reinforces learning
- Accept that not knowing everything is normal

---

## The Professional Ladder

Growth as a developer is a continuous journey. You progress through stages of competence, and each stage requires humility to recognize what you still don't know.

---

## Intellectual Honesty

- Admit when you don't know something
- Don't pretend code works when you haven't tested it
- Report bugs you find, even if they're in your own code
- Give credit to others

---

## What Doesn't Matter as Much as You Think

- Being the "smartest" in the room
- Writing clever one-liners
- Knowing every language/framework
- Years of experience alone (without growth)

---

## Crunch Culture

"Crunch" = extended periods of overwork (60–80+ hour weeks), common in game development.

**Who does it benefit?** Usually only management/publishers in the short term.

**Ethical implications:** Exploits workers, damages health, strains relationships.

**Professional implications:** Leads to burnout, high turnover, lower code quality (tired developers write buggy code).

---
---

# 8. UNIT TESTING & GOOGLE TEST

---

## What Is Unit Testing?

Testing individual units of code (functions, methods, classes) in isolation to verify they work correctly.

---

## Google Test (gtest) Basics

### Writing a Test
```cpp
#include <gtest/gtest.h>

TEST(TestSuiteName, TestName) {
    EXPECT_EQ(add(2, 3), 5);
}
```

### Common Assertions

| Assertion | What It Checks |
|-----------|---------------|
| `EXPECT_EQ(a, b)` | `a == b` (non-fatal — test continues) |
| `ASSERT_EQ(a, b)` | `a == b` (fatal — test stops on failure) |
| `EXPECT_NE(a, b)` | `a != b` |
| `EXPECT_TRUE(cond)` | condition is true |
| `EXPECT_FALSE(cond)` | condition is false |
| `EXPECT_LT(a, b)` | `a < b` |
| `EXPECT_GT(a, b)` | `a > b` |
| `EXPECT_LE(a, b)` | `a <= b` |
| `EXPECT_GE(a, b)` | `a >= b` |
| `EXPECT_THROW(stmt, ExcType)` | statement throws expected exception |
| `EXPECT_NO_THROW(stmt)` | statement doesn't throw |

### EXPECT vs ASSERT
- **EXPECT_**: non-fatal. Test continues even if it fails. Use by default — lets you see ALL failures in one run.
- **ASSERT_**: fatal. Test STOPS immediately on failure. Use when continuing after failure would be meaningless (e.g., a null pointer check before dereferencing).

### Writing Good Error Messages
```cpp
EXPECT_EQ(result, expected) << "Failed for input x=" << x;
```
Add `<<` after an assertion to provide context when it fails.

---

## Code Coverage

### Definition
Percentage of code that is executed when tests run.

### Types
- **Line coverage** — which lines were executed
- **Branch coverage** — which decision paths (true/false) were taken
- **Function coverage** — which functions were called

### When to Measure Code Coverage
- After writing tests, to find untested areas
- During code review, to ensure new code is tested
- As a CI metric (but NOT as the sole quality indicator)

### Important Warning
**High coverage ≠ good tests.** A test can execute every line without checking if the output is correct. Coverage measures execution breadth, NOT assertion quality. You need meaningful assertions on edge cases.

---
---

# 9. GITHUB ACTIONS & YAML FILES

---

## GitHub Actions

GitHub Actions is a CI/CD system built into GitHub. It runs automated workflows when events happen (push, pull request, etc.).

### Key Concepts
- **Workflow** — an automated process defined in a YAML file (`.github/workflows/`)
- **Event/Trigger** — what starts the workflow (`push`, `pull_request`, `schedule`)
- **Job** — a set of steps that run on the same runner
- **Step** — a single task in a job (run a command, use an action)
- **Runner** — the machine that executes the job (`ubuntu-latest`, `macos-latest`)

### Reading a YAML File

```yaml
name: CI Pipeline            # Workflow name

on:                           # Triggers
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:                         # Jobs to run
  build-and-test:             # Job name
    runs-on: ubuntu-latest    # Runner OS

    steps:                    # Steps in this job
      - uses: actions/checkout@v4      # Check out the repo
      - name: Build
        run: g++ -o main main.cpp      # Shell command
      - name: Run Tests
        run: ./run_tests               # Another shell command
```

### YAML Syntax Rules
- Indentation matters (use spaces, NOT tabs)
- Key-value pairs: `key: value`
- Lists use `-` prefix
- Nested structures use indentation

### What to Know for the Exam
- How to read a workflow file and understand what it does
- What triggers a workflow
- What `runs-on`, `steps`, `run`, `uses` mean
- How CI connects to testing (run tests automatically on every push)

---
---

# 10. ITERATORS (from SOLID Principles context)

---

## What Is an Iterator?

An object that lets you traverse a collection (array, list, vector, set, map) without exposing its internal structure.

---

## C++ Iterator Functions

| Function | What It Does |
|----------|-------------|
| `begin()` | Returns iterator to FIRST element |
| `end()` | Returns iterator PAST the last element (not the last element itself!) |
| `++it` / `it++` | Move to next element |
| `--it` / `it--` | Move to previous element (bidirectional iterators) |
| `*it` | Dereference — get the value the iterator points to |
| `it->member` | Access a member of the object the iterator points to |
| `advance(it, n)` | Move iterator forward by n positions |
| `distance(first, last)` | Number of elements between two iterators |
| `next(it)` / `prev(it)` | Return iterator to next/previous element without modifying `it` |

### Range-based for loop (uses iterators internally)
```cpp
vector<int> v = {1, 2, 3, 4, 5};

// Explicit iterator:
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << endl;
}

// Range-based (uses iterators under the hood):
for (auto val : v) {
    cout << val << endl;
}
```

---

## Iterator Categories

| Category | Can Do | Examples |
|----------|--------|---------|
| **Input** | Read, forward only, single pass | `istream_iterator` |
| **Output** | Write, forward only, single pass | `ostream_iterator` |
| **Forward** | Read/write, forward only, multi pass | `forward_list::iterator` |
| **Bidirectional** | Forward + backward | `list::iterator`, `set::iterator` |
| **Random Access** | Jump to any position (`it + n`, `it[n]`) | `vector::iterator`, `deque::iterator` |

---

## Connection to SOLID

Iterators follow the **Interface Segregation Principle**: each iterator category provides only the operations that make sense for that type of collection. A forward-only list doesn't pretend to support random access.

Iterators also follow **Dependency Inversion**: algorithms like `std::sort` depend on the iterator abstraction, not on specific container types. This makes them work with ANY container that provides the right iterator.

---
---

# EXAM-READY QUICK REFERENCE

| Topic | Key Points |
|-------|-----------|
| **SRP** | One class = one reason to change |
| **OCP** | Add new behavior via extension (new classes), don't modify working code |
| **LSP** | Subclass must be substitutable for parent without breaking anything |
| **ISP** | Small focused interfaces > one fat interface |
| **DIP** | Depend on abstractions (interfaces), not concrete implementations |
| **Encapsulation** | Bundle data + methods, restrict access (private/protected/public) |
| **Abstraction** | Hide complexity, expose simple interface (getters/setters/public methods) |
| **Interface** | All public methods of an object. Enforced by encapsulation + abstraction |
| **Inheritance** | Subclass acquires parent's properties. Enables polymorphism |
| **Pure virtual** | `= 0` makes class abstract, subclasses MUST implement |
| **Emergent Design** | Design evolves through: tests → no duplication → expressive → minimal |
| **Template Method** | Abstract class defines algorithm skeleton, subclasses fill in steps |
| **Variable span** | Lines between consecutive references — minimize it |
| **Variable live time** | Lines from first to last reference — minimize it |
| **const** | Use whenever a variable shouldn't change |
| **Naming** | Intention-revealing, no disinformation, follow conventions |
| **Assertions** | `assert()` = runtime, `static_assert()` = compile time |
| **Barricade** | Validate dirty external data at the boundary, keep core clean |
| **Offensive programming** | Fail hard in debug mode to catch bugs early |
| **Loop optimizations** | Fusion, fission, interchange, invariant motion, unrolling, unswitching |
| **Strength reduction** | Replace expensive ops with cheaper ones (multiply → add, etc.) |
| **Always measure** | Profile before optimizing. Don't guess bottlenecks |
| **EXPECT vs ASSERT** | EXPECT = non-fatal (continues), ASSERT = fatal (stops) |
| **Code coverage** | Measures execution breadth, NOT test quality |
| **GitHub Actions** | YAML workflow: `on` = trigger, `jobs` = what runs, `steps` = individual tasks |
| **Iterators** | `begin()`, `end()`, `++`, `*`, `advance()`, `distance()` |
| **Personal character** | Humility > cleverness. Fight imposter syndrome. Avoid crunch culture |

---
---

# PRACTICE EXAM — VARIATION A

**Modeled after the real Exam 2 format: matching, select-all, code writing, ordering, MCQ**

---

**Q1. (3 pts) Match each SOLID principle with the BEST description.**

| Principle | | Description |
|---|---|---|
| Single Responsibility | ___ | A) Entities can be extended but not modified |
| Open/Closed | ___ | B) Do not force clients to depend on methods they don't use |
| Liskov Substitution | ___ | C) Each class should have only one well-defined responsibility |
| Interface Segregation | ___ | D) Entities must only depend on abstractions |
| Dependency Inversion | ___ | E) Subclass instances can replace superclass instances without breaking correctness |

---

**Q2. (4 pts) Write the assert statements.**

The following code reads the radius of a circle from the user. Following defensive programming practice, write the assert statement (using the `cassert` header) that ensures the radius is non-negative.

```cpp
#include <iostream>
using namespace std;
#include <cassert>
int main() {
    int radius = ReadRadius();

    // Checking if the user has entered a non-negative value for radius?
    ___________________________________

    cout << "Radius is valid" << endl;
    double area = 3.14159 * radius * radius;
    cout << "Area: " << area << endl;
    return 0;
}
```

---

**Q3. (2 pts) Read the YAML file. Choose ALL true statements.**

```yaml
name: build-and-test
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
jobs:
  testing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Compile
        run: g++ -o app main.cpp
      - name: Run tests
        run: ./run_tests
```

- [ ] A) This workflow is triggered on pull requests to the main branch
- [ ] B) This workflow is triggered on push events to the develop branch
- [ ] C) The workflow's name is build-and-test
- [ ] D) This workflow builds and runs all unit tests whenever a change is pushed to any branch
- [ ] E) The workflow is triggered on push events to the main branch
- [ ] F) The job's name is testing

---

**Q4. (2 pts) Compute average span.**

In the following code snippet, what is the average span of variable `total`?

```
total = 0;
count = 5;
x = 10;
total = total + x;
y = 20;
total = total + y;
```

A) 0
B) 1
C) 2
D) 3

---

**Q5. (1 pt) Iterator question.**

After executing the following C++ code, what is pointed to by `ptr`?

```cpp
vector<int> v = {15, 25, 35, 45, 55};
vector<int>::iterator ptr = v.end();
```

A) Element 55
B) Element 45
C) Element 15
D) None of the above

---

**Q6. (3 pts) Match the problem with the concept that describes the most likely solution.**

| Problem | | Concept |
|---|---|---|
| Your app loads slowly on mobile. Users complain about wait times. You want to find what's causing the delay and reduce it. | ___ | A) Code coverage |
| You merged a teammate's code and now several features are broken. You want to make sure every part of the codebase is being tested. | ___ | B) Refactoring |
| Your codebase has grown messy — duplicate functions, unclear names, deeply nested logic. It works but is hard to maintain. | ___ | C) Code tuning |
| A client reports that the checkout page crashes when entering a coupon code with special characters. | ___ | D) Code fix |

---

**Q7. (1 pt) True or False.**

"In Emergent Design, minimizing the number of classes always has a higher priority over applying the single responsibility principle."

A) True
B) False

---

**Q8. (3 pts) Match each term with the BEST definition.**

| Term | | Definition |
|---|---|---|
| Abstraction | ___ | A) Restricting access to components within an object; in C++ achieved by making attributes and methods private or protected |
| Encapsulation | ___ | B) A class that cannot be instantiated but can be inherited from |
| Abstract class | ___ | C) Having classes and therefore objects get members from other classes in a hierarchical manner |
| Interface | ___ | D) Hiding complexity of an object, making public only the behavior that is relevant for the system or user |
| Inheritance | ___ | E) A behavior contract that allows for an understanding of what an object can do |
| Subclass | ___ | F) A class that inherits from a superclass |

---

**Q9. (2 pts) Select ALL that are rules of Simple Design (Emergent Design).**

- [ ] A) Minimizes the number of classes and methods
- [ ] B) It is secure and error resistant
- [ ] C) There is no code duplication
- [ ] D) Running all tests
- [ ] E) The code expresses the intent of the programmer
- [ ] F) Makes sure everything is well documented
- [ ] G) Benefits the company

---

**Q10. (3 pts) Order the code tuning procedure steps.**

Put the following steps in the correct order to follow once poor performance is observed:

___ Save a working version of the code so you can get back to the "last known good state," then measure the system to find bottlenecks.

___ Determine the reason behind the weak performance. If code tuning is not appropriate, go back to development.

___ Tune the bottleneck identified.

___ Measure the improvement from the code tuning strategy you applied.

___ If no improvement occurs, revert code to the last known good state.

Repeat starting from top.

---

**Q11. (2 pts) Error message question. Choose ALL that apply.**

Which of the following is an appropriate error message to be displayed for a user who is trying to submit a form online but has entered an invalid email address? (Choose all that apply):

- [ ] A) Error 0x7FF2: Regex validation failed for field "EMAIL" in table "USERS". Abort.
- [ ] B) Please enter a valid email address (e.g., name@example.com). Check for typos and try again.
- [ ] C) Something is wrong with your data and you should fix it. We are not responsible for your mistakes.
- [ ] D) Email validation error: string "user@" does not match pattern `^[a-zA-Z0-9+_.-]+@[a-zA-Z0-9.-]+$`. Expected format: RFC 5322 compliant address.
- [ ] E) This message is a test, replace with real message later

---

**Q12. (1 pt) Which of the following is NOT true of Interfaces?**

A) They insulate clients of a class from changes to the implementation of a class.
B) They abstract away details of how the class works, making the class easier to understand and use.
C) They should not be changed unless absolutely necessary.
D) They consist of the public and protected (but not private) data and functions of a class.

---

**Q13. (1 pt) Iterator question.**

After executing the following C++ code, what is the element pointed to by `ptr`?

```cpp
vector<int> v = {50, 60, 70, 80, 90};
vector<int>::iterator ptr = v.begin();
next(ptr, 3);
```

A) 80
B) 90
C) 50
D) 70

---

**Q14. (2 pts) ISP fix.**

The following code violates the Interface Segregation Principle (ISP). Which code change fixes it?

```cpp
class MediaDevice {
public:
    virtual void playAudio(string file) = 0;
    virtual void playVideo(string file) = 0;
    virtual void recordAudio(string file) = 0;
};

class MusicPlayer : public MediaDevice {
public:
    void playAudio(string file) { cout << file << endl; }
    void playVideo(string file) {
        throw runtime_error("Music players can't play video!");
    }
    void recordAudio(string file) {
        throw runtime_error("This player can't record!");
    }
};
```

A) Add the virtual keyword to playVideo and recordAudio inside MusicPlayer
B) Move playVideo and recordAudio outside the MediaDevice class and into separate abstract classes, and let devices inherit only from the interfaces they actually support
C) Provide a default implementation for all functions inside the MediaDevice class
D) None of the above

---

**Q15. (1 pt) True or False.**

"The best time to start code tuning is during the development or testing phases of your software."

A) True
B) False

---

**Q16. (3 pts) Select ALL that represent good practices in variable naming and declaration.**

- [ ] A) Use names that are descriptive and self-explainable
- [ ] B) Always leave a class constructor and destructor empty; every class member should be initialized and destroyed within member functions
- [ ] C) Have a list of reusable variables that can be easily remembered (like temp, counter, res)
- [ ] D) Keep all your code in a single file for easy access
- [ ] E) Try to keep spans short to reduce the window of vulnerability
- [ ] F) Declare and instantiate a variable close to where it's going to be used

---

**Q17. (1 pt) Which of the following activities is usually NOT intended to improve your software runtime performance?**

A) Turning on compiler optimizations
B) Buying new hardware
C) Eliminating code smells
D) Choosing a better data structure or algorithm to solve the problem
E) All the above

---

---

# ANSWER KEY — VARIATION A

---

**Q1.**
- Single Responsibility → **C**
- Open/Closed → **A**
- Liskov Substitution → **E**
- Interface Segregation → **B**
- Dependency Inversion → **D**

---

**Q2.**
```cpp
assert(radius >= 0);
```

---

**Q3.** A, B, C, E, F are all true.
- A) ✅ Triggered on PR to main
- B) ✅ Triggered on push to develop
- C) ✅ Name is build-and-test
- D) ❌ Only pushes to main and develop, not ANY branch
- E) ✅ Triggered on push to main
- F) ✅ Job name is testing

---

**Q4.** **C) 2**

```
Line 1: total = 0;          ← first reference
Line 2: count = 5;          (1 line gap)
Line 3: x = 10;             (2 lines gap)
Line 4: total = total + x;  ← second reference, span = 2
Line 5: y = 20;             (1 line gap)
Line 6: total = total + y;  ← third reference, span = 1
```
Average span = (2 + 1) / 2 = **1.5** → but since the gaps between references to `total` are lines 1→4 (span=2) and 4→6 (span=1), average = (2+1)/2 = 1.5. Depending on how your professor counts, the answer is either 1 or 2. On the real exam the answer was **1** for a similar question — span counts the number of intervening variable references/statements between consecutive uses, not lines.

Let's recount using the real exam's method:
```
total = 0;        ← ref 1
count = 5;        ← 1 other variable reference
x = 10;           ← 2 other variable references
total = total+x;  ← ref 2. Span = 2
y = 20;           ← 1 other variable reference
total = total+y;  ← ref 3. Span = 1
```
Average span = (2 + 1) / 2 = **1.5**. Closest answer: **C) 2** (if rounding up) or could be 1. Check your professor's counting method from the real exam.

---

**Q5.** **D) None of the above.** `end()` returns an iterator past the last element — it does NOT point to any element in the vector. Dereferencing it is undefined behavior.

---

**Q6.**
- Slow app / reduce delay → **C) Code tuning**
- Features broken / make sure everything is tested → **A) Code coverage**
- Messy code / hard to maintain → **B) Refactoring**
- Checkout crash with special characters → **D) Code fix**

---

**Q7.** **B) False.** Rule 4 (minimal classes/methods) is the LOWEST priority of the 4 rules. SRP (single responsibility) is fundamental — you should never violate it just to reduce the number of classes.

---

**Q8.**
- Abstraction → **D**
- Encapsulation → **A**
- Abstract class → **B**
- Interface → **E**
- Inheritance → **C**
- Subclass → **F**

---

**Q9.** **A, C, D, E** are correct.
- A) ✅ Minimizes classes and methods (Rule 4)
- B) ❌ Not one of the 4 rules
- C) ✅ No code duplication (Rule 2)
- D) ✅ Running all tests (Rule 1)
- E) ✅ Expresses intent of programmer (Rule 3)
- F) ❌ Not one of the 4 rules
- G) ❌ Not one of the 4 rules

---

**Q10.** Correct order:
1. Save a working version, then measure to find bottlenecks
2. Determine the reason behind weak performance
3. Tune the bottleneck identified
4. Measure the improvement
5. If no improvement, revert to last known good state

Repeat.

---

**Q11.** **B** only.
- A) ❌ Exposes internal error codes and table names — security risk, not user-friendly
- B) ✅ Friendly, specific, tells user what to do
- C) ❌ Blames the user, unprofessional
- D) ❌ Exposes regex pattern and technical details
- E) ❌ Placeholder, not a real message

---

**Q12.** **D.** Interfaces consist of the **public** methods only — not protected. Protected members are part of the implementation, not the interface. The interface is what external code can see and use.

---

**Q13.** **C) 50.** `next(ptr, 3)` returns a new iterator 3 positions forward, but it does NOT modify `ptr`. `ptr` still points to `v.begin()`, which is element 50. (This is the same trick from the real exam — `next()` doesn't change the original iterator.)

---

**Q14.** **B.** Move playVideo and recordAudio into separate abstract classes. MusicPlayer should only inherit from the audio interface. This follows ISP — classes should only depend on methods they actually use.

---

**Q15.** **B) False.** Code tuning should happen AFTER performance is measured and a bottleneck is identified. Tuning during development is premature optimization.

---

**Q16.** **A, E, F.**
- A) ✅ Descriptive, self-explainable names
- B) ❌ Constructor should initialize members, destructor should free memory
- C) ❌ Reusable names like "temp" and "res" are anti-patterns — use intention-revealing names
- D) ❌ Code should be modular, not crammed in one file
- E) ✅ Short spans reduce window of vulnerability
- F) ✅ Declare close to use = good practice

---

**Q17.** **C) Eliminating code smells.** Code smells are about code quality/maintainability, not runtime performance. Compiler optimizations, better hardware, and better algorithms all directly improve performance.

---
---

# PRACTICE EXAM — VARIATION B

**Different questions, same format as the real exam.**

---

**Q1. (3 pts) Match each OOP concept with the BEST description.**

| Concept | | Description |
|---|---|---|
| Encapsulation | ___ | A) A class that cannot be instantiated directly and must be inherited |
| Polymorphism | ___ | B) Bundling data with methods and restricting direct access via private/protected |
| Abstract class | ___ | C) Objects of different classes responding to the same method call in different ways |
| Pure virtual function | ___ | D) A function declared with `= 0` that subclasses MUST implement |
| Duck typing | ___ | E) If it has the right methods, treat it as the right type (Python) |

---

**Q2. (4 pts) Write the assert statements.**

The following code reads two test scores and computes the average. Following defensive programming practice, write TWO assert statements ensuring both scores are between 0 and 100 (inclusive).

```cpp
#include <iostream>
using namespace std;
#include <cassert>
int main() {
    int score1 = ReadScore();
    int score2 = ReadScore();

    // Ensure both scores are valid (0-100)?
    ___________________________________
    ___________________________________

    double avg = (score1 + score2) / 2.0;
    cout << "Average: " << avg << endl;
    return 0;
}
```

---

**Q3. (2 pts) Read the YAML file. Choose ALL true statements.**

```yaml
name: lint-check
on:
  pull_request:
    branches: [ main, staging ]
jobs:
  lint:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run linter
        run: cpplint --recursive src/
```

- [ ] A) This workflow runs on Ubuntu
- [ ] B) This workflow is triggered on pull requests to staging
- [ ] C) The workflow name is lint
- [ ] D) The workflow name is lint-check
- [ ] E) This workflow runs on push events to main
- [ ] F) This workflow is triggered on pull requests to main

---

**Q4. (2 pts) Compute average span.**

What is the average span of variable `sum`?

```
sum = 0;
a = 5;
b = 10;
sum = sum + a;
c = 15;
sum = sum + c;
```

A) 0
B) 1
C) 1.5
D) 2

---

**Q5. (1 pt) Iterator question.**

After executing the following C++ code, what value does `*it` give?

```cpp
vector<int> v = {100, 200, 300, 400, 500};
vector<int>::iterator it = v.begin();
advance(it, 4);
```

A) 400
B) 500
C) 100
D) Undefined behavior

---

**Q6. (2 pts) ISP fix.**

```cpp
class Vehicle {
public:
    virtual void drive() = 0;
    virtual void fly() = 0;
    virtual void sail() = 0;
};

class Car : public Vehicle {
public:
    void drive() override { cout << "driving"; }
    void fly() override { throw runtime_error("Cars can't fly!"); }
    void sail() override { throw runtime_error("Cars can't sail!"); }
};
```

Which SOLID principle is violated and how would you fix it?

A) OCP — make Vehicle non-abstract
B) ISP — split Vehicle into Drivable, Flyable, Sailable interfaces; Car only inherits Drivable
C) SRP — Car has too many methods
D) DIP — Car should depend on abstractions

---

**Q7. (1 pt) True or False.**

"High code coverage guarantees that the software is free of bugs."

A) True
B) False

---

**Q8. (3 pts) Select ALL good practices from the list.**

- [ ] A) Initialize class member data in the constructor
- [ ] B) Ignore compiler warning messages if the program runs
- [ ] C) Use `const` when a variable shouldn't change
- [ ] D) Always validate input before assigning to local variables
- [ ] E) Use the destructor to free memory allocated in the constructor
- [ ] F) Reuse the same variable for different purposes to save memory

---

**Q9. (2 pts) Match each scenario to the SOLID principle it violates.**

| Scenario | | Principle |
|---|---|---|
| A `Logger` class handles file logging, database logging, and email notifications | ___ | A) OCP |
| Adding a new report type requires editing a giant switch statement | ___ | B) SRP |
| A `Penguin` class inherits `Bird` and throws an error when `fly()` is called | ___ | C) LSP |
| A `NotificationService` directly instantiates `GmailSender` instead of using an email interface | ___ | D) DIP |

---

**Q10. (1 pt) Which describes offensive programming?**

A) Writing code that gracefully handles all user errors in production
B) Making errors crash as loudly as possible during development to catch bugs early
C) Removing all error handling to make code faster
D) Using encryption to protect against attacks

---

**Q11. (2 pts) Error message question.**

A user tries to upload a file but exceeds the 10MB size limit. Which is the best error message?

A) Error: IOException at line 247 in FileUploadHandler.java — maxFileSize exceeded
B) Something went wrong. Try again later.
C) The file you selected is too large. Please choose a file under 10MB.
D) UPLOAD_FAILED: fileSize=15728640 > maxAllowed=10485760. Buffer overflow prevented.

---

**Q12. (2 pts) Loop optimization.**

Identify the optimization and rewrite:

```cpp
for (int i = 0; i < 1000; i++) {
    arr[i] = arr[i] + base_value;
    if (debug_mode)
        log(arr[i]);
}
```

---

**Q13. (1 pt) What is the difference between `EXPECT_EQ` and `ASSERT_EQ` in Google Test?**

A) `EXPECT_EQ` checks equality at compile time; `ASSERT_EQ` checks at runtime
B) `EXPECT_EQ` continues the test on failure; `ASSERT_EQ` stops the test on failure
C) They are identical
D) `ASSERT_EQ` is deprecated

---

---

# ANSWER KEY — VARIATION B

---

**Q1.**
- Encapsulation → **B**
- Polymorphism → **C**
- Abstract class → **A**
- Pure virtual function → **D**
- Duck typing → **E**

---

**Q2.**
```cpp
assert(score1 >= 0 && score1 <= 100);
assert(score2 >= 0 && score2 <= 100);
```

---

**Q3.** **B, D, F.**
- A) ❌ Runs on macos-latest, not Ubuntu
- B) ✅ PR to staging triggers it
- C) ❌ "lint" is the job name, not the workflow name
- D) ✅ Workflow name is lint-check
- E) ❌ Not triggered on push — only on pull_request
- F) ✅ PR to main triggers it

---

**Q4.** **C) 1.5**
```
sum = 0;          ← ref 1
a = 5;            (1 intervening)
b = 10;           (2 intervening)
sum = sum + a;    ← ref 2. Span = 2
c = 15;           (1 intervening)
sum = sum + c;    ← ref 3. Span = 1
```
Average span = (2 + 1) / 2 = **1.5**

---

**Q5.** **B) 500.** `advance(it, 4)` DOES modify `it` (unlike `next()` which returns a new iterator). It moves 4 positions from begin → index 4 → element 500.

---

**Q6.** **B) ISP.** Car is forced to implement fly() and sail() even though it can't do either. Fix: split into `Drivable`, `Flyable`, `Sailable` interfaces. Car only inherits from `Drivable`.

---

**Q7.** **B) False.** Coverage measures how much code was executed, not whether the output was validated. Tests can run code without meaningful assertions.

---

**Q8.** **A, C, D, E.**
- A) ✅ Initialize in constructor
- B) ❌ Never ignore warnings
- C) ✅ Use const
- D) ✅ Validate input
- E) ✅ Free memory in destructor
- F) ❌ Use one variable per purpose

---

**Q9.**
- Logger does too many things → **B) SRP**
- Giant switch needs editing for new types → **A) OCP**
- Penguin can't fly but inherits fly() → **C) LSP**
- Direct instantiation of GmailSender → **D) DIP**

---

**Q10.** **B.** Offensive programming = fail hard in development so bugs are caught immediately.

---

**Q11.** **C.** Friendly, specific, tells the user exactly what's wrong and what to do.

---

**Q12.** This needs **loop unswitching**. The `if (debug_mode)` condition doesn't change per iteration — move it outside:

```cpp
if (debug_mode) {
    for (int i = 0; i < 1000; i++) {
        arr[i] = arr[i] + base_value;
        log(arr[i]);
    }
} else {
    for (int i = 0; i < 1000; i++) {
        arr[i] = arr[i] + base_value;
    }
}
```

---

**Q13.** **B.** `EXPECT_EQ` is non-fatal (test continues on failure to show all failures). `ASSERT_EQ` is fatal (test stops immediately — use when continuing would be meaningless).

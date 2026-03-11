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

# PRACTICE EXAM

**Mix of Multiple Choice + Free Response**

---

## PART A: MULTIPLE CHOICE (Q1–Q30)

---

**1.** Which SOLID principle states that a class should have only one reason to change?

A) Open/Closed Principle
B) Liskov Substitution Principle
C) Single Responsibility Principle
D) Dependency Inversion Principle

---

**2.** A function uses a long if/else chain to handle different shape types. Every time a new shape is added, this function must be edited. Which SOLID principle is being violated?

A) SRP
B) OCP
C) ISP
D) LSP

---

**3.** A `Square` class inherits from `Rectangle` and overrides `setWidth()` to also change the height. Code using a `Rectangle*` pointer breaks when it receives a `Square`. Which principle is violated?

A) SRP
B) OCP
C) LSP
D) DIP

---

**4.** An `Animal` interface requires `fly()`, `swim()`, and `run()`. A `Dog` class is forced to implement `fly()` even though dogs can't fly. Which principle is violated?

A) SRP
B) OCP
C) LSP
D) ISP

---

**5.** A class `OrderService` directly creates a `MySQLDatabase` object inside its constructor. Which SOLID principle does this violate?

A) SRP
B) LSP
C) ISP
D) DIP

---

**6.** In C++, which access modifier makes a member accessible to subclasses but NOT to code outside the class?

A) public
B) private
C) protected
D) friend

---

**7.** What is the purpose of a pure virtual function (`= 0`) in C++?

A) It provides a default implementation that subclasses can optionally override
B) It makes the class abstract and forces subclasses to provide an implementation
C) It prevents the function from being called
D) It makes the function run faster

---

**8.** In Python, if an object has a `sound()` method, Python will call it regardless of whether the class inherits from `Animal`. This behavior is called:

A) Polymorphism
B) Encapsulation
C) Duck typing
D) Dependency injection

---

**9.** What is the correct priority order of the 4 Rules of Simple Design (Emergent Design)?

A) No duplication → Run tests → Expressive → Minimal
B) Run tests → No duplication → Expressive → Minimal
C) Expressive → Minimal → No duplication → Run tests
D) Minimal → Run tests → Expressive → No duplication

---

**10.** Which design pattern breaks an algorithm into steps, puts them in an abstract class, and lets subclasses override specific steps?

A) Strategy Pattern
B) Observer Pattern
C) Template Method Pattern
D) Singleton Pattern

---

**11.** Which is NOT a disadvantage of emergent design?

A) No plan for changes can cause design to deteriorate
B) Ignores crucial dependencies
C) Requires too much upfront planning
D) Lack of coordination can cause development deadlocks

---

**12.** The "span" of a variable refers to:

A) The total number of lines in the function
B) The number of lines between consecutive references to that variable
C) The number of times the variable is used
D) The scope of the variable

---

**13.** Why should you minimize the "live time" of a variable?

A) It makes the program run faster
B) It reduces the window of vulnerability where bugs can be introduced
C) It uses less memory
D) It satisfies the compiler

---

**14.** According to C++ naming conventions, which is correct?

A) Constants use camelCase: `maxSize`
B) Class names use ALL_CAPS: `MY_CLASS`
C) Constants use ALL_CAPS: `MAX_SIZE`
D) Variable names use MixedCase: `VariableName`

---

**15.** What does `static_assert` do in C++?

A) Checks a condition at runtime and aborts if false
B) Checks a condition at compile time and fails compilation if false
C) Logs a warning message at runtime
D) Disables assertions in release builds

---

**16.** In defensive programming, a "barricade" refers to:

A) A firewall that blocks network attacks
B) A validation layer at the boundary that separates dirty external data from the clean core system
C) A class that prevents inheritance
D) A lock that prevents concurrent access

---

**17.** Which error-handling approach is most appropriate for a safety-critical system (e.g., medical device) when invalid input is detected?

A) Return a neutral value and continue
B) Log a warning to a file
C) Shut down the system
D) Return the same answer as last time

---

**18.** In offensive programming, you should:

A) Hide all errors from users
B) Make errors fail as loudly and visibly as possible during development
C) Catch all exceptions silently
D) Only test the happy path

---

**19.** What is loop-invariant code motion?

A) Splitting a loop into two separate loops
B) Combining two loops into one
C) Moving computations that don't change per iteration outside the loop
D) Reversing the order of loop iterations

---

**20.** Which loop optimization improves cache performance for multi-dimensional arrays?

A) Loop fusion
B) Loop fission
C) Loop interchange
D) Loop unrolling

---

**21.** Strength reduction replaces:

A) Loops with recursion
B) Expensive operations with cheaper equivalents (e.g., multiplication with addition)
C) Variables with constants
D) Dynamic allocation with stack allocation

---

**22.** What should you ALWAYS do before optimizing code?

A) Rewrite the entire module
B) Measure and profile to find the actual bottleneck
C) Optimize every function equally
D) Remove all comments

---

**23.** In Google Test, what is the difference between `EXPECT_EQ` and `ASSERT_EQ`?

A) They do the same thing
B) `EXPECT_EQ` is non-fatal (test continues on failure); `ASSERT_EQ` is fatal (test stops)
C) `ASSERT_EQ` is non-fatal; `EXPECT_EQ` is fatal
D) `EXPECT_EQ` is for integers only; `ASSERT_EQ` is for strings

---

**24.** High code coverage means:

A) The code is bug-free
B) Many lines/branches were executed by tests, but not necessarily validated correctly
C) Every edge case has been tested
D) The architecture is well-designed

---

**25.** In a GitHub Actions YAML file, what does `on: push` specify?

A) The operating system to use
B) The event that triggers the workflow
C) The name of the job
D) The test framework to use

---

**26.** What does `end()` return in C++?

A) An iterator to the last element
B) An iterator past the last element
C) The value of the last element
D) The size of the container

---

**27.** Which iterator category supports jumping to any position (e.g., `it + 5`, `it[3]`)?

A) Input iterator
B) Forward iterator
C) Bidirectional iterator
D) Random access iterator

---

**28.** According to the Personal Character lecture, the best programmer is:

A) The most intelligent one
B) The one who writes the cleverest code
C) The most humble one — with discipline, curiosity, and experience
D) The one with the most years of experience

---

**29.** "Crunch culture" in software development refers to:

A) Optimizing code for performance
B) Extended periods of overwork (60–80+ hour weeks)
C) Compressing code into fewer files
D) Reducing team size

---

**30.** Which assertion should you use in Google Test when a null pointer check must pass before the rest of the test can run?

A) `EXPECT_NE(ptr, nullptr)` — let the test continue even if it fails
B) `ASSERT_NE(ptr, nullptr)` — stop the test immediately if it fails
C) `EXPECT_TRUE(ptr)` — always use EXPECT
D) No assertion needed — just dereference and hope

---
---

## PART B: FREE RESPONSE

---

**31.** A class called `ReportManager` does the following:
- Fetches data from a database
- Performs calculations on the data
- Formats the results into a PDF
- Sends the PDF via email

Identify which SOLID principle this violates and explain why. Propose a redesign.

---

**32.** Look at the following code:

```cpp
for (int i = 0; i < n; i++) {
    x = y + z;
    result[i] = x * x + i;
}
```

Identify the code tuning optimization that should be applied here. Rewrite the optimized version and explain why it's faster.

---

**33.** Explain the difference between `assert()` (dynamic assertion) and `static_assert()` in C++. For each, give one example of when you would use it.

---

**34.** A student writes the following Google Test:

```cpp
TEST(MathTest, TestAdd) {
    add(2, 3);
}
```

The test passes with 100% code coverage of the `add()` function. Explain why this test is bad despite achieving full coverage.

---

**35.** Explain what a "barricade" is in defensive programming. Draw or describe the flow of data from an external source to the core system, and explain what kind of checking happens at each stage.

---

**36.** A developer needs to add a new payment method (Apple Pay) to an e-commerce system. The current code looks like:

```cpp
double processPayment(string type, double amount) {
    if (type == "credit") { /* credit card logic */ }
    else if (type == "paypal") { /* paypal logic */ }
    // Must add Apple Pay here by editing this function
}
```

Which SOLID principle is being violated? Rewrite the code (or describe the redesign) so that adding Apple Pay doesn't require modifying existing code.

---

**37.** Explain the connection between iterators and the Interface Segregation Principle. Why do C++ iterators come in different categories (input, forward, bidirectional, random access) instead of one universal iterator with all capabilities?

---

**38.** True or False: Emergent design means "no design." Explain your answer and describe how the 4 Rules of Simple Design prevent design from deteriorating.

---
---

# ANSWER KEY

---

## PART A: MULTIPLE CHOICE

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **C** | SRP = single responsibility = one reason to change |
| 2 | **B** | OCP = open for extension, closed for modification. Adding a new shape should NOT require editing existing code |
| 3 | **C** | LSP = subclass must be substitutable for parent without breaking behavior. Square breaks Rectangle's contract |
| 4 | **D** | ISP = don't force clients to depend on methods they don't use. Dog shouldn't need fly() |
| 5 | **D** | DIP = depend on abstractions, not concretions. OrderService should depend on a Database interface, not MySQL directly |
| 6 | **C** | Protected = accessible to subclasses but not outside the class hierarchy |
| 7 | **B** | Pure virtual (`= 0`) makes the class abstract — it can't be instantiated and subclasses MUST implement the function |
| 8 | **C** | Duck typing = if it has the right methods, Python treats it as the right type regardless of inheritance |
| 9 | **B** | Tests first, then refactor duplication, then make it expressive, then minimize |
| 10 | **C** | Template Method = abstract class defines skeleton, subclasses fill in specific steps |
| 11 | **C** | Emergent design specifically avoids too much upfront planning — that's its whole point |
| 12 | **B** | Span = lines between consecutive references to the same variable |
| 13 | **B** | Shorter live time = smaller window where a variable can be incorrectly modified |
| 14 | **C** | C++ convention: constants, typedefs, macros in ALL_CAPS |
| 15 | **B** | static_assert is checked at compile time — fails compilation if false |
| 16 | **B** | Barricade = validation boundary between untrusted external data and clean internal system |
| 17 | **C** | Safety-critical systems should shut down rather than operate with invalid data |
| 18 | **B** | Offensive programming = make bugs as visible as possible during development |
| 19 | **C** | Loop-invariant code motion = move constant computations outside the loop |
| 20 | **C** | Loop interchange = swap inner/outer loops for better cache locality (row-major access) |
| 21 | **B** | Strength reduction = replace expensive ops (multiply) with cheaper ones (add) |
| 22 | **B** | Always measure/profile before optimizing. Don't guess where bottlenecks are |
| 23 | **B** | EXPECT = non-fatal (continues), ASSERT = fatal (stops test immediately) |
| 24 | **B** | Coverage = execution breadth, NOT correctness. Tests can execute code without validating it |
| 25 | **B** | `on:` specifies the trigger event for the workflow |
| 26 | **B** | `end()` returns an iterator PAST the last element (one beyond), not TO the last element |
| 27 | **D** | Random access iterators support `+`, `-`, `[]` — jumping to any position |
| 28 | **C** | The lecture's core message: humility, discipline, curiosity > raw intelligence |
| 29 | **B** | Crunch = extended overwork periods, common in game dev, leads to burnout |
| 30 | **B** | ASSERT stops the test if it fails — essential when continuing would crash (null dereference) |

---

## PART B: FREE RESPONSE

---

**31.** This violates **SRP (Single Responsibility Principle)**. `ReportManager` has FOUR reasons to change: database schema changes, calculation logic changes, PDF format changes, email service changes. Changing any one of these could break the others.

**Redesign:**
```
class DataFetcher     — responsible for getting data from the database
class DataCalculator  — responsible for performing calculations
class PDFFormatter    — responsible for formatting results into a PDF
class EmailSender     — responsible for sending emails
```
Each class has one job and one reason to change. `ReportManager` can orchestrate them, but the logic is separated.

---

**32.** This needs **loop-invariant code motion**. The expression `x = y + z` and `x * x` don't change across iterations — they're computed n times for no reason.

**Optimized:**
```cpp
x = y + z;
int t = x * x;
for (int i = 0; i < n; i++) {
    result[i] = t + i;
}
```

This is faster because `y + z` and `x * x` are now computed ONCE instead of n times. The loop body only does one addition per iteration instead of two additions and a multiplication.

---

**33.**
- **`assert()` (dynamic)** is checked at **runtime**. If the condition is false, the program aborts with an error message. Use it for conditions that should never happen during execution, like a pointer being null after allocation: `assert(ptr != nullptr);`

- **`static_assert()` (static)** is checked at **compile time**. If the condition is false, compilation fails. Use it for platform/type assumptions that must be true for the code to work correctly: `static_assert(sizeof(int) == 4, "int must be 4 bytes");`

The key difference: static_assert catches problems before the program even runs. assert catches problems during execution. static_assert cannot check runtime values (like user input); assert cannot check compile-time properties (like type sizes).

---

**34.** The test has NO assertions. It calls `add(2, 3)` but never checks if the result is correct. The function executes (100% coverage), but the test would pass even if `add(2, 3)` returned 999 or threw an exception.

This demonstrates why **coverage ≠ quality**. Coverage measures execution breadth (did the code run?), not correctness (did it produce the right output?). A proper test would be:

```cpp
TEST(MathTest, TestAdd) {
    EXPECT_EQ(add(2, 3), 5);
}
```

---

**35.** A barricade is a validation layer at the boundary between untrusted external data and the clean core system.

**Data flow:**
```
External source (user input, API, file, internet)
        │
        ▼
   [BARRICADE]
   - Validate all inputs (type, range, format)
   - Sanitize data (strip dangerous characters)
   - Reject/handle invalid data with error handling
        │
        ▼
   Clean, validated data
        │
        ▼
   [CORE SYSTEM]
   - Assumes data is clean
   - Uses assertions (not error handling) for impossible states
   - Focuses on business logic, not validation
```

Inside the barricade: use **error handling** (bad data is expected).
Inside the core: use **assertions** (bad data should be impossible — if it appears, something is seriously wrong).

---

**36.** This violates **OCP (Open/Closed Principle)**. Adding Apple Pay requires modifying the `processPayment` function, which risks breaking the existing credit card and PayPal logic.

**Redesign using polymorphism:**
```cpp
class PaymentProcessor {
public:
    virtual double process(double amount) = 0;
};

class CreditCardProcessor : public PaymentProcessor {
    double process(double amount) override { /* credit card logic */ }
};

class PayPalProcessor : public PaymentProcessor {
    double process(double amount) override { /* paypal logic */ }
};

class ApplePayProcessor : public PaymentProcessor {
    double process(double amount) override { /* apple pay logic */ }
};
```

Now adding Apple Pay = adding a new class. The existing `CreditCardProcessor` and `PayPalProcessor` are never touched. The calling code uses `PaymentProcessor*` and doesn't care which implementation it gets.

---

**37.** The Iterator categories follow ISP because each category only provides the operations that make sense for that type of collection:

- A `forward_list` can only go forward, so its iterator only supports `++` (forward iterator)
- A `list` can go both directions, so its iterator supports `++` and `--` (bidirectional)
- A `vector` has contiguous memory, so its iterator supports `+n`, `-n`, `[]` (random access)

If there was ONE universal iterator with all capabilities, a forward_list would be forced to implement `it - 5` or `it[3]` even though those operations don't make sense for a singly-linked list. They'd either throw errors or give wrong results — violating both ISP (forced to depend on unused methods) and LSP (substituting a forward iterator where random access is expected would break).

By segregating into categories, algorithms can specify exactly what they need. `std::sort` requires random access iterators. `std::find` only needs input iterators. Each algorithm depends on the minimal interface it actually uses.

---

**38.** FALSE. Emergent design does NOT mean "no design." It means design evolves iteratively through feedback and refactoring, rather than being fully planned upfront.

The 4 Rules of Simple Design prevent deterioration:

1. **Run all tests** — ensures the system works after every change. You can't accidentally break things if tests catch regressions
2. **No duplication** — forces you to refactor and consolidate shared logic, keeping the codebase clean
3. **Expressive** — requires clear naming and small functions, so the design remains understandable as it grows
4. **Minimal classes and methods** — prevents over-engineering and unnecessary complexity

Together, these rules create a feedback loop: write tests → refactor to remove duplication → make it readable → keep it small. The design "emerges" from consistently applying these rules, not from having no design at all.

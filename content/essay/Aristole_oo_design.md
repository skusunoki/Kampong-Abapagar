+++
title = "What Would Aristotle Think of Your Code?"
subtitle = "A Philosophical Defence of the Interface — and a Gentle Rebuttal of Uncle Bob"
date  = 2025-03-21
draft = false
tags = [
"OOP",
"Philosophy",
"Java",
"C++",
"ABAP",
"Design Patterns","Clean Code"
]
categories = "Software Design"
description = "Starting from Uncle Bob's 'Interface Considered Harmful', this post reinterprets OO concepts through Aristotle's Four Causes — and concludes that SAP accidentally implemented Aristotle."
slug = "aristotle-oo-design"
type = "docs"
weight = 20
+++

## What Would Aristotle Think of Your Code?


Let's be honest. Most of us have never read Aristotle. We've been too busy arguing about whether tabs or spaces are the one true path to enlightenment. But bear with me for a few minutes, because it turns out that a Greek philosopher who died in 322 BC had some surprisingly useful things to say about object-oriented design — and, inadvertently, about why Uncle Bob got one argument slightly wrong. 

The journey begins, as so many great software debates do, with a blog post.

---

## 1. Uncle Bob Walks Into a Blog Post

In January 2015, Robert C. Martin — affectionately known as Uncle Bob, less affectionately known as the reason your team has a 200-line style guide — published a piece titled ['Interface Considered Harmful'](https://blog.cleancoder.com/uncle-bob/2015/01/08/InterfaceConsideredHarmful.html).

His argument, delivered in his trademark Socratic dialogue style, goes roughly like this:

> The Java `interface` keyword exists only because the language designers were too lazy to implement proper multiple inheritance. C++ already solved the Diamond Problem decades ago. Therefore, `interface` is a hack, and if you could just inherit from multiple concrete classes, you wouldn't need it.

To illustrate, he uses the Observer pattern. In Java, if you want a widget that is also observable, you're stuck: you can `extend Widget` or you can hold a `Subject` by composition and write boring delegation code. You can't do both cleanly. In C++, you simply write:

```cpp
// C++ — clean, direct, expressive
class ObservableWidget : public Widget, public Subject {
    // Widget concerns here
    // Observer concerns here
    // Separation of concerns: preserved
};
```

Uncle Bob's verdict: the `interface` keyword is harmful. Guilty. Sentence: deprecated.

This is a compelling argument. It is also, I will argue, about 70% correct — which in software debates is practically a landslide.

---

## 2. Enter Aristotle (Surprisingly Punctual)

Aristotle proposed that any thing — a statue, a rainstorm, a moderately well-designed microservice — can be explained by four causes. Not causes in the modern billiard-ball sense, but four distinct types of explanatory answer to the question "why does this exist?"

| Cause | Aristotle's term | The marble statue analogy | In your codebase |
|---|---|---|---|
| Material Cause | Hylē (ὕλη) | The marble itself | Abstract base class (`XxxBase`) |
| Formal Cause | Morphē (μορφή) | The sculptor's design | Concrete class |
| Efficient Cause | Kinesis (κίνησις) | The sculptor's act of carving | Constructor / Factory |
| Final Cause | Telos (τέλος) | The beauty the statue is meant to embody | Interface |

A framework gives you marble. You inherit it, shape it into a concrete class, wire it together with a factory, and the whole thing is oriented towards an interface — a declaration of purpose, a commitment to a contract, a *telos*.

If that sounds a bit grand for a Tuesday afternoon coding session, good. It should. You're not just writing classes. You're participating in a very ancient conversation about what things are and why they exist.

---

## 3. The Interface is Not an Externally Imposed Prison. It's an Internally Chosen Destiny.

Uncle Bob's critique implicitly treats the interface as something imposed from outside — a bureaucratic contract that a class is forced to sign. But look at who actually does the declaring:

```java
// Java
class ObservableWidget extends WidgetBase
                       implements IObservable {
    // ObservableWidget reaches OUT to IObservable
    // and says: I embody this purpose.
}
```

```abap
* ABAP
CLASS zcl_observable_widget DEFINITION
  INHERITING FROM zcl_widget_base.
  PUBLIC SECTION.
    INTERFACES zif_observable.  " Same gesture — I choose this telos
ENDCLASS.
```

```cpp
// C++ — uniform syntax, no distinction
class ObservableWidget : public WidgetBase,   // material
                         public IObservable {  // telos
    // Both look identical. Philosophy weeps.
};
```

The implementing class *reaches out* to the interface. It is an act of commitment, not submission. The widget declares: "I will be observable. This is my purpose."

In Aristotelian terms: the telos is not external to the thing — it is immanent within it. The acorn does not have "become an oak tree" imposed on it from outside. That *is* what an acorn is for. An acorn that refuses to become a tree is not a rebel. It is a failed acorn.

> **The philosophical punchline:** The `interface` keyword, whatever its historical origin, encodes a philosophically important distinction — the difference between *receiving material* (`extends` / `INHERITING FROM`) and *committing to a purpose* (`implements` / `INTERFACES`). C++ erases this distinction with a single colon. Java and ABAP preserve it.

---

## 4. Is-A Is a Lie (And Your Type System Knows It)

The standard story of inheritance is: `Dog extends Animal` because a dog IS-A animal. Classification. Taxonomy. Linnaean hierarchy in your IDE.

This story produces the Square-Rectangle problem, which has been reliably causing arguments at whiteboard interviews since approximately 1995.

> A square is a special case of a rectangle. So `Rectangle` should be the parent class. But wait — a rectangle has two independent side lengths; a square has one. The child class has *fewer* degrees of freedom than the parent. The hierarchy is backwards. Congratulations, your taxonomy has broken your type system.

The fix is to stop thinking about inheritance as classification and start thinking about it as **material succession**:

```java
// Java — inheritance as material, not taxonomy
abstract class QuadrilateralBase {
    // raw material: four sides, four angles
}

class Rectangle extends QuadrilateralBase { ... }
class Square    extends QuadrilateralBase { ... }

// Rectangle is NOT a kind of Square.
// Rectangle is MADE FROM the material of QuadrilateralBase.
// The constructor (Efficient Cause) validates the shape.
```

```abap
* ABAP — same principle
CLASS zcl_quadrilateral_base DEFINITION ABSTRACT.
  " raw material: four sides
ENDCLASS.

CLASS zcl_rectangle DEFINITION
  INHERITING FROM zcl_quadrilateral_base.
  PUBLIC SECTION.
    METHODS constructor
      IMPORTING iv_width  TYPE decfloat16
                iv_height TYPE decfloat16.
ENDCLASS.

CLASS zcl_square DEFINITION
  INHERITING FROM zcl_quadrilateral_base.
  PUBLIC SECTION.
    METHODS constructor
      IMPORTING iv_side TYPE decfloat16.
ENDCLASS.
```

Under this view, *type* — in the strong, semantic sense — belongs to interfaces, not to class hierarchies. `IHasArea` is a type. `IResizable` is a type. `Rectangle` is a material fact about implementation, not a type commitment.

This is, incidentally, why value objects are so satisfying to write. A value object like `Length` or `Money` makes a complete commitment: it implements an interface (*telos*), it is shaped by a constructor that validates it (*efficient cause*), and it never changes — because it has already fully achieved its purpose. In Aristotelian terms, a value object is pure actuality — no residual potentiality. Aristotle called this state *entelecheia*, and reserved it for God. In software, we just call it immutable.

The Square-Rectangle problem is a canonical example of LSP violation[^martin2002][^wikipedia],
first formalized by Liskov and Wing[^liskov1994]. Cook et al. established
that inheritance and subtyping are orthogonal relations[^cook1990] — a result
this essay converges with, though by a different route.

[^martin2002]: Robert C. Martin, *Agile Software Development: Principles,
Patterns, and Practices*, Prentice Hall, 2002. Chapter 10: LSP.

[^meyer1997]: Bertrand Meyer, *Object-Oriented Software Construction*,
2nd ed., Prentice Hall, 1997.

[^liskov1987]: Barbara Liskov, "Data Abstraction and Hierarchy", *SIGPLAN Notices*, 23(5), 1988. (Keynote address, OOPSLA 1987)

[^liskov1994]: Barbara H. Liskov and Jeannette M. Wing, "A Behavioral Notion of Subtyping", *ACM Transactions on Programming Languages and Systems*, 16(6), pp. 1811–1841, 1994.


[^meyer1997]: Bertrand Meyer, *Object-Oriented Software Construction*, 2nd ed., Prentice Hall, 1997. Chapter on inheritance misuse ("convenience inheritance").

[^cook1990]: William R. Cook, Walter L. Hill, and Peter S. Canning, "Inheritance Is Not Subtyping", *Proceedings of the 17th ACM SIGPLAN-SIGACT Symposium on Principles of Programming Languages (POPL)*, pp. 125–135, 1990.

[^harmse2015]: Henriette Harmse, "The Rectangle/Square Controversy", 2015. https://henrietteharmse.com/2015/04/18/the-rectanglesquare-controversy/

[^wikipedia]: "Circle–ellipse problem", *Wikipedia*. https://en.wikipedia.org/wiki/Circle%E2%80%93ellipse_problem

[^unclebob2015]: Robert C. Martin, "Interface Considered Harmful", *The Clean Coder Blog*, January 2015. https://blog.cleancoder.com/uncle-bob/2015/01/08/InterfaceConsideredHarmful.html
---

## 5. Naming Things: The `XxxBase` Convention

If abstract classes are material (Hylē), they deserve a name that reflects their role as substrate. The community has various conventions:

| Convention | Example | Problem |
|---|---|---|
| `Abstract` prefix | `AbstractWidget` | Redundant — everyone knows it's abstract |
| `Template` suffix | `WidgetTemplate` | Confused with the Template Method pattern |
| `Hyle` suffix | `WidgetHyle` | Philosophically perfect. Practically unemployable. |
| `Base` suffix | `WidgetBase` | Concise, material connotation, no false associations ✓ |

`WidgetBase` wins. It says: *this is the foundation*. The uncarved block. The marble before the sculptor arrives. Concrete classes inherit from it, shape it, and commit to their interfaces.

```java
// Java
abstract class WidgetBase {
    protected LoggerBase logger;   // aggregated material
    protected ConfigBase config;   // aggregated material
}

class ConcreteButton extends WidgetBase implements IClickable, IObservable {
    // Material: from WidgetBase
    // Telos:    IClickable, IObservable
    // Shape:    ConcreteButton
}
```

```abap
* ABAP
CLASS zcl_widget_base DEFINITION ABSTRACT.
  PROTECTED SECTION.
    DATA mo_logger TYPE REF TO zcl_logger_base.
    DATA mo_config TYPE REF TO zcl_config_base.
ENDCLASS.

CLASS zcl_concrete_button DEFINITION
  INHERITING FROM zcl_widget_base.
  PUBLIC SECTION.
    INTERFACES zif_clickable.
    INTERFACES zif_observable.
ENDCLASS.
```

---

## 6. What About Uncle Bob's Combinatorial Explosion Objection?

When I proposed the `XxxBase` pattern, Uncle Bob (in my imagination, which is where most of my best arguments happen) immediately objected: "But if you need Widget+Logger, Widget+Database, and Widget+Logger+Database, you'll need three separate base classes. Combinatorial explosion!"

This is a fair point. It is also, upon examination, equally fatal to multiple inheritance:

```cpp
// C++ — combinatorial explosion happens here too
class WidgetWithLogger      : public WidgetBase, public LoggerBase { };
class WidgetWithDB          : public WidgetBase, public DatabaseBase { };
class WidgetWithLoggerAndDB : public WidgetBase, public LoggerBase, public DatabaseBase { };
```

The explosion is a design smell regardless of mechanism. The answer is not to choose a different inheritance strategy; it is to ask why your widget needs to know about databases in the first place. (Spoiler: it probably doesn't. That's what dependency injection is for.)

Refactoring eliminates unnecessary combinations. What remains is a small, stable set of base classes that are reused freely. The `XxxBase` pattern does not cause the explosion; it merely makes it *visible*, which is the first step towards fixing it.

---

## 7. Java's Default Methods: The Serpent in the Garden

Java 8 introduced `default` methods in interfaces. Suddenly, an interface could contain implementation:

```java
// Java 8 — the telos starts doing the morphē's job
interface IObservable {
    void subscribe(Observer o);

    // A default method. Convenient. Philosophically unsettling.
    default void subscribeAll(List<Observer> observers) {
        observers.forEach(this::subscribe);
    }
}
```

From our Aristotelian perspective, this is a category error: the Final Cause is doing the Formal Cause's job. The purpose-definer is becoming the shape-giver. The god is doing the sculptor's work.

The motivation was legitimate: backward compatibility. Once an interface is a public API used by thousands of implementors worldwide, adding a new method without a default would break everything. This is not a laziness problem. It is an *ownership and versioning* problem that persists regardless of how much AI assistance you have.

Nevertheless, the philosophical verdict stands: `default` methods are a necessary evil at the platform layer, and a genuine evil at the application layer.

| Layer | Default methods | Reason |
|---|---|---|
| Language / JDK / ABAP kernel | Permitted | Backward compatibility of public APIs |
| Shared platform / framework team | Permitted for cross-cutting concerns only | Security, telemetry, tracing |
| Domain interfaces (application) | **Prohibited** | Interfaces must remain pure telos |

> **ABAP note:** ABAP interfaces do not support default implementations at all. What looks like a limitation is, from our framework, a virtue. SAP accidentally enforced philosophical purity.

---

## 8. The Verdict: Uncle Bob Was 70% Right

Let us render judgment.

**Uncle Bob's correct claims:**

- ✅ The Diamond Problem was solved long before Java was born.
- ✅ Multiple class inheritance would have enabled cleaner composition in many cases.
- ✅ Delegation boilerplate is genuinely annoying.

**Uncle Bob's overcorrection:**

- ❌ The `interface` keyword encodes a philosophically important distinction — between material succession and teleological commitment — that C++'s uniform colon syntax erases.
- ❌ Java's `extends`/`implements` and ABAP's `INHERITING FROM`/`INTERFACES` accidentally preserve this distinction at the syntactic level.
- ❌ The correct fix would have been: allow multiple class inheritance *and* keep the `interface` keyword.

> A good prosecutor does not acquit the guilty by convicting the innocent. The `interface` keyword is innocent. The prohibition of multiple class inheritance is the guilty party.

---

## 9. Summary: The Four Causes in Three Languages

| Cause | Role | Java | ABAP | C++ |
|---|---|---|---|---|
| Hylē (Material) | `XxxBase` — substrate | `abstract class XxxBase` | `CLASS ... ABSTRACT.` | abstract base class |
| Morphē (Formal) | Concrete class | `class Xxx extends XxxBase` | `INHERITING FROM` | `class Xxx : public XxxBase` |
| Kinesis (Efficient) | Constructor / Factory | `new Xxx()` / Factory | `NEW zcl_xxx( )` | `make_unique<Xxx>()` |
| Telos (Final) | Interface — purpose | `implements IXxx` | `INTERFACES zif_xxx.` | `: public IXxx` |
| Entelecheia | Value object — telos = morphē | `final` + immutable | `FINAL CREATE PRIVATE` | `const` value type |

---

## 10. Conclusion: You Are More Than a Craftsman

There is a view of programming that treats it as mere craft: you take requirements, you produce working software, you go home. Clean Code is about writing tidy craft. Design patterns are reusable craft templates. SOLID is a checklist for responsible craft.

This view is not wrong. It is just incomplete.

When you sit down to design an interface — not to implement it, but to *design* it — you enter a different mode. You are asking: what is this thing *for*? What is its essential purpose, stripped of all accidental implementation detail? You are not crafting; you are *legislating*. You are not making something; you are defining what it *means* to be something.

That is the act Aristotle called *telos*. That is what the `interface` keyword, in all its allegedly harmful simplicity, is trying to capture.

Uncle Bob was right that the Diamond Problem is solved. He was right that delegation boilerplate is tedious. But he was wrong to conclude that the interface should be abolished. The interface is not the problem. The interface is the closest thing your type system has to philosophy.

*Aristotle would have written clean interfaces. He might even have used ABAP. (He did, after all, believe in the importance of proper categories.)*

---

## Afterword: A Note on ABAP

ABAP programmers are sometimes told, with a mixture of pity and condescension, that their language is behind the times. No multiple inheritance. No generics until relatively recently. No lambda expressions until ABAP 7.4.

From the framework developed in this post, ABAP's constraints look rather different. The mandatory separation of `INHERITING FROM` (material) and `INTERFACES` (telos) is not a limitation — it is enforced philosophical discipline. The absence of default interface implementations is not backwardness — it is purity.

SAP did not read Aristotle. But they accidentally implemented him.

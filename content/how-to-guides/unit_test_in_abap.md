
+++
title = "ABAP Unit Testing: A Practical Guide with Dependency Injection"
date = 2025-03-22T00:00:00Z
draft = false
description = "A hands-on guide to ABAP Unit Testing — covering local test class syntax, assertions, dependency injection, interface casting, database/function module mocking, and the factory pattern."
tags = ["ABAP", "Unit Testing", "SAP", "TDD", "Dependency Injection", "Clean Code"]
categories = ["Development", "Testing"]
author = "Your Name"
type = "docs"
weight = 40
+++

Unit testing in ABAP is your safety net when modifying or refactoring business-critical SAP code. This guide covers everything you need to write effective, isolated tests — from local class syntax to dependency injection patterns and the factory approach.

---

## Table of Contents

1. [Local Test Class Syntax](#1-local-test-class-syntax)
2. [Test Method Syntax](#2-test-method-syntax)
3. [Dependency Injection Basics](#3-dependency-injection-basics)
4. [Interfaces vs. Classes: When to Use Which](#4-interfaces-vs-classes-when-to-use-which)
5. [Casting to an Interface](#5-casting-to-an-interface)
6. [DI Example: Injecting a Database Access Class](#6-di-example-injecting-a-database-access-class)
7. [DI Example: Injecting a Function Module Wrapper](#7-di-example-injecting-a-function-module-wrapper)
8. [The Factory Pattern and Its Benefits](#8-the-factory-pattern-and-its-benefits)

---

## 1. Local Test Class Syntax

ABAP Unit tests are written as **local classes** placed in the test include of a program or class pool. The key additions that make a class a test class are:

- `FOR TESTING` — marks the class as a test class (hidden from production runtime)
- `RISK LEVEL` — declares what side effects the tests may have
- `DURATION` — declares the expected maximum runtime

```abap
CLASS ltc_order_calculator DEFINITION
  FINAL
  FOR TESTING
  RISK LEVEL HARMLESS
  DURATION SHORT.

  PRIVATE SECTION.
    METHODS:
      test_total_with_discount FOR TESTING,
      test_total_without_discount FOR TESTING.

ENDCLASS.
```

### RISK LEVEL options

| Value | Meaning |
|-------|---------|
| `HARMLESS` | No data is changed. Read-only. Preferred for pure logic tests. |
| `DANGEROUS` | May change persistent data (DB writes, etc.). |
| `CRITICAL` | May cause serious side effects (e.g., send emails, post documents). |

Always aim for `HARMLESS`. If your test requires `DANGEROUS` or `CRITICAL`, that is a signal your production code needs better isolation.

### DURATION options

| Value | Meaning |
|-------|---------|
| `SHORT` | Expected to finish in under 1 second. Default choice. |
| `MEDIUM` | Up to 1 minute. |
| `LONG` | Up to 5 minutes. Rarely appropriate for unit tests. |

---

## 2. Test Method Syntax

Each test method must be declared with `FOR TESTING` and must be `PUBLIC` or `PRIVATE` (not `PROTECTED`). The method has no parameters and no return value — results are communicated via assertions.

```abap
CLASS ltc_order_calculator IMPLEMENTATION.

  METHOD test_total_with_discount.
    " Arrange
    DATA(lo_calculator) = NEW lcl_order_calculator( ).

    " Act
    DATA(lv_total) = lo_calculator->calculate_total(
      iv_price    = '100.00'
      iv_quantity = 3
      iv_discount = '10.00'
    ).

    " Assert
    cl_abap_unit_assert=>assert_equals(
      act = lv_total
      exp = '270.00'
      msg = 'Total with 10% discount on 3 x 100 should be 270'
    ).
  ENDMETHOD.

  METHOD test_total_without_discount.
    " Arrange
    DATA(lo_calculator) = NEW lcl_order_calculator( ).

    " Act
    DATA(lv_total) = lo_calculator->calculate_total(
      iv_price    = '50.00'
      iv_quantity = 4
      iv_discount = '0.00'
    ).

    " Assert
    cl_abap_unit_assert=>assert_equals(
      act = lv_total
      exp = '200.00'
      msg = 'Total without discount on 4 x 50 should be 200'
    ).
  ENDMETHOD.

ENDCLASS.
```

> **Arrange / Act / Assert (AAA)** is the recommended structure for each test method. It keeps tests readable and intention clear.

### Common assertion methods

```abap
" Equality
cl_abap_unit_assert=>assert_equals( act = lv_result exp = lv_expected ).

" Boolean true/false
cl_abap_unit_assert=>assert_true(  act = lv_flag ).
cl_abap_unit_assert=>assert_false( act = lv_flag ).

" Reference is initial (null check)
cl_abap_unit_assert=>assert_bound(     act = lo_object ).
cl_abap_unit_assert=>assert_not_bound( act = lo_object ).

" Expecting an exception
cl_abap_unit_assert=>assert_subrc( exp = 4 ).

" Explicit fail
cl_abap_unit_assert=>fail( msg = 'Should not have reached this line' ).
```

---

## 3. Dependency Injection Basics

**Dependency Injection (DI)** means that a class receives its collaborators from the outside, rather than creating them internally. This is the foundation of testability.

### Without DI — hard to test

```abap
CLASS lcl_order_service DEFINITION.
  PUBLIC SECTION.
    METHODS get_order_total
      IMPORTING iv_order_id TYPE vbeln
      RETURNING VALUE(rv_total) TYPE wrbtr.
ENDCLASS.

CLASS lcl_order_service IMPLEMENTATION.
  METHOD get_order_total.
    " Directly reads the database — impossible to test without real data
    SELECT SINGLE netwr INTO rv_total
      FROM vbak WHERE vbeln = iv_order_id.
  ENDMETHOD.
ENDCLASS.
```

This class is impossible to unit test without a matching entry in `VBAK`. It is also `RISK LEVEL DANGEROUS` at best.

### With DI — easily testable

```abap
CLASS lcl_order_service DEFINITION.
  PUBLIC SECTION.
    METHODS constructor
      IMPORTING io_db_reader TYPE REF TO lif_order_db_reader.
    METHODS get_order_total
      IMPORTING iv_order_id TYPE vbeln
      RETURNING VALUE(rv_total) TYPE wrbtr.
  PRIVATE SECTION.
    DATA mo_db_reader TYPE REF TO lif_order_db_reader.
ENDCLASS.

CLASS lcl_order_service IMPLEMENTATION.
  METHOD constructor.
    mo_db_reader = io_db_reader.
  ENDMETHOD.

  METHOD get_order_total.
    rv_total = mo_db_reader->read_total( iv_order_id ).
  ENDMETHOD.
ENDCLASS.
```

Now the database dependency is injected. In tests you pass a **test double**; in production you pass the real implementation.

---

## 4. Interfaces vs. Classes: When to Use Which

When injecting a dependency, you have two options: declare the dependency as a **reference to an interface** or as a **reference to a class**.

### Rule of thumb

> Prefer an **interface** as the type of an injected dependency.  
> This allows you to substitute any implementation — including test doubles — without changing the consuming class.

### Interface: method names contain the interface name

When a class implements an interface, its methods are accessed using the **interface-qualified name**:

```abap
INTERFACE lif_order_db_reader.
  METHODS read_total
    IMPORTING iv_order_id TYPE vbeln
    RETURNING VALUE(rv_total) TYPE wrbtr.
ENDINTERFACE.

CLASS lcl_real_db_reader DEFINITION.
  PUBLIC SECTION.
    INTERFACES lif_order_db_reader.
ENDCLASS.

CLASS lcl_real_db_reader IMPLEMENTATION.
  " Method name is ALWAYS interface~method when implementing an interface
  METHOD lif_order_db_reader~read_total.
    SELECT SINGLE netwr INTO rv_total
      FROM vbak WHERE vbeln = iv_order_id.
  ENDMETHOD.
ENDCLASS.
```

The tilde (`~`) notation — `lif_order_db_reader~read_total` — is the signature that a method belongs to an interface. This is how you distinguish interface methods from regular class methods at a glance.

### Class: method names contain no interface prefix

```abap
CLASS lcl_tax_calculator DEFINITION.
  PUBLIC SECTION.
    METHODS calculate_tax
      IMPORTING iv_amount TYPE wrbtr
      RETURNING VALUE(rv_tax) TYPE wrbtr.
ENDCLASS.

CLASS lcl_tax_calculator IMPLEMENTATION.
  " Plain method name — no interface prefix
  METHOD calculate_tax.
    rv_tax = iv_amount * '0.10'.
  ENDMETHOD.
ENDCLASS.
```

When to use a class reference instead of an interface:

- The dependency is a **stable utility with no variation** (e.g., a pure math helper)
- You are injecting a **superclass** and relying on polymorphism (less common in ABAP)

For anything that touches I/O, the database, external systems, or system time, **always define and inject via an interface**.

---

## 5. Casting to an Interface

When you hold a concrete class instance but need to pass it where an interface reference is expected, you use **CAST** (or the older `?=` operator):

```abap
DATA lo_real_reader  TYPE REF TO lcl_real_db_reader.
DATA lo_if_reader    TYPE REF TO lif_order_db_reader.

lo_real_reader = NEW lcl_real_db_reader( ).

" Upcast: always safe because lcl_real_db_reader implements lif_order_db_reader
lo_if_reader = lo_real_reader.   " implicit upcast — no syntax needed

" Alternatively, explicit cast:
lo_if_reader = CAST lif_order_db_reader( lo_real_reader ).
```

In test code you typically do the opposite — you create a **test double class** that also implements the interface, and pass it in:

```abap
DATA lo_test_double  TYPE REF TO ltc_mock_db_reader.   " local test double
DATA lo_if_reader    TYPE REF TO lif_order_db_reader.

lo_test_double = NEW ltc_mock_db_reader( ).
lo_if_reader   = lo_test_double.   " upcast: safe because mock implements the interface

DATA(lo_service) = NEW lcl_order_service( lo_if_reader ).
```

> **Downcast** (interface → concrete class) requires `CAST` with a `CATCH CX_SY_MOVE_CAST_ERROR` guard. Avoid this in production code; it defeats polymorphism.

---

## 6. DI Example: Injecting a Database Access Class

This is the most common use case. We wrap all DB reads in a dedicated class behind an interface.

### Interface + real implementation

```abap
"------------------------------------------------------------
" Interface
"------------------------------------------------------------
INTERFACE lif_order_db_reader.
  METHODS read_order_header
    IMPORTING iv_vbeln         TYPE vbeln
    RETURNING VALUE(rs_header) TYPE vbak.
ENDINTERFACE.

"------------------------------------------------------------
" Real implementation (production code)
"------------------------------------------------------------
CLASS lcl_order_db_reader DEFINITION.
  PUBLIC SECTION.
    INTERFACES lif_order_db_reader.
ENDCLASS.

CLASS lcl_order_db_reader IMPLEMENTATION.
  METHOD lif_order_db_reader~read_order_header.
    SELECT SINGLE * FROM vbak
      INTO rs_header
      WHERE vbeln = iv_vbeln.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------
" Service that consumes the interface
"------------------------------------------------------------
CLASS lcl_order_service DEFINITION.
  PUBLIC SECTION.
    METHODS constructor
      IMPORTING io_db TYPE REF TO lif_order_db_reader.
    METHODS is_high_value_order
      IMPORTING iv_vbeln        TYPE vbeln
      RETURNING VALUE(rv_result) TYPE abap_bool.
  PRIVATE SECTION.
    DATA mo_db TYPE REF TO lif_order_db_reader.
ENDCLASS.

CLASS lcl_order_service IMPLEMENTATION.
  METHOD constructor.
    mo_db = io_db.
  ENDMETHOD.

  METHOD is_high_value_order.
    DATA(ls_header) = mo_db->read_order_header( iv_vbeln ).
    rv_result = xsdbool( ls_header-netwr > 100000 ).
  ENDMETHOD.
ENDCLASS.
```

### Test double + test class

```abap
"------------------------------------------------------------
" Test double (lives only in test include)
"------------------------------------------------------------
CLASS ltc_mock_db_reader DEFINITION FOR TESTING.
  PUBLIC SECTION.
    INTERFACES lif_order_db_reader.
    DATA ms_order TYPE vbak.          " test data injected by the test method
ENDCLASS.

CLASS ltc_mock_db_reader IMPLEMENTATION.
  METHOD lif_order_db_reader~read_order_header.
    " Return whatever the test set up — no DB access at all
    rs_header = ms_order.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------
" Test class
"------------------------------------------------------------
CLASS ltc_order_service DEFINITION FINAL FOR TESTING
  RISK LEVEL HARMLESS
  DURATION SHORT.

  PRIVATE SECTION.
    METHODS test_high_value_order     FOR TESTING.
    METHODS test_not_high_value_order FOR TESTING.
ENDCLASS.

CLASS ltc_order_service IMPLEMENTATION.

  METHOD test_high_value_order.
    " Arrange
    DATA(lo_mock) = NEW ltc_mock_db_reader( ).
    lo_mock->ms_order = VALUE vbak( vbeln = '0000000001' netwr = '200000.00' ).

    DATA(lo_service) = NEW lcl_order_service( lo_mock ).

    " Act
    DATA(lv_result) = lo_service->is_high_value_order( '0000000001' ).

    " Assert
    cl_abap_unit_assert=>assert_true(
      act = lv_result
      msg = 'Order with NETWR > 100000 should be high value'
    ).
  ENDMETHOD.

  METHOD test_not_high_value_order.
    " Arrange
    DATA(lo_mock) = NEW ltc_mock_db_reader( ).
    lo_mock->ms_order = VALUE vbak( vbeln = '0000000002' netwr = '500.00' ).

    DATA(lo_service) = NEW lcl_order_service( lo_mock ).

    " Act
    DATA(lv_result) = lo_service->is_high_value_order( '0000000002' ).

    " Assert
    cl_abap_unit_assert=>assert_false(
      act = lv_result
      msg = 'Order with NETWR <= 100000 should not be high value'
    ).
  ENDMETHOD.

ENDCLASS.
```

Both tests run with `RISK LEVEL HARMLESS` and complete in milliseconds — no SAP system data required.

---

## 7. DI Example: Injecting a Function Module Wrapper

Legacy ABAP systems rely heavily on Function Modules (FMs). You cannot mock an FM directly, but you can wrap it in a class and inject the wrapper.

### Interface + real wrapper

```abap
"------------------------------------------------------------
" Interface
"------------------------------------------------------------
INTERFACE lif_conversion_exit.
  METHODS convert_to_external
    IMPORTING iv_internal        TYPE matnr
    RETURNING VALUE(rv_external) TYPE matnr_ext.
ENDINTERFACE.

"------------------------------------------------------------
" Real wrapper — calls the actual function module
"------------------------------------------------------------
CLASS lcl_conversion_exit DEFINITION.
  PUBLIC SECTION.
    INTERFACES lif_conversion_exit.
ENDCLASS.

CLASS lcl_conversion_exit IMPLEMENTATION.
  METHOD lif_conversion_exit~convert_to_external.
    CALL FUNCTION 'CONVERSION_EXIT_MATN1_OUTPUT'
      EXPORTING input  = iv_internal
      IMPORTING output = rv_external.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------
" Service that uses the wrapper
"------------------------------------------------------------
CLASS lcl_material_service DEFINITION.
  PUBLIC SECTION.
    METHODS constructor
      IMPORTING io_conv TYPE REF TO lif_conversion_exit.
    METHODS get_display_number
      IMPORTING iv_matnr          TYPE matnr
      RETURNING VALUE(rv_display) TYPE matnr_ext.
  PRIVATE SECTION.
    DATA mo_conv TYPE REF TO lif_conversion_exit.
ENDCLASS.

CLASS lcl_material_service IMPLEMENTATION.
  METHOD constructor.
    mo_conv = io_conv.
  ENDMETHOD.

  METHOD get_display_number.
    rv_display = mo_conv->convert_to_external( iv_matnr ).
  ENDMETHOD.
ENDCLASS.
```

### Test double + test class

```abap
"------------------------------------------------------------
" Test double — returns a predictable value without calling the FM
"------------------------------------------------------------
CLASS ltc_mock_conversion_exit DEFINITION FOR TESTING.
  PUBLIC SECTION.
    INTERFACES lif_conversion_exit.
ENDCLASS.

CLASS ltc_mock_conversion_exit IMPLEMENTATION.
  METHOD lif_conversion_exit~convert_to_external.
    " Strip leading zeros — simple simulation of the real FM behaviour
    rv_external = iv_internal.
    SHIFT rv_external LEFT DELETING LEADING '0'.
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------
" Test class
"------------------------------------------------------------
CLASS ltc_material_service DEFINITION FINAL FOR TESTING
  RISK LEVEL HARMLESS
  DURATION SHORT.

  PRIVATE SECTION.
    METHODS test_leading_zeros_stripped FOR TESTING.
ENDCLASS.

CLASS ltc_material_service IMPLEMENTATION.

  METHOD test_leading_zeros_stripped.
    " Arrange
    DATA(lo_mock)    = NEW ltc_mock_conversion_exit( ).
    DATA(lo_service) = NEW lcl_material_service( lo_mock ).

    " Act
    DATA(lv_display) = lo_service->get_display_number( '000000000000TG50' ).

    " Assert
    cl_abap_unit_assert=>assert_equals(
      act = lv_display
      exp = 'TG50'
      msg = 'Leading zeros should be stripped from material number'
    ).
  ENDMETHOD.

ENDCLASS.
```

> **Tip:** This pattern applies equally to BAPI calls, RFC calls, and any other FM-based integration. Wrap first, inject second, test freely.

---

## 8. The Factory Pattern and Its Benefits

As your application grows, constructing objects with all their dependencies wired together becomes messy. The **factory pattern** centralises object creation.

### Without a factory — caller must know everything

```abap
" Somewhere in your report or program:
DATA(lo_db)      = NEW lcl_order_db_reader( ).
DATA(lo_conv)    = NEW lcl_conversion_exit( ).
DATA(lo_service) = NEW lcl_order_service( lo_db ).
DATA(lo_material)= NEW lcl_material_service( lo_conv ).
```

Every caller must know which concrete class to use and in what order to wire them. This is fragile and violates the **Single Responsibility Principle**.

### With a factory

```abap
"------------------------------------------------------------
" Factory interface
"------------------------------------------------------------
INTERFACE lif_service_factory.
  METHODS create_order_service
    RETURNING VALUE(ro_service) TYPE REF TO lcl_order_service.
  METHODS create_material_service
    RETURNING VALUE(ro_service) TYPE REF TO lcl_material_service.
ENDINTERFACE.

"------------------------------------------------------------
" Production factory — wires real implementations
"------------------------------------------------------------
CLASS lcl_service_factory DEFINITION.
  PUBLIC SECTION.
    INTERFACES lif_service_factory.
ENDCLASS.

CLASS lcl_service_factory IMPLEMENTATION.
  METHOD lif_service_factory~create_order_service.
    ro_service = NEW lcl_order_service( NEW lcl_order_db_reader( ) ).
  ENDMETHOD.

  METHOD lif_service_factory~create_material_service.
    ro_service = NEW lcl_material_service( NEW lcl_conversion_exit( ) ).
  ENDMETHOD.
ENDCLASS.

"------------------------------------------------------------
" Test factory — wires test doubles instead
"------------------------------------------------------------
CLASS ltc_test_factory DEFINITION FOR TESTING.
  PUBLIC SECTION.
    INTERFACES lif_service_factory.
ENDCLASS.

CLASS ltc_test_factory IMPLEMENTATION.
  METHOD lif_service_factory~create_order_service.
    DATA(lo_mock) = NEW ltc_mock_db_reader( ).
    lo_mock->ms_order = VALUE vbak( netwr = '999999.00' ).
    ro_service = NEW lcl_order_service( lo_mock ).
  ENDMETHOD.

  METHOD lif_service_factory~create_material_service.
    ro_service = NEW lcl_material_service( NEW ltc_mock_conversion_exit( ) ).
  ENDMETHOD.
ENDCLASS.
```

### A consumer that accepts a factory

```abap
CLASS lcl_report_controller DEFINITION.
  PUBLIC SECTION.
    METHODS constructor
      IMPORTING io_factory TYPE REF TO lif_service_factory.
    METHODS run
      IMPORTING iv_vbeln TYPE vbeln.
  PRIVATE SECTION.
    DATA mo_factory TYPE REF TO lif_service_factory.
ENDCLASS.

CLASS lcl_report_controller IMPLEMENTATION.
  METHOD constructor.
    mo_factory = io_factory.
  ENDMETHOD.

  METHOD run.
    DATA(lo_order_svc) = mo_factory->create_order_service( ).
    IF lo_order_svc->is_high_value_order( iv_vbeln ) = abap_true.
      " ... handle high value order
    ENDIF.
  ENDMETHOD.
ENDCLASS.
```

### Benefits of the factory pattern

| Benefit | Why it matters |
|---------|----------------|
| **Single wiring point** | All dependency wiring lives in one place. Changing a concrete class means editing the factory only. |
| **Swap implementations easily** | Pass `ltc_test_factory` in tests, `lcl_service_factory` in production. The controller never knows the difference. |
| **Encapsulates construction complexity** | Multi-step object graphs are hidden from callers. |
| **Supports Open/Closed Principle** | Add a new service by extending the factory, not by modifying callers. |
| **Enables integration-level test doubles** | A test factory can wire a mix of real and mock dependencies for integration tests. |

---

## Summary

| Concept | Key takeaway |
|---------|-------------|
| `FOR TESTING` | Marks a local class as test-only; excluded from production runtime |
| `RISK LEVEL HARMLESS` | Aim for this; forces you to eliminate I/O from unit tests |
| Test method (`FOR TESTING`) | No parameters, no return value; uses `CL_ABAP_UNIT_ASSERT` for results |
| Interface as dependency type | Enables substitution with test doubles; always prefer over class references for I/O |
| `interface~method` naming | Identifies interface-implemented methods in ABAP source |
| Upcast to interface | Implicit; downcast requires `CAST` and error handling |
| DB wrapper pattern | Wrap `SELECT` in a class behind an interface; inject and mock in tests |
| FM wrapper pattern | Wrap `CALL FUNCTION` in a class behind an interface; same approach as DB |
| Factory pattern | Centralises wiring; swap production vs. test dependencies at a single point |

With these techniques in place, your ABAP code becomes significantly easier to test, maintain, and evolve — even in large, legacy SAP landscapes.


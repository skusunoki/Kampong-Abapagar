+++
date = '2025-05-11T20:30:53+08:00'
draft = false
title = 'Limitations of Legacy ABAP'

weight = 20
type = "docs"
+++

## Definition

The term **Legacy ABAP** is ambiguous. In this site, Legacy ABAP refers to obsolete language features, ABAP programmer practices that are no longer necessary in modern tool sets, and architectures that have been adopted as a necessary evil. 

## Obsolete language features

The ABAP language is also changing with the times, and older language features are becoming less compatible.

- Obsolete technics to make complex programs to separate parts (Modularization) like subroutines, and function modules
- Obsolete calls like `CALL TRANSACTION`, `CALL METHOD`, and `CALL DIALOG`
- Internal table definition with `WITH HEADER LINE`

## Old style ABAP programmer practices

Some of the difficulty in SAP projects comes from the persistence of practices that should have been eliminated in other IT projects.

- Old-style function modules instead of object oriented development objects like classes and interfaces
- No unit test code
- Many intermediate variables
- Commented out dead code

## Tight coupling architecture patterns 

The code style is straightforward, prioritizing minimal effort under the tight coupling architecture. However, scale of SAP projects is not small. It's rare for things to end so simply. Priotize the code must be thoroughly tested. 

- Highly dependent on database table definition
- Direct database tables access with `SELECT` queries
- Direct use of `SY-UNAME` and `SY-DATUM`
- Procedural design and programming style instead of domain model style




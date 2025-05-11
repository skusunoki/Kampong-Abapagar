+++
date = '2025-05-11T20:30:53+08:00'
draft = false
title = 'Limitations of Legacy ABAP'

weight = 20
type = "docs"
+++

## Definition

The term **Legacy ABAP** is ambiguous. In this site, Legacy ABAP refers to obsolete language features, ABAP programmer practices that are no longer necessary in modern toolsets, and entire architectures that have been adopted as a necessary evil. 

## Obsolete language features

The ABAP language is also changing with the times, and older language features are becoming less compatible.

- Obsolete modularization technics like subroutines, and function modules
- Obsolete calls like `CALL TRANSACTION`, `CALL METHOD`, and `CALL DIALOG`
- Internal table definition with `WITH HEADER LINE`

## Old style ABAP programmer practices

Some of the difficulty in SAP projects comes from the persistence of practices that should have been eliminated in other IT projects.

- Old-style function modules instead of object oriented development objects like classes and interfaces
- No unit test code
- Many intermediate variables
- Commented out dead code

## Tight coupling architecture patterns 

The writing style is straightforward, prioritizing minimal effort, but the scale of SAP projects is not small. 

- Highly dependent on database table definition and sytem variables like `SY-UNAME` and `SY-DATUM`
- Direct database tables access with `SELECT` queries
- Procedural design and programming style instead of domain model style



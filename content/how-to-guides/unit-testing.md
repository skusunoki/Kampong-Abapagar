+++
date = '2025-05-20T23:02:03+08:00'
draft = true
title = 'How to Start Unit Testing'
type = "docs"
+++

## 1st step to do

First, create a test class.

```abap
CLASS lycl_fizzbuzz DEFINITION FOR TESTING RISK LEVEL HARMLESS DURATION SHORT.
ENDCLASS.
```

This defines a local test class for unit testing. The `FOR TESTING` addition marks it as a test class. `RISK LEVEL HARMLESS` indicates the test won't modify production data, and `DURATION SHORT` means the test should complete quickly (under 60 seconds).

```abap
  PUBLIC SECTION.
    METHODS t01_1_to_10 FOR TESTING.
```

This declares a test method within the test class. The `FOR TESTING` addition identifies this method as an actual test case that will be executed by the ABAP Unit framework. The method name `t01_1_to_10` suggests it's testing numbers 1 to 10.

```abap
CLASS lycl_fizzbuzz IMPLEMENTATION.
  METHOD t01_1_to_10.
    FINAL(lo_fizzbuzz) = NEW zcl_fizzbuzz( ).
    FINAL(lt_fizzbuzz_act) = lo_fizzbuzz( )->generate_fizzbuzz( 10 ).
    FINAL(lt_fizzbuzz_exp) = VALUE #( ( number = 1 result = '' )
                                      ( number = 2 result = '' )
                                      ( number = 3 result = 'fizz' )
                                      ( number = 4 result = '' )
                                      ( number = 5 result = 'buzz' )
                                      ( number = 6 result = 'fizz' )
                                      ( number = 7 result = '' )
                                      ( number = 8 result = '' )
                                      ( number = 9 result = 'fizz' )
                                      ( number = 10 result = 'buzz' ) ).
    cl_abap_unit=>assert_equals( exp = lt_fizzbuzz_act
                                 act = lt_fizzbuzz_exp ).
  ENDMETHOD.
ENDCLASS.
```

This is the implementation of the test method. It follows the typical "Arrange-Act-Assert" pattern:

- **Arrange**: Creates an instance of the `ZCL_FIZZBUZZ` class
- **Act**: Calls the `generate_fizzbuzz( 10 )` method to get actual results
- **Assert**: Defines expected results using the VALUE constructor and compares them with actual results using `cl_abap_unit_assert=>assert_equals()`

The test validates the FizzBuzz logic where multiples of 3 return 'fizz', multiples of 5 return 'buzz', and other numbers return empty strings.


+++
date = '2025-05-20T23:02:03+08:00'
draft = true
title = 'How to Start Unit Testing'
type = "docs"
+++

## 1st step to do

```abap
class lycl_fizzbuzz definition for testing risk level harmless duration short.
endclass.
```

```abap
    methods t01_1_to_10 for testing.
```

```abap
class lycl_fizzbuzz implementation.
    methods t01_1_to_10.
        data(lo_fizzbuzz) = new zcl_fizzbuzz( ).
        data(lt_fizzbuzz_act) = lo_fizzbuzz( )->generate_fizzbuzz( 10 ).
        data(lt_fizzbuzz_exp) = value #( ( number = 1 result = '' )
                                         ( number = 2 result = '' )
                                         ( number = 3 result = 'fizz' )
                                         ( number = 4 result = '' )
                                         ( number = 5 result = 'buzz' )
                                         ( number = 6 result = 'fizz' )
                                         ( number = 7 result = '' ) 
                                         ( numebr = 8 result = '' )
                                         ( number = 9 result = 'fizz' )
                                         ( number = 10 result = 'buzz' )
        ).
        cl_abap_unit=>assert_equals( exp = lt_fizzbuzz_act 
                                     act = lt_fizzbuzz_exp ).

    endmethod.
endclass.
```


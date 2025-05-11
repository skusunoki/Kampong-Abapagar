+++
date = '2025-05-11T11:10:31+08:00'
draft = true
title = 'How to Calculate Sum in Multiple Currencies'
weight = 30
type = "docs"
+++

## Use `VALUE` with `FOR GROUPS k of r IN itab` and `REDUCE` with `IN GROUP k` syntax

If the internal table contains multiple currencies, the program should calculate the sums for each currency. `VALUE` operator is expected to generate new internal table with the currency as the key. `REDUCE` operator is expected to generate sum of each currency.

### Prerequisite

There are two structures and two table types required. The first pair of structure and table is for original data. The second pair is for sum. 

```abap
TYPES: BEGIN OF purchase_order_item,
    purchase_order         TYPE ebeln,
    purchase_order_item    TYPE ebelp,
    item_amount            TYPE netwr,
    po_currency            TYPE curr,
END OF purchase_order_item.

TYPES: BEGIN OF sum_of_purchase_order_item,
    item_amount            TYPE netwr,
    po_currency            TYPE curr,
END OF purchase_order_item.

TYPE t_purchase_order_item TYPE STANDARD TABLE OF purchase_order_item WITH EMPTY KEY.
TYPE t_sum_of_purchase_order_item TYPE STANDARD TABLE OF purchase_order_item WITH EMPTY KEY.
```
The internal table `l_po_items` has following records. It seems that there are multiple currencies of purchase order amount. 

```abap
DATA(l_po_items) = VALUE t_purchase_order_item(
    ( purchase_order = '4500000100' purchase_order_item = '00010' item_amount = '100.00' po_currency = 'USD' )
    ( purchase_order = '4500000200' purchase_order_item = '00011' item_amount = '200.00' po_currency = 'SGD' )
    ( purchase_order = '4500000300' purchase_order_item = '00012' item_amount = '300.00' po_currency = 'SGD' )
    ( purchase_order = '4500000400' purchase_order_item = '00013' item_amount = '400.00' po_currency = 'USD' )
).
```

### Body of process

Now that the internal table of sum can be generated from original internal table. `po_currency` is interim variable to hold purchase order currency as group key. `r`, `s`, and `m` stand for Record, Sum, and group Member for each.  

```abap
DATA(l_sum_of_po_item_amount) = VALUE t_sum_of_purchase_order_item(
    FOR GROUPS po_currency OF r IN l_po_items GROUP BY r-po_currency ASCENDING
    ( item_amount = REDUCE bwert( INIT s TYPE bwert FOR m IN GROUP po_currency NEXT s = s + m-item_amount )
      po_currency = po_currency ) ).       
```

### Expected result

```abap
cl_abapunit_assert=>assert_equals(
    act = l_sum_of_po_item_amount
    exp = VALUE t_sum_of_po_item_amount( 
        ( item_amount = '500.00' po_currency = 'SGD'
          item_amount = '500.00' po_currency = 'USD' )
    )
).
```

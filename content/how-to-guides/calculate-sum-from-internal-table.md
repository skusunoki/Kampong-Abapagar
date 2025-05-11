+++
date = '2025-05-11T09:44:32+08:00'
draft = true
title = 'How to Calculate Sum From Internal Table'

weight = 20
type = "docs"
+++

## Use `REDUCE` with `FOR r IN itab` syntax

Awesome constructor operator REDUCE can remove loop and clarify the intent of your code to calculate sum from internal table. 

### Prerequisite

It's typical that `TYPE` statement defines structure and internal table with multiple fields including key, characteristics, and figures to clarify the intent of business data. In this case, there are the internal table of purchase order item with amount required. Prefix `t_` of table type  `t_purchase_order_item` is commonly used for table type[^1]. 

```abap
TYPES: BEGIN OF purchase_order_item,
    purchase_order         TYPE ebeln,
    purchase_order_item    TYPE ebelp,
    item_amount            TYPE bwert,
    po_currency            TYPE waers,
END OF purchase_order_item.

TYPE t_purchase_order_item TYPE STANDARD TABLE OF purchase_order_item WITH EMPTY KEY.
```

The internal table `l_po_items` has following records. It seems that Currency of purchase order amount should be USD[^2]. 

```abap
FINAL(l_po_items) = VALUE t_purchase_order_item(
    ( purchase_order = '4500000100' purchase_order_item = '00010' item_amount = '100.00' po_currency = 'USD' )
    ( purchase_order = '4500000100' purchase_order_item = '00020' item_amount = '200.00' po_currency = 'USD' )
    ( purchase_order = '4500000100' purchase_order_item = '00030' item_amount = '300.00' po_currency = 'USD' )
    ( purchase_order = '4500000100' purchase_order_item = '00040' item_amount = '400.00' po_currency = 'USD' )
).
```

### Body of process

Now that sum of internal table can be generated. `s`, and `r` are abbreviation of sum and record for each.

```abap
FINAL(l_sum_of_po_item_amount) = REDUCE bwert( INIT s TYPE bwert FOR r IN l_po_items NEXT s = s + r-item_amount ).
```

### Expected result

```abap
cl_abap_unit_assert=>assert_equals(
    act = l_sum_of_po_item_amount
    exp = '1000.00'
).
```

[^1]: Using plurals isn't mainstream in ABAP[^3] projects.
[^2]: If you want know how to calculate sum of amount in multiple currencies. See [How to Calculate Sum in Multiple Currencies](../calculate-sum-in-multiple-currencies)
[^3]: ABAP: Advanced Business Application Programming
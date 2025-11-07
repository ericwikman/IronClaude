---
layout: doc
outline: deep
lang: en-US
---

# ISNUMBER function

## Overview
ISNUMBER is a function of the Information category that determines whether a value is a number. The function evaluates the provided value and returns TRUE if the result is numeric, or FALSE if it is any other data type.

## Usage

### Syntax
**ISNUMBER(<span title="Any" style="color:#1E88E5">value</span>) => <span title="Boolean" style="color:#43A047">is_number</span>**

### Argument descriptions
* *value* (any type, required). The value to test. This can be a direct value, a cell reference, a formula, or any expression that evaluates to a result.

### Additional guidance
* ISNUMBER evaluates its argument before checking its type. If a cell reference or formula is provided, the function tests the result of the evaluation, not the cell content itself.
* Numbers with formatting (such as currency symbols or percentage signs) are still recognized as numbers by ISNUMBER, as the underlying value is numeric.
* Text that appears numeric (such as "123" entered as text with a quote prefix) is not considered a number and will return FALSE.

### Returned value
ISNUMBER returns a [Boolean](/features/value-types#booleans) value: TRUE if the evaluated *value* argument is a [number](/features/value-types#numbers), or FALSE if it is any other data type (string, Boolean, error, blank cell, etc.).

### Error conditions
* In common with many other IronCalc functions, ISNUMBER propagates errors that are found in its argument.
* If no argument, or more than one argument, is supplied, then ISNUMBER returns the [`#ERROR!`](/features/error-types.md#error) error.

## Examples

**Example 1: Testing a direct numeric value.**
Testing whether a literal number is numeric.
`=ISNUMBER(42)`
*Result: TRUE (the value 42 is a number)*

**Example 2: Testing a cell reference containing text.**
Testing whether a cell containing text returns a numeric type.
`=ISNUMBER(A1)` where A1 contains "Hello"
*Result: FALSE (the text "Hello" is not a number)*

**Example 3: Testing a cell with currency formatting.**
Testing whether a currency-formatted value is recognized as a number.
`=ISNUMBER(A1)` where A1 contains $100.35
*Result: TRUE (currency-formatted values have an underlying numeric value)*

**Example 4: Testing a formula result.**
Testing whether the result of a calculation is numeric.
`=ISNUMBER(SUM(A1:A10))`
*Result: TRUE (the SUM function returns a numeric value)*

**Example 5: Testing text that looks like a number.**
Testing whether text entered with a quote prefix is recognized as a number.
`=ISNUMBER(A1)` where A1 contains '13 (entered as text)
*Result: FALSE (text with a quote prefix is stored as text, not a number)*

[See some examples in IronCalc](https://app.ironcalc.com/?example=isnumber).

## Links
* See also IronCalc's [ISTEXT](/functions/information/istext.md), [ISBLANK](/functions/information/isblank.md), [ISLOGICAL](/functions/information/islogical.md), and [ISERROR](/functions/information/iserror.md) functions for testing other data types.
* Visit Microsoft Excel's [ISNUMBER function](https://support.microsoft.com/en-us/office/isnumber-function-0f2d7971-6019-40a0-a171-f2d869135665) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093296) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/ISNUMBER) provide versions of the ISNUMBER function.

---
layout: doc
outline: deep
lang: en-US
---

# COLUMN function

## Overview
COLUMN is a function of the Lookup and Reference category that returns the column number of a specified cell or range reference.

COLUMN can be used with or without an argument. When called without an argument, it returns the column number of the cell containing the formula. When called with a reference argument, it returns the column number of the first (leftmost) cell in that reference.

## Usage

### Syntax
**COLUMN([<span title="Reference" style="color:#E91E63">reference</span>]) => <span title="Number" style="color:#1E88E5">column_number</span>**

### Argument descriptions
* *reference* ([reference](/features/value-types#references), [optional](/features/optional-arguments.md)). A reference to a cell or range. If omitted, COLUMN returns the column number of the cell containing the formula. If a range is provided, COLUMN returns the column number of the leftmost cell in the range.

### Additional guidance
* When COLUMN is used without an argument, it returns the column number of the current cell, which is useful for creating dynamic formulas that respond to their location.
* Column numbers are 1-indexed, meaning column A is 1, column B is 2, and so on.
* When referencing a range with multiple columns, only the leftmost column number is returned. Use the [COLUMNS](/functions/lookup_and_reference/columns.md) function to count the number of columns in a range.

### Returned value
COLUMN returns a [number](/features/value-types#numbers) representing the column number of the referenced cell or the leftmost cell in the referenced range.

### Error conditions
* In common with many other IronCalc functions, COLUMN propagates errors that are found in its argument.
* If more than one argument is supplied, then COLUMN returns the [`#ERROR!`](/features/error-types.md#error) error.
* If the *reference* argument contains an invalid reference, then COLUMN returns the [`#VALUE!`](/features/error-types.md#value) error.

## Details
COLUMN is a lightweight function designed to extract positional information from cell references. It complements the [ROW](/functions/lookup_and_reference/row.md) function, which returns row numbers. Both functions are frequently used in combination with other functions like [INDEX](/functions/lookup_and_reference/index.md) to construct dynamic formulas.

## Examples

**Example 1: Get the column number of the current cell**
When placed in cell C5, this formula returns the column number of that cell.
`=COLUMN()`
*Result: 3 (because C is the 3rd column)*

**Example 2: Get the column number of a referenced cell**
This formula returns the column number of the cell B1, regardless of where the formula is placed.
`=COLUMN(B1)`
*Result: 2 (because B is the 2nd column)*

**Example 3: Get the column number of a range reference**
When a range is provided, COLUMN returns the column number of the leftmost cell in the range.
`=COLUMN(D10:F15)`
*Result: 4 (because D is the 4th column, even though the range extends to column F)*

[See some examples in IronCalc](https://app.ironcalc.com/?example=column).

## Links
* See also IronCalc's [ROW](/functions/lookup_and_reference/row.md), [COLUMNS](/functions/lookup_and_reference/columns.md), and [INDEX](/functions/lookup_and_reference/index.md) functions.
* Visit Microsoft Excel's [COLUMN function](https://support.microsoft.com/en-us/office/column-function-e9ea4a90-b0b0-45d8-b620-81f8e3e5a3c6) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093352) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/COLUMN) provide versions of the COLUMN function.
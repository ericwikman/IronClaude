---
layout: doc
outline: deep
lang: en-US
---

# VLOOKUP function

## Overview
VLOOKUP is a function of the Lookup and Reference category that searches for a value in the first column of a table array and returns a value in the same row from a specified column. The "V" in VLOOKUP stands for "vertical," indicating that the function searches vertically down the first column of the table.

VLOOKUP is one of the most widely used functions for data retrieval and is particularly useful when working with large datasets, performing data analysis, or creating dynamic reports that need to pull information from structured tables.

## Usage

### Syntax
**VLOOKUP(<span title="Any" style="color:#1E88E5">lookup_value</span>, <span title="Range" style="color:#E91E63">table_array</span>, <span title="Number" style="color:#1E88E5">column_index</span>, [<span title="Boolean" style="color:#43A047">is_sorted</span>=TRUE]) => <span title="Any" style="color:#1E88E5">result</span>**

### Argument descriptions
* *lookup_value* ([any type](/features/value-types), required). The value to search for in the first column of the *table_array*. This can be a number, text string, boolean value, or reference to a cell containing such a value.
* *table_array* ([range](/features/value-types#ranges), required). The range of cells that contains the data. VLOOKUP searches for the *lookup_value* in the first (leftmost) column of this range and returns a value from the same row in the specified column.
* *column_index* ([number](/features/value-types#numbers), required). The column number in the *table_array* from which to return a value. The first column (the search column) is column 1. This value is rounded down to the nearest integer if a non-integer is provided.
* *is_sorted* ([boolean](/features/value-types#booleans), [optional](/features/optional-arguments.md)). A logical value that specifies whether the first column of the *table_array* is sorted in ascending order. If TRUE or omitted (default), VLOOKUP performs an approximate match using binary search. If FALSE, VLOOKUP performs an exact match using linear search and supports wildcard pattern matching for text values.

### Additional guidance
* When *is_sorted* is TRUE (the default), the first column of *table_array* should be sorted in ascending order for accurate results. In this mode, VLOOKUP finds the largest value that is less than or equal to the *lookup_value* if an exact match is not found.
* When *is_sorted* is FALSE, VLOOKUP searches for an exact match only. For text values in unsorted mode, you can use wildcard characters in the *lookup_value*: `*` (asterisk) matches any sequence of characters, and `?` (question mark) matches any single character. To search for a literal `*` or `?`, precede it with a tilde (`~`).
* String comparisons are case-insensitive. For example, "apple" will match "APPLE" or "Apple".
* The *column_index* must be greater than 0 and cannot exceed the number of columns in the *table_array*, otherwise VLOOKUP returns a [`#REF!`](/features/error-types.md#ref) error.
* Empty cells in the search column are treated as empty strings when comparing with text values, or as zero when comparing with numeric values.

### Returned value
VLOOKUP returns the value from the cell located at the intersection of the matched row and the column specified by *column_index*. The returned value can be of any [data type](/features/value-types) (number, text, boolean, or error) depending on what is stored in that cell.

### Error conditions
* In common with many other IronCalc functions, VLOOKUP propagates errors that are found in its arguments.
* If fewer than 3 arguments or more than 4 arguments are supplied, then VLOOKUP returns the [`#ERROR!`](/features/error-types.md#error) error.
* If the value of the *column_index* argument is not (or cannot be converted to) a [number](/features/value-types#numbers), then VLOOKUP returns the [`#VALUE!`](/features/error-types.md#value) error.
* If the value of the *is_sorted* argument is not (or cannot be converted to) a [boolean](/features/value-types#booleans), then VLOOKUP returns the [`#VALUE!`](/features/error-types.md#value) error.
* If the *table_array* argument is not a valid range, then VLOOKUP returns the [`#VALUE!`](/features/error-types.md#value) or [`#NA`](/features/error-types.md#na) error.
* If the *column_index* is less than 1 or greater than the number of columns in the *table_array*, then VLOOKUP returns the [`#REF!`](/features/error-types.md#ref) error.
* If the *lookup_value* is not found in the first column of the *table_array*, then VLOOKUP returns the [`#NA`](/features/error-types.md#na) error. In sorted mode, this occurs when the *lookup_value* is smaller than the smallest value in the first column.

## Examples

**Example 1: Basic lookup with sorted data (default behavior)**

Suppose you have a product price list where column A contains product IDs sorted in ascending order, and column B contains prices. To find the price of product 105:

`=VLOOKUP(105, A1:B10, 2)`

*Result: Returns the price from column 2 of the row where product ID 105 is found in column 1. Since is_sorted is omitted, it defaults to TRUE and uses binary search.*

**Example 2: Exact match lookup with unsorted data**

If your data is not sorted and you need an exact match, set the fourth parameter to FALSE:

`=VLOOKUP("banana", A1:C20, 3, FALSE)`

*Result: Searches for an exact match of "banana" in the first column and returns the corresponding value from the third column. The search is case-insensitive, so "BANANA" or "Banana" would also match.*

**Example 3: Using wildcards in unsorted mode**

When searching with wildcards in unsorted mode:

`=VLOOKUP("app*", A1:D15, 2, FALSE)`

*Result: Finds the first cell in column A that starts with "app" (such as "apple", "application", etc.) and returns the value from column 2 of that row.*

**Example 4: Approximate match with sorted numeric data**

For a tax bracket lookup where income thresholds are sorted in column A:

`=VLOOKUP(45000, A1:B10, 2, TRUE)`

*Result: If 45000 is not found, returns the tax rate corresponding to the largest threshold less than or equal to 45000. For example, if the thresholds are 0, 20000, 40000, 60000, it would match the 40000 row.*

[See some examples in IronCalc](https://app.ironcalc.com/?example=vlookup).

## Links
* See also IronCalc's [HLOOKUP](/functions/lookup_and_reference/hlookup.md) function for horizontal lookups, and [INDEX](/functions/lookup_and_reference/index.md) and [MATCH](/functions/lookup_and_reference/match.md) functions for more flexible lookup operations.
* Visit Microsoft Excel's [VLOOKUP function](https://support.microsoft.com/en-us/office/vlookup-function-0bbc8083-26fe-4963-8ab8-93a18ad188a1) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093318) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/VLOOKUP) provide versions of the VLOOKUP function.

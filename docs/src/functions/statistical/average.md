---
layout: doc
outline: deep
lang: en-US
---

# AVERAGE function

## Overview
AVERAGE is a function of the Statistical category that calculates the arithmetic mean of a set of numeric values.

The function sums all numeric values provided in its arguments and divides by the count of those values. AVERAGE can process individual values, cell references, and ranges, automatically ignoring non-numeric values in ranges while including numeric strings provided as direct arguments.

## Usage

### Syntax
**AVERAGE(<span title="Number or Range" style="color:#1E88E5">value1</span>, [<span title="Number or Range" style="color:#1E88E5">value2</span>], ...) => <span title="Number" style="color:#1E88E5">average</span>**

### Argument descriptions
* *value1* ([number](/features/value-types#numbers) or [range](/features/value-types#ranges), required). The first numeric value, cell reference, or range to include in the average calculation.
* *value2, ...* ([number](/features/value-types#numbers) or [range](/features/value-types#ranges), [optional](/features/optional-arguments.md)). Additional numeric values, cell references, or ranges to include in the average calculation. Up to 255 additional arguments may be provided.

### Additional guidance
* AVERAGE handles different data types in arguments depending on how they are provided:
  * **Numbers in ranges or references:** Only numeric values are counted. Text, booleans, and empty cells are ignored.
  * **Direct numeric arguments:** Numbers provided directly (not from cell references) are always included.
  * **Boolean values:** When provided as direct arguments, TRUE is treated as 1 and FALSE as 0. When in ranges or cell references, boolean values are ignored.
  * **Text values:** When provided as direct arguments, text that can be parsed as a number (e.g., "123") is included in the calculation. Text that cannot be parsed returns an error. Text in ranges or cell references is ignored.
* All ranges must be within the same sheet. Ranges spanning multiple sheets will return an error.
* Empty cells are always ignored and do not affect the count.

### Returned value
AVERAGE returns a [number](/features/value-types#numbers) representing the arithmetic mean of all counted numeric values. This is calculated as the sum of all values divided by the count of values.

### Error conditions
* In common with many other IronCalc functions, AVERAGE propagates errors that are found in its arguments.
* If no arguments are supplied, then AVERAGE returns the [`#ERROR!`](/features/error-types.md#error) error.
* If no numeric values are found among the arguments (resulting in a division by zero), then AVERAGE returns the [`#DIV/0!`](/features/error-types.md#div0) error.
* If a range spans multiple sheets, then AVERAGE returns the [`#VALUE!`](/features/error-types.md#value) error.
* If a text value provided as a direct argument cannot be converted to a number, then AVERAGE returns the [`#VALUE!`](/features/error-types.md#value) error.

## Examples

**Example 1: Basic average of numeric values**

Calculate the average of a simple set of numbers.

`=AVERAGE(10, 20, 30, 40)`

*Result: 25 (the sum 100 divided by 4 values)*

**Example 2: Average of a range with mixed data types**

Given cells B1=1, B2=2, B3=3, B4="2" (text), B5 is empty, B6=TRUE, calculate the average of the range.

`=AVERAGE(B1:B6)`

*Result: 2 (the sum 6 divided by 3 numeric values: 1, 2, and 3. The text "2", empty cell, and boolean TRUE are ignored)*

**Example 3: Average with direct boolean and numeric arguments**

Calculate an average where boolean values are included as direct arguments.

`=AVERAGE(10, TRUE, FALSE, 20)`

*Result: 7.75 (the sum 31 divided by 4 values, where TRUE=1 and FALSE=0)*

[See some examples in IronCalc](https://app.ironcalc.com/?example=average).

## Links
* See also IronCalc's [AVERAGEA](/functions/statistical/averagea.md) function, which includes text and boolean values from ranges in its calculation.
* See also IronCalc's [AVERAGEIF](/functions/statistical/averageif.md) and [AVERAGEIFS](/functions/statistical/averageifs.md) functions for conditional averaging.
* Visit Microsoft Excel's [AVERAGE function](https://support.microsoft.com/en-us/office/average-function-047bac88-d466-426c-a32b-8f33eb960cf6) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093615) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/AVERAGE) provide versions of the AVERAGE function.

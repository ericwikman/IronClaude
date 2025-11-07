---
layout: doc
outline: deep
lang: en-US
---

# SUM function

## Overview
SUM is a function of the Math and Trigonometry category that calculates the sum of all numbers provided in its arguments. It is one of the most commonly used functions in spreadsheet applications, allowing users to quickly total values from multiple cells, ranges, or direct numeric inputs.

## Usage

### Syntax
**SUM(<span title="Number" style="color:#1E88E5">number1</span>, [<span title="Number" style="color:#1E88E5">number2</span>, ...]) => <span title="Number" style="color:#1E88E5">total</span>**

### Argument descriptions
* *number1* ([number](/features/value-types#numbers), required). The first numeric value, cell reference, range, or array to include in the sum.
* *number2, ...* ([number](/features/value-types#numbers), [optional](/features/optional-arguments.md)). Additional numeric values, cell references, ranges, or arrays to include in the sum. Up to 255 arguments may be provided.

### Additional guidance
* SUM ignores text values and Boolean values (TRUE/FALSE) within ranges and arrays. Only numeric values are included in the calculation.
* Empty cells and empty arguments are treated as zero and do not affect the sum.
* If a range spans an entire column (e.g., A:A) or an entire row (e.g., 1:1), SUM automatically optimizes performance by summing only the cells within the worksheet's used dimension rather than all theoretical cells.
* SUM can process arrays created with curly brace notation, including both horizontal arrays (e.g., {1, 2, 3}), vertical arrays (e.g., {1; 2; 3}), and two-dimensional arrays (e.g., {1, 2; 3, 4}).

### Returned value
SUM returns a [number](/features/value-types#numbers) representing the total of all numeric values in the supplied arguments.

### Error conditions
* If no arguments are supplied, then SUM returns the [`#ERROR!`](/features/error-types.md#error) error with the message "Wrong number of arguments".
* If a range argument spans cells on different sheets (e.g., Sheet1!A1:Sheet2!A10), then SUM returns the [`#VALUE!`](/features/error-types.md#value) error with the message "Ranges are in different sheets".
* If any argument references an invalid worksheet index, then SUM returns the [`#ERROR!`](/features/error-types.md#error) error.
* In common with many other IronCalc functions, SUM propagates errors that are found in its arguments. If any cell or array element contains an error value, SUM immediately returns that error.

## Examples

**Example 1: Summing individual values**
Calculate the sum of three numeric values.
`=SUM(1, 2, 3)`
*Result: 6. The function adds the three numbers together.*

**Example 2: Summing a range of cells**
Calculate the total of values in cells A1 through A3.
`=SUM(A1:A3)`
*Result: If A1 contains 1, A2 contains 2, and A3 contains 3, the result is 6.*

**Example 3: Summing with empty arguments**
Calculate the sum when some arguments are omitted.
`=SUM(1, , 3)`
*Result: 4. The empty argument is treated as zero.*

**Example 4: Summing a horizontal array**
Calculate the sum of values in a horizontal array.
`=SUM({1, 2, 3})`
*Result: 6. All values in the array are added together.*

**Example 5: Summing a two-dimensional array**
Calculate the sum of all values in a 2D array.
`=SUM({1, 2; 3, 4; 5, 6})`
*Result: 21. All six values in the array are summed.*

**Example 6: Combining multiple ranges and values**
Calculate the sum of a range and an additional value.
`=SUM(A1, B1)`
*Result: If A1 contains 42 and B1 contains 23, the result is 65.*

[See some examples in IronCalc](https://app.ironcalc.com/?example=sum).

## Links
* See also IronCalc's [SUMIF](/functions/math_and_trigonometry/sumif.md) and [SUMIFS](/functions/math_and_trigonometry/sumifs.md) functions for conditional summing.
* Visit Microsoft Excel's [SUM function](https://support.microsoft.com/en-us/office/sum-function-043e1c7d-7726-4e80-8f32-07b23e057f89) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093669) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/SUM) provide versions of the SUM function.
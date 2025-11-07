---
layout: doc
outline: deep
lang: en-US
---

# ABS function

## Overview
ABS is a function of the Math and Trigonometry category that returns the absolute value of a number, which is the number without its sign.

ABS is commonly used to work with distances, magnitudes, or differences where only the positive magnitude is relevant, regardless of whether the input value is positive or negative.

## Usage

### Syntax
**ABS(<span title="Number" style="color:#1E88E5">number</span>) => <span title="Number" style="color:#1E88E5">abs</span>**

### Argument descriptions
* *number* ([number](/features/value-types#numbers), required). A numeric value for which the absolute value is to be calculated. Can be a positive number, negative number, zero, or a [Boolean](/features/value-types#booleans) value (where TRUE is treated as 1 and FALSE is treated as 0).

### Additional guidance
* ABS ignores the sign of the input value and always returns a non-negative result.
* The function accepts [Boolean](/features/value-types#booleans) values: TRUE is converted to 1 and FALSE is converted to 0 before the absolute value is calculated.
* If the input is a text string that can be parsed as a number, ABS will attempt to convert it and return the absolute value; otherwise, it returns the [`#VALUE!`](/features/error-types.md#value) error.

### Returned value
ABS returns a [number](/features/value-types#numbers) representing the absolute value of the input. The returned value is always non-negative (zero or positive).

### Error conditions
* In common with many other IronCalc functions, ABS propagates errors that are found in its arguments.
* If no argument, or more than one argument, is supplied, then ABS returns the [`#ERROR!`](/features/error-types.md#error) error.
* If the value of the *number* argument is not (or cannot be converted to) a [number](/features/value-types#numbers), then ABS returns the [`#VALUE!`](/features/error-types.md#value) error.
* If the *number* argument contains an error (such as [`#DIV/0!`](/features/error-types.md#div0), [`#N/A`](/features/error-types.md#na), etc.), then ABS returns that error unchanged.

## Examples

**Example 1: Converting a negative number to its positive magnitude.**
Calculate the absolute value of a negative number, commonly used when you need the magnitude of a temperature change or price difference.
`=ABS(-42)`
*Result: 42. The absolute value of -42 is 42.*

**Example 2: Converting a positive number (which returns itself).**
Calculate the absolute value of an already positive number, showing that ABS returns the same positive value unchanged.
`=ABS(3.14)`
*Result: 3.14. The absolute value of 3.14 remains 3.14.*

**Example 3: Converting a Boolean value.**
Demonstrate ABS behavior with Boolean inputs, where TRUE equals 1 and FALSE equals 0.
`=ABS(TRUE)`
*Result: 1. The Boolean TRUE is converted to 1, and its absolute value is 1.*

**Example 4: Error propagation.**
Show how ABS propagates errors from its argument, such as a division by zero error.
`=ABS(1/0)`
*Result: #DIV/0!. The division by zero error is propagated through ABS.*

[See some examples in IronCalc](https://app.ironcalc.com/?example=abs).

## Links
* See also IronCalc's [SIGN](/functions/math_and_trigonometry/sign.md) function for determining whether a number is positive, negative, or zero.
* Visit Microsoft Excel's [ABS function](https://support.microsoft.com/en-us/office/abs-function-3060899f-9519-4e3d-b236-b3f3903b54f5) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093462) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/ABS) provide versions of the ABS function.
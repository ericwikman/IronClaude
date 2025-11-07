---
layout: doc
outline: deep
lang: en-US
---

# IF function

## Overview
IF is a function of the Logical category that returns one value when a specified condition evaluates to TRUE and another value when it evaluates to FALSE.

IF is one of the most fundamental and widely-used functions in spreadsheet applications. It enables conditional logic, allowing formulas to make decisions and return different results based on whether a test condition is met.

## Usage

### Syntax
**IF(<span title="Boolean" style="color:#43A047">logical_test</span>, <span title="Any" style="color:#1E88E5">value_if_true</span>, [<span title="Any" style="color:#1E88E5">value_if_false</span>]) => <span title="Any" style="color:#1E88E5">result</span>**

### Argument descriptions
* *logical_test* ([Boolean](/features/value-types#booleans), required). A condition or expression that evaluates to TRUE or FALSE. This can be a comparison (such as `A1 > 10`), a boolean value, or any expression that can be converted to a boolean.
* *value_if_true* (any [data type](/features/value-types), required). The value to return when *logical_test* evaluates to TRUE. This can be a number, text, boolean, cell reference, formula, or any other valid expression.
* *value_if_false* (any [data type](/features/value-types), [optional](/features/optional-arguments.md)). The value to return when *logical_test* evaluates to FALSE. If omitted, IF returns FALSE when the condition is not met.

### Additional guidance
* IF uses lazy evaluation, meaning it only evaluates the branch (*value_if_true* or *value_if_false*) that will actually be returned. The other branch is not evaluated at all.
* When the *value_if_false* argument is omitted and the condition is FALSE, IF returns the boolean value FALSE (not an empty cell or zero).
* The *value_if_true* and *value_if_false* arguments can be of different data types. The function returns whatever type is specified for the branch that is evaluated.
* IF functions can be nested inside one another to test multiple conditions. However, for testing multiple conditions, consider using the [IFS](/functions/logical/ifs.md) function for improved readability.

### Returned value
IF returns a value whose [data type](/features/value-types) matches the type of the argument that was evaluated (*value_if_true* or *value_if_false*). The return type can be a [number](/features/value-types#numbers), [text](/features/value-types#strings), [Boolean](/features/value-types#booleans), or any other valid spreadsheet value.

### Error conditions
* In common with many other IronCalc functions, IF propagates errors that are found in its arguments.
* If fewer than 2 arguments or more than 3 arguments are supplied, then IF returns the [`#ERROR!`](/features/error-types.md#error) error.
* If the value of the *logical_test* argument cannot be converted to a [Boolean](/features/value-types#booleans), then IF returns the [`#VALUE!`](/features/error-types.md#value) error.
* If an error occurs during the evaluation of *logical_test*, that error is propagated and returned by the IF function.

## Examples

**Example 1: Basic conditional check**
Check if a value exceeds a threshold and return appropriate text.
```
=IF(A1 > 100, "Over budget", "Within budget")
```
*Result: Returns "Over budget" if the value in A1 is greater than 100, otherwise returns "Within budget".*

**Example 2: Using IF with two arguments**
Test a condition without specifying a value for FALSE.
```
=IF(B1 = "Complete", "Done")
```
*Result: Returns "Done" if B1 contains "Complete", otherwise returns FALSE.*

**Example 3: Numeric comparison with calculations**
Apply different calculation methods based on a condition.
```
=IF(C1 < 0, C1 * -1, C1 * 2)
```
*Result: If C1 is negative, returns its absolute value (multiplied by -1). If C1 is zero or positive, returns C1 multiplied by 2.*

**Example 4: Nested IF for multiple conditions**
Test multiple conditions using nested IF functions.
```
=IF(D1 >= 90, "A", IF(D1 >= 80, "B", IF(D1 >= 70, "C", "F")))
```
*Result: Returns a letter grade based on the score in D1. Scores of 90 or above get "A", 80-89 get "B", 70-79 get "C", and below 70 gets "F".*

[See some examples in IronCalc](https://app.ironcalc.com/?example=if).

## Links
* See also IronCalc's [IFS](/functions/logical/ifs.md) function for testing multiple conditions without nesting.
* See also IronCalc's [IFERROR](/functions/logical/iferror.md) and [IFNA](/functions/logical/ifna.md) functions for specialized error handling.
* See also IronCalc's [SWITCH](/functions/logical/switch.md) function for matching a value against multiple cases.
* Visit Microsoft Excel's [IF function](https://support.microsoft.com/en-us/office/if-function-69aed7c9-4e8a-4755-a9bc-aa8bbff73be2) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093364) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/IF) provide versions of the IF function.

---
layout: doc
outline: deep
lang: en-US
---

# TODAY function

## Overview
TODAY is a function of the Date and Time category that returns the current date as a [serial number](/features/serial-numbers.md).

TODAY retrieves the current date from the system clock and converts it to a serial number that represents the date portion only. The function respects the timezone configuration of the IronCalc model, ensuring that the returned date matches the model's local date.

## Usage

### Syntax
**TODAY() => <span title="Number" style="color:#1E88E5">serial_number</span>**

### Argument descriptions
TODAY takes no arguments. The function is called with empty parentheses.

### Additional guidance
* TODAY is a volatile function, meaning it recalculates automatically whenever the worksheet is evaluated. The returned value will update to reflect the current date.
* The date returned by TODAY is determined by the timezone configuration of the IronCalc model. Two users in different timezones may see different dates at the same moment in time.
* TODAY returns only the date portion as a serial number. To include the current time as well, use the [NOW](/functions/date_and_time/now.md) function.
* The serial number returned by TODAY can be formatted using cell formatting or the [TEXT](/functions/text/text.md) function to display the date in various human-readable formats.

### Returned value
TODAY returns a [number](/features/value-types#numbers) representing the current date as a [serial number](/features/serial-numbers.md). The serial number corresponds to the number of days since December 31, 1899.

The returned value is an integer (the fractional time component is removed), representing midnight at the start of the current day in the model's configured timezone.

### Error conditions
* If any arguments are supplied to TODAY, the function returns the [`#ERROR!`](/features/error-types.md#error) error.
* If the system clock or timezone configuration produces an invalid date, TODAY returns the [`#ERROR!`](/features/error-types.md#error) error.

## Examples

**Example 1: Getting the current date.**
To retrieve today's date as a serial number:
```
=TODAY()
```
*Result: A serial number such as `45005` (representing March 20, 2023). The exact value depends on the current date when the formula is evaluated.*

**Example 2: Calculating the number of days until a future date.**
To find how many days remain until December 31, 2025:
```
=DATE(2025,12,31)-TODAY()
```
*Result: The number of days between today and December 31, 2025. This value decreases by 1 each day.*

**Example 3: Extracting the current year.**
To get the current year as a number:
```
=YEAR(TODAY())
```
*Result: `2023` (or the current year when evaluated). This is useful for creating dynamic date-based formulas.*

**Example 4: Displaying the current date in a readable format.**
To display today's date as formatted text:
```
=TEXT(TODAY(), "dddd, mmmm d, yyyy")
```
*Result: A text string such as `"Monday, March 20, 2023"` formatted according to the specified pattern.*

[See some examples in IronCalc](https://app.ironcalc.com/?example=today).

## Links
* See also IronCalc's [NOW](/functions/date_and_time/now.md) function for retrieving the current date and time together.
* See also IronCalc's [DATE](/functions/date_and_time/date.md), [YEAR](/functions/date_and_time/year.md), [MONTH](/functions/date_and_time/month.md), and [DAY](/functions/date_and_time/day.md) functions for working with dates.
* Visit Microsoft Excel's [TODAY function](https://support.microsoft.com/en-us/office/today-function-5eb3078d-a82c-4736-8930-2f51a028397) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3092984) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/TODAY) provide versions of the TODAY function.

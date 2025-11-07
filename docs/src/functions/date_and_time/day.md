---
layout: doc
outline: deep
lang: en-US
---

# DAY function

## Overview
DAY is a function of the Date and Time category that extracts the day of the month from a date [serial number](/features/serial-numbers.md). This function is commonly used to isolate the day component from date values for analysis, reporting, or conditional logic.

## Usage

### Syntax
**DAY(<span title="Number" style="color:#1E88E5">date</span>) => <span title="Number" style="color:#1E88E5">day</span>**

### Argument descriptions
* *date* ([number](/features/value-types#numbers), required). The date for which the day of the month is to be extracted, expressed as a [serial number](/features/serial-numbers.md). The serial number must be in the range [1, 2958465], where 1 corresponds to December 31, 1899 and 2958465 corresponds to December 31, 9999.

### Additional guidance
* If the supplied *date* argument has a fractional part (representing a time component), DAY uses its [floor value](https://en.wikipedia.org/wiki/Floor_and_ceiling_functions), effectively discarding the time portion.
* At present, DAY does not accept a string representation of a date literal as an argument. For example, the formula `=DAY("2024-12-31")` returns the [`#VALUE!`](/features/error-types.md#value) error. Use the [DATEVALUE](/functions/date_and_time/datevalue.md) function to convert date text to a serial number first.

### Returned value
DAY returns a [number](/features/value-types#numbers) representing the day of the month according to the [Gregorian calendar](https://en.wikipedia.org/wiki/Gregorian_calendar). The returned value is always an integer in the range [1, 31].

### Error conditions
* In common with many other IronCalc functions, DAY propagates errors that are found in its argument.
* If no argument, or more than one argument, is supplied, then DAY returns the [`#ERROR!`](/features/error-types.md#error) error.
* If the value of the *date* argument is not (or cannot be converted to) a [number](/features/value-types#numbers), then DAY returns the [`#VALUE!`](/features/error-types.md#value) error.
* If the *date* argument is less than 1 or greater than 2,958,465, then DAY returns the [`#NUM!`](/features/error-types.md#num) error, indicating the date is outside the valid range.

## Details
* IronCalc utilizes Rust's [chrono](https://docs.rs/chrono/latest/chrono/) crate to implement the DAY function, ensuring accurate date calculations across the supported date range.
* IronCalc handles the historical Excel 1900 leap year bug by treating serial number 60 as February 28, 1900, rather than the non-existent February 29, 1900 that Excel recognizes. From serial number 61 onward (March 1, 1900), IronCalc's date calculations align with all major spreadsheet applications.

## Examples

**Example 1: Extract day from a serial number.**
Extract the day of the month from serial number 45321 (December 28, 2024).
```
=DAY(45321)
```
*Result: 28*

**Example 2: Extract day from today's date.**
Get the current day of the month using the TODAY function.
```
=DAY(TODAY())
```
*Result: Returns the day component of the current date (e.g., 7 if today is the 7th).*

**Example 3: Extract day from a date with time component.**
The DAY function ignores the time portion of a date-time serial number.
```
=DAY(27819.75)
```
*Result: 29 (February 29, 1976; the .75 representing 6:00 PM is discarded).*

**Example 4: Handle boundary cases.**
Test the maximum valid date serial number.
```
=DAY(2958465)
```
*Result: 31 (December 31, 9999).*

[See some examples in IronCalc](https://app.ironcalc.com/?example=day).

## Links
* See also IronCalc's [MONTH](/functions/date_and_time/month.md) and [YEAR](/functions/date_and_time/year.md) functions for extracting other date components.
* See also IronCalc's [DATEVALUE](/functions/date_and_time/datevalue.md) function for converting date text to serial numbers.
* Visit Microsoft Excel's [DAY function](https://support.microsoft.com/en-gb/office/day-function-8a7d1cbb-6c7d-4ba1-8aea-25c134d03101) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093040) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/DAY) provide versions of the DAY function.

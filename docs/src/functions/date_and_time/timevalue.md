---
layout: doc
outline: deep
lang: en-US
---

# TIMEVALUE function

## Overview
TIMEVALUE is a function of the Date and Time category that converts a time stored as text to a [serial number](/features/serial-numbers.md) representing a fraction of a 24-hour day.

## Usage
### Syntax
**TIMEVALUE(<span title="Text" style="color:#1E88E5">time_text</span>) => <span title="Number" style="color:#1E88E5">timevalue</span>**

### Argument descriptions
* *time_text* ([text](/features/value-types#strings), required). A text string that represents a time in a recognized format. The text can represent a time-only value or include a date component (which will be ignored).

### Additional guidance
* If the *time_text* argument includes both date and time information, TIMEVALUE extracts and processes only the time portion, ignoring the date component.
* Leading and trailing whitespace in the *time_text* argument is automatically trimmed before processing.
* TIMEVALUE supports multiple time formats, including:
  * 24-hour format: "14:30", "14:30:45"
  * 12-hour format with AM/PM: "2:30 PM", "2 PM", "2:30:45 AM"
  * Date-time combinations: "2023-01-01 14:30:00", "22-Aug-2011 6:35 AM"
  * ISO 8601 formats: "2023-01-01T14:30:00"
* Special handling is provided for edge cases such as "12:00 AM" (midnight) and "12:00 PM" (noon) to match Excel behavior.

### Returned value
TIMEVALUE returns a [number](/features/value-types#numbers) between 0 and 1 representing the time as a fraction of a 24-hour day. For example, 12:00 PM (noon) returns 0.5, representing half a day.

### Error conditions
* In common with many other IronCalc functions, TIMEVALUE propagates errors that are found in its argument.
* If no argument, or more than one argument, is supplied, then TIMEVALUE returns the [`#ERROR!`](/features/error-types.md#error) error.
* If the value of the *time_text* argument is not (or cannot be converted to) a [text](/features/value-types#strings) value, then TIMEVALUE returns the [`#VALUE!`](/features/error-types.md#value) error.
* If the *time_text* argument cannot be recognized as a valid time format, then TIMEVALUE returns the [`#VALUE!`](/features/error-types.md#value) error.
* If the *time_text* argument contains invalid time values (such as hours greater than 24 or minutes greater than 59), then TIMEVALUE returns the [`#VALUE!`](/features/error-types.md#value) error.

## Examples

**Example 1: Converting 12-hour format time with AM/PM notation.**
Convert "2:24 AM" to its decimal representation as a fraction of a day.
`=TIMEVALUE("2:24 AM")`
*Result: 0.1 (representing 2 hours and 24 minutes, which is 2.4/24 = 0.1 of a day).*

**Example 2: Converting time from a date-time string.**
Extract the time portion from a full date-time string, ignoring the date.
`=TIMEVALUE("22-Aug-2011 6:35 AM")`
*Result: 0.274305556 (representing 6:35 AM as a fraction of a day).*

**Example 3: Converting 24-hour format time.**
Convert "18:45" in 24-hour notation to its decimal representation.
`=TIMEVALUE("18:45")`
*Result: 0.78125 (representing 18 hours and 45 minutes, which is 18.75/24 = 0.78125 of a day).*

**Example 4: Handling noon and midnight edge cases.**
Convert "12:00 PM" (noon) to its decimal representation.
`=TIMEVALUE("12:00 PM")`
*Result: 0.5 (representing noon, which is exactly half a day).*

[See some examples in IronCalc](https://app.ironcalc.com/?example=timevalue).

## Links
* See also IronCalc's [DATEVALUE](/functions/date_and_time/datevalue.md) function for converting date text to serial numbers.
* See also IronCalc's [TIME](/functions/date_and_time/time.md) function for creating time values from hour, minute, and second components.
* Visit Microsoft Excel's [TIMEVALUE function](https://support.microsoft.com/en-us/office/timevalue-function-0b615c12-33d8-4431-bf3d-f3eb6d186645) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093056) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/TIMEVALUE) provide versions of the TIMEVALUE function.
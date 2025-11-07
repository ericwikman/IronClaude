---
layout: doc
outline: deep
lang: en-US
---

# PMT function

## Overview
PMT (<u>P</u>ay<u>m</u>en<u>t</u>) is a function of the Financial category that calculates the fixed periodic payment required for a loan or investment based on constant payments and a constant interest rate.

PMT can be used to determine the payment amount for loans, mortgages, annuities, and other financial instruments where regular payments are made over a specified number of periods. The function assumes a fixed interest rate and equal payment amounts throughout the payment schedule.

## Usage

### Syntax
**PMT(<span title="Number" style="color:#1E88E5">rate</span>, <span title="Number" style="color:#1E88E5">nper</span>, <span title="Number" style="color:#1E88E5">pv</span>, [<span title="Number" style="color:#1E88E5">fv</span>=0], [<span title="Boolean" style="color:#43A047">type</span>=FALSE]) => <span title="Number" style="color:#1E88E5">pmt</span>**

### Argument descriptions
* *rate* ([number](/features/value-types#numbers), required). The fixed percentage interest rate per period.
* *nper* ([number](/features/value-types#numbers), required). "nper" stands for <u>n</u>umber of <u>per</u>iods, in this case the total number of payment periods. While this will often be an integer, non-integer values are accepted and processed.
* *pv* ([number](/features/value-types#numbers), required). "pv" is the <u>p</u>resent <u>v</u>alue or principal amount of the loan or investment.
* *fv* ([number](/features/value-types#numbers), [optional](/features/optional-arguments.md)). "fv" is the <u>f</u>uture <u>v</u>alue or cash balance after the last payment is made (default 0).
* *type* ([Boolean](/features/value-types#booleans), [optional](/features/optional-arguments.md)). A logical value indicating whether payments are due at the end (FALSE or 0) of each period or at the beginning (TRUE or any non-zero value). The default is FALSE when omitted.

### Additional guidance
* Make sure that the *rate* argument specifies the interest rate applicable to the payment period, based on the value chosen for *nper*. For example, if *nper* represents monthly payments, *rate* must be the monthly interest rate (annual rate divided by 12).
* The *pv* and *fv* arguments should be expressed in the same currency unit.
* The returned payment value is typically negative, representing money paid out. The sign convention follows standard financial practice where cash outflows are negative and cash inflows are positive.

### Returned value
PMT returns a [number](/features/value-types#numbers) representing the fixed periodic payment amount expressed in the same [currency unit](/features/units) that was used for the *pv* and *fv* arguments.

### Error conditions
* In common with many other IronCalc functions, PMT propagates errors that are found in its arguments.
* If fewer than 3 arguments or more than 5 arguments are supplied, then PMT returns the [`#ERROR!`](/features/error-types.md#error) error.
* If the value of any of the *rate*, *nper*, *pv* or *fv* arguments is not (or cannot be converted to) a [number](/features/value-types#numbers), then PMT returns the [`#VALUE!`](/features/error-types.md#value) error.
* If the value of the *type* argument is not (or cannot be converted to) a [Boolean](/features/value-types#booleans), then PMT again returns the [`#VALUE!`](/features/error-types.md#value) error.
* If *rate* is less than or equal to -1, then PMT returns the [`#NUM!`](/features/error-types.md#num) error.
* If *nper* equals 0 and *rate* equals 0, then PMT returns the [`#NUM!`](/features/error-types.md#num) error.
* If the calculation produces a result that is not a valid number (NaN or infinite), then PMT returns the [`#NUM!`](/features/error-types.md#num) error.

## Details
* When $\text{rate} = 0$, the payment is calculated using simple division:
$ \text{pmt} = -\dfrac{\text{pv} + \text{fv}}{\text{nper}}$

* When $\text{rate} \neq 0$ and $\text{type} = 0$ (payments at the end of the period), the payment is given by:
$ \text{pmt} = \dfrac{\text{fv} \times \text{rate} + \text{pv} \times \text{rate} \times (1 + \text{rate})^\text{nper}}{1 - (1 + \text{rate})^\text{nper}}$

* When $\text{rate} \neq 0$ and $\text{type} \neq 0$ (payments at the beginning of the period), the payment is given by:
$ \text{pmt} = \dfrac{(\text{fv} + \text{pv} \times (1 + \text{rate})^\text{nper}) \times \text{rate}}{(1 + \text{rate}) \times (1 - (1 + \text{rate})^\text{nper})}$

* These formulas are derived from the fundamental compound interest equation:
$ \text{pv} \times (1+\text{rate})^\text{nper} + \text{pmt} \times (1+\text{rate} \times \text{type}) \times \dfrac{(1+\text{rate})^\text{nper}-1}{\text{rate}} + \text{fv} = 0$

## Examples

**Example 1: Calculate monthly payment for a car loan.**
Calculate the monthly payment for a $10,000 car loan with an 8% annual interest rate over 10 months.
`=PMT(8%/12, 10, 10000)`
*Result: -$1,037.03. The payment is negative because it represents a cash outflow. The annual rate is divided by 12 to get the monthly rate.*

**Example 2: Calculate lease payment with payments at the beginning of the period.**
Calculate the monthly payment for the same loan when payments are made at the beginning of each month.
`=PMT(8%/12, 10, 10000, , 1)`
*Result: -$1,030.16. The payment is slightly lower because payments made at the beginning of the period accrue less interest.*

**Example 3: Calculate payment with a future value target.**
Calculate the payment needed when the interest rate is 20% per period, with 3 periods, starting with -200, to reach a future value of 0.
`=PMT(0.2, 3, -200)`
*Result: 92.42. The positive result represents cash inflow needed to pay off the debt.*

[See some examples in IronCalc](https://app.ironcalc.com/?example=pmt).

## Links
* See also IronCalc's [FV](/functions/financial/fv.md), [NPER](/functions/financial/nper.md), [PV](/functions/financial/pv.md) and [RATE](/functions/financial/rate.md) functions for related financial calculations.
* See also IronCalc's [IPMT](/functions/financial/ipmt.md) and [PPMT](/functions/financial/ppmt.md) functions for calculating the interest and principal portions of a payment.
* Visit Microsoft Excel's [PMT function](https://support.microsoft.com/en-us/office/pmt-function-0214da64-9a63-4996-bc20-214433fa6441) page.
* Both [Google Sheets](https://support.google.com/docs/answer/3093185) and [LibreOffice Calc](https://wiki.documentfoundation.org/Documentation/Calc_Functions/PMT) provide versions of the PMT function.

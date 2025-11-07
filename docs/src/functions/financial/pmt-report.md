# Code Context Report for: `PMT`

This report contains all relevant code snippets for the `PMT` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `base/src/functions/financial.rs`
**Lines:** `343 - 389`

```rust
/// PMT(rate, nper, pv, [fv], [type])
pub(crate) fn fn_pmt(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    let arg_count = args.len();
    if !(3..=5).contains(&arg_count) {
        return CalcResult::new_args_number_error(cell);
    }
    let rate = match self.get_number(&args[0], cell) {
        Ok(f) => f,
        Err(s) => return s,
    };
    // number of periods
    let nper = match self.get_number(&args[1], cell) {
        Ok(f) => f,
        Err(s) => return s,
    };
    // present value
    let pv = match self.get_number(&args[2], cell) {
        Ok(f) => f,
        Err(s) => return s,
    };
    // future_value
    let fv = if arg_count > 3 {
        match self.get_number(&args[3], cell) {
            Ok(f) => f,
            Err(s) => return s,
        }
    } else {
        0.0
    };
    let period_start = if arg_count > 4 {
        match self.get_number(&args[4], cell) {
            Ok(f) => f != 0.0,
            Err(s) => return s,
        }
    } else {
        // at the end of the period
        false
    };
    match compute_payment(rate, nper, pv, fv, period_start) {
        Ok(p) => CalcResult::Number(p),
        Err(error) => CalcResult::Error {
            error: error.0,
            origin: cell,
            message: error.1,
        },
    }
}
```

## 2. Helper Function: compute_payment

**File:** `base/src/functions/financial.rs`
**Lines:** `44 - 75`

```rust
fn compute_payment(
    rate: f64,
    nper: f64,
    pv: f64,
    fv: f64,
    period_start: bool,
) -> Result<f64, (Error, String)> {
    if rate == 0.0 {
        if nper == 0.0 {
            return Err((Error::NUM, "Period count must be non zero".to_string()));
        }
        return Ok(-(pv + fv) / nper);
    }
    if rate <= -1.0 {
        return Err((Error::NUM, "Rate must be > -1".to_string()));
    };
    let rate_nper = if nper == 0.0 {
        1.0
    } else {
        (1.0 + rate).powf(nper)
    };
    let result = if period_start {
        // type = 1
        (fv + pv * rate_nper) * rate / ((1.0 + rate) * (1.0 - rate_nper))
    } else {
        (fv * rate + pv * rate * rate_nper) / (1.0 - rate_nper)
    };
    if result.is_nan() || result.is_infinite() {
        return Err((Error::NUM, "Invalid result".to_string()));
    }
    Ok(result)
}
```

## 3. Financial Functions Context

**File:** `base/src/functions/financial.rs`
**Lines:** `176 - 192`

```rust
// These formulas revolve around compound interest and annuities.
// The financial functions pv, rate, nper, pmt and fv:
// rate = interest rate per period
// nper (number of periods) = loan term
// pv (present value) = loan amount
// fv (future value) = cash balance after last payment. Default is 0
// type = the annuity type indicates when payments are due
//         * 0 (default) Payments are made at the end of the period
//         * 1 Payments are made at the beginning of the period (like a lease or rent)
// The variable period_start is true if type is 1
// They are linked by the formulas:
// If rate != 0
//   $pv*(1+rate)^nper+pmt*(1+rate*type)*((1+rate)^nper-1)/rate+fv=0$
// If rate = 0
//   $pmt*nper+pv+fv=0$
// All, except for rate are easily solvable in terms of the others.
// In these formulas the payment (pmt) is normally negative
```

## 4. Unit Tests

### Test Case 1: Argument count validation

**File:** `base/src/test/test_fn_financial.rs`
**Lines:** `6 - 49`

```rust
#[test]
fn fn_arguments() {
    let mut model = new_empty_model();
    model._set("A1", "=PMT()");
    model._set("A2", "=PMT(1,1)");
    model._set("A3", "=PMT(1,1,1,1,1,1)");

    model._set("B1", "=FV()");
    model._set("B2", "=FV(1,1)");
    model._set("B3", "=FV(1,1,1,1,1,1)");

    model._set("C1", "=PV()");
    model._set("C2", "=PV(1,1)");
    model._set("C3", "=PV(1,1,1,1,1,1)");

    model._set("D1", "=NPER()");
    model._set("D2", "=NPER(1,1)");
    model._set("D3", "=NPER(1,1,1,1,1,1)");

    model._set("E1", "=RATE()");
    model._set("E2", "=RATE(1,1)");
    model._set("E3", "=RATE(1,1,1,1,1,1)");

    model.evaluate();

    assert_eq!(model._get_text("A1"), *"#ERROR!");
    assert_eq!(model._get_text("A2"), *"#ERROR!");
    assert_eq!(model._get_text("A3"), *"#ERROR!");

    assert_eq!(model._get_text("B1"), *"#ERROR!");
    assert_eq!(model._get_text("B2"), *"#ERROR!");
    assert_eq!(model._get_text("B3"), *"#ERROR!");

    assert_eq!(model._get_text("C1"), *"#ERROR!");
    assert_eq!(model._get_text("C2"), *"#ERROR!");
    assert_eq!(model._get_text("C3"), *"#ERROR!");

    assert_eq!(model._get_text("D1"), *"#ERROR!");
    assert_eq!(model._get_text("D2"), *"#ERROR!");
    assert_eq!(model._get_text("D3"), *"#ERROR!");

    assert_eq!(model._get_text("E1"), *"#ERROR!");
    assert_eq!(model._get_text("E2"), *"#ERROR!");
    assert_eq!(model._get_text("E3"), *"#ERROR!");
}
```

### Test Case 2: Currency formatting with USD

**File:** `base/src/test/test_currency.rs`
**Lines:** `5 - 14`

```rust
#[test]
fn test_cell_currency_dollar() {
    let mut model = new_empty_model();
    model._set("A1", "=PMT(8/1200,10,10000)");
    model.evaluate();

    assert_eq!(model._get_text("A1"), "-$1,037.03");

    assert!(model.set_currency("EUR").is_ok());
}
```

### Test Case 3: Currency formatting with EUR

**File:** `base/src/test/test_currency.rs`
**Lines:** `16 - 24`

```rust
#[test]
fn test_cell_currency_euro() {
    let mut model = new_empty_model();
    assert!(model.set_currency("EUR").is_ok());
    model._set("A1", "=PMT(8/1200,10,10000)");
    model.evaluate();

    assert_eq!(model._get_text("A1"), "-€1,037.03");
}
```

## 5. Usage Examples

### Example 1: User input with cell references and default type

**File:** `base/src/test/test_set_user_input.rs`
**Lines:** `350 - 369`

```rust
let mut model = new_empty_model();
model._set("A2", "8%");
model._set("A3", "10");
model._set("A4", "$10,000");

model
    .set_user_input(0, 5, 1, "=PMT(A2/12,A3,A4)".to_string())
    .unwrap();
model
    .set_user_input(0, 6, 1, "=PMT(A2/12,A3,A4,,1)".to_string())
    .unwrap();
model
    .set_user_input(0, 7, 1, "=PMT(0.2, 3, -200)".to_string())
    .unwrap();

model.evaluate();

// This two are negative numbers
assert_eq!(model._get_text("A5"), *"-$1,037.03");
assert_eq!(model._get_text("A6"), *"-$1,030.16");
```

### Example 2: Direct usage in compute_ipmt

**File:** `base/src/functions/financial.rs`
**Lines:** `107 - 146`

```rust
fn compute_ipmt(
    rate: f64,
    period: f64,
    period_count: f64,
    present_value: f64,
    future_value: f64,
    period_start: bool,
) -> Result<f64, (Error, String)> {
    // http://www.staff.city.ac.uk/o.s.kerr/CompMaths/WSheet4.pdf
    // https://www.experts-exchange.com/articles/1948/A-Guide-to-the-PMT-FV-IPMT-and-PPMT-Functions.html
    // type = 0 (end of period)
    // impt = -[(1+rate)^(period-1)*(pv*rate+pmt)-pmt]
    // ipmt = FV(rate, period-1, payment, pv, type) * rate
    // type = 1 (beginning of period)
    // ipmt = (FV(rate, period-2, payment, pv, type) - payment) * rate
    let payment = compute_payment(
        rate,
        period_count,
        present_value,
        future_value,
        period_start,
    )?;
    if period < 1.0 || period >= period_count + 1.0 {
        return Err((
            Error::NUM,
            format!("Period must be between 1 and {}", period_count + 1.0),
        ));
    }
    if period == 1.0 && period_start {
        Ok(0.0)
    } else {
        let p = if period_start {
            period - 2.0
        } else {
            period - 1.0
        };
        let c = if period_start { -payment } else { 0.0 };
        let fv = compute_future_value(rate, p, payment, present_value, period_start)?;
        Ok((fv + c) * rate)
    }
}
```

### Example 3: Direct usage in compute_ppmt

**File:** `base/src/functions/financial.rs`
**Lines:** `149 - 174`

```rust
fn compute_ppmt(
    rate: f64,
    period: f64,
    period_count: f64,
    present_value: f64,
    future_value: f64,
    period_start: bool,
) -> Result<f64, (Error, String)> {
    let payment = compute_payment(
        rate,
        period_count,
        present_value,
        future_value,
        period_start,
    )?;
    // It's a bit unfortunate that the first thing compute_ipmt does is compute_payment again
    let ipmt = compute_ipmt(
        rate,
        period,
        period_count,
        present_value,
        future_value,
        period_start,
    )?;
    Ok(payment - ipmt)
}
```

## 6. Related Type Definitions

### Type: `Node`

**File:** `base/src/expressions/parser/mod.rs`
**Lines:** `106 - 136`

```rust
pub enum Node {
    BooleanKind(bool),
    NumberKind(f64),
    StringKind(String),
    ReferenceKind {
        sheet_name: Option<String>,
        sheet_index: u32,
        absolute_row: bool,
        absolute_column: bool,
        row: i32,
        column: i32,
    },
    RangeKind {
        sheet_name: Option<String>,
        sheet_index: u32,
        absolute_row1: bool,
        absolute_column1: bool,
        row1: i32,
        column1: i32,
        absolute_row2: bool,
        absolute_column2: bool,
        row2: i32,
        column2: i32,
    },
    WrongReferenceKind {
        sheet_name: Option<String>,
        absolute_row: bool,
        absolute_column: bool,
        row: i32,
        column: i32,
    },
    // ... (additional variants exist in the full enum)
}
```

### Type: `CellReferenceIndex`

**File:** `base/src/expressions/types.rs`
**Lines:** `37 - 42`

```rust
#[derive(Serialize, Deserialize, Debug, Clone, PartialEq, Eq, Copy)]
pub struct CellReferenceIndex {
    pub sheet: u32,
    pub column: i32,
    pub row: i32,
}
```

### Type: `CalcResult`

**File:** `base/src/calc_result.rs`
**Lines:** `12 - 48`

```rust
#[derive(Clone)]
pub(crate) enum CalcResult {
    String(String),
    Number(f64),
    Boolean(bool),
    Error {
        error: Error,
        origin: CellReferenceIndex,
        message: String,
    },
    Range {
        left: CellReferenceIndex,
        right: CellReferenceIndex,
    },
    EmptyCell,
    EmptyArg,
    Array(Vec<Vec<ArrayNode>>),
}

impl CalcResult {
    pub fn new_error(error: Error, origin: CellReferenceIndex, message: String) -> CalcResult {
        CalcResult::Error {
            error,
            origin,
            message,
        }
    }
    pub fn new_args_number_error(origin: CellReferenceIndex) -> CalcResult {
        CalcResult::Error {
            error: Error::ERROR,
            origin,
            message: "Wrong number of arguments".to_string(),
        }
    }
    pub fn is_error(&self) -> bool {
        matches!(self, CalcResult::Error { .. })
    }
}
```

### Type: `Error`

**File:** `base/src/expressions/token.rs`
**Lines:** `84 - 97`

```rust
pub enum Error {
    REF,
    NAME,
    VALUE,
    DIV,
    NA,
    NUM,
    ERROR,
    NIMPL,
    SPILL,
    CALC,
    CIRC,
    NULL,
}
```

## 7. Function Registration

### Registration in Function Enum

**File:** `base/src/functions/mod.rs`
**Lines:** `236`

```rust
Pmt,
```

### Registration in Function Iterator

**File:** `base/src/functions/mod.rs`
**Lines:** `477`

```rust
Function::Pmt,
```

### Registration in Function Name Mapping

**File:** `base/src/functions/mod.rs`
**Lines:** `816`

```rust
"PMT" => Some(Function::Pmt),
```

### Registration in Display Implementation

**File:** `base/src/functions/mod.rs`
**Lines:** `1055`

```rust
Function::Pmt => write!(f, "PMT"),
```

### Registration in evaluate_function

**File:** `base/src/functions/mod.rs`
**Lines:** `1332`

```rust
Function::Pmt => self.fn_pmt(args, cell),
```

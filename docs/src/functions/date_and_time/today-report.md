# Code Context Report for: `TODAY`

This report contains all relevant code snippets for the `TODAY` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `base/src/functions/date_and_time.rs`
**Lines:** `953 - 969`

```rust
pub(crate) fn fn_today(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if !args.is_empty() {
        return CalcResult::Error {
            error: Error::ERROR,
            origin: cell,
            message: "Wrong number of arguments".to_string(),
        };
    }
    match self.current_excel_serial() {
        Some(serial) => CalcResult::Number(serial.floor()),
        None => CalcResult::Error {
            error: Error::ERROR,
            origin: cell,
            message: "Invalid date".to_string(),
        },
    }
}
```

### Helper Function: current_excel_serial

**File:** `base/src/functions/date_and_time.rs`
**Lines:** `753 - 763`

```rust
// Returns the current date/time as an Excel serial number in the model's configured timezone.
// Used by TODAY() and NOW().
fn current_excel_serial(&self) -> Option<f64> {
    let seconds = get_milliseconds_since_epoch() / 1000;
    DateTime::from_timestamp(seconds, 0).map(|dt| {
        let local_time = dt.with_timezone(&self.tz);
        let days_from_1900 = local_time.num_days_from_ce() - EXCEL_DATE_BASE;
        let fraction = (local_time.num_seconds_from_midnight() as f64) / (60.0 * 60.0 * 24.0);
        days_from_1900 as f64 + fraction
    })
}
```

### Related Constants

**File:** `base/src/functions/date_and_time.rs`
**Lines:** `8 - 9`

```rust
const SECONDS_PER_DAY: i32 = 86_400;
const SECONDS_PER_DAY_F64: f64 = SECONDS_PER_DAY as f64;
```

## 2. Unit Tests

### Test Case 1: Basic TODAY functionality

**File:** `base/src/test/test_today.rs`
**Lines:** `10 - 19`

```rust
#[test]
fn today_basic() {
    let mut model = new_empty_model();
    model._set("A1", "=TODAY()");
    model._set("A2", "=TEXT(A1, \"yyyy/m/d\")");
    model.evaluate();

    assert_eq!(model._get_text("A1"), *"08/11/2022");
    assert_eq!(model._get_text("A2"), *"2022/11/8");
}
```

### Test Case 2: Invalid timezone handling

**File:** `base/src/test/test_today.rs`
**Lines:** `21 - 25`

```rust
#[test]
fn today_with_wrong_tz() {
    let model = Model::new_empty("model", "en", "Wrong Timezone");
    assert!(model.is_err());
}
```

### Test Case 3: TODAY in UTC timezone

**File:** `base/src/test/test_today.rs`
**Lines:** `27 - 37`

```rust
#[test]
fn now_basic_utc() {
    mock_time::set_mock_time(TIMESTAMP_2023);
    let mut model = Model::new_empty("model", "en", "UTC").unwrap();
    model._set("A1", "=TODAY()");
    model._set("A2", "=NOW()");
    model.evaluate();

    assert_eq!(model._get_text("A1"), *"20/03/2023");
    assert_eq!(model._get_text("A2"), *"45005.572511574");
}
```

### Test Case 4: TODAY in Europe/Berlin timezone

**File:** `base/src/test/test_today.rs`
**Lines:** `39 - 50`

```rust
#[test]
fn now_basic_europe_berlin() {
    mock_time::set_mock_time(TIMESTAMP_2023);
    let mut model = Model::new_empty("model", "en", "Europe/Berlin").unwrap();
    model._set("A1", "=TODAY()");
    model._set("A2", "=NOW()");
    model.evaluate();

    assert_eq!(model._get_text("A1"), *"20/03/2023");
    // This is UTC + 1 hour: 45005.572511574 + 1/24
    assert_eq!(model._get_text("A2"), *"45005.614178241");
}
```

### Test Constant

**File:** `base/src/test/test_today.rs`
**Lines:** `7 - 8`

```rust
// 14:44 20 Mar 2023 Berlin
const TIMESTAMP_2023: i64 = 1679319865208;
```

## 3. Usage Examples

### Example 1: TODAY combined with YEAR function in CSV paste operation

**File:** `base/src/test/user_model/test_paste_csv.rs`
**Lines:** `30 - 50`

```rust
#[test]
fn csv_paste_formula() {
    let mut model = UserModel::new_empty("model", "en", "UTC").unwrap();

    let csv = "=YEAR(TODAY())";
    let area = Area {
        sheet: 0,
        row: 1,
        column: 1,
        width: 1,
        height: 1,
    };
    model.set_selected_cell(1, 1).unwrap();
    model.paste_csv_string(&area, csv).unwrap();

    assert_eq!(
        model.get_formatted_cell_value(0, 1, 1),
        Ok("2022".to_string())
    );
    assert_eq!([1, 1, 1, 1], model.get_selected_view().range);
}
```

## 4. Related Type Definitions

### Type: `CalcResult`

**File:** `base/src/calc_result.rs`
**Lines:** `12 - 28`

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
```

### Type: `Node`

**File:** `base/src/expressions/parser/mod.rs`
**Lines:** `105 - 202`

```rust
#[derive(PartialEq, Clone, Debug)]
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
    WrongRangeKind {
        sheet_name: Option<String>,
        absolute_row1: bool,
        absolute_column1: bool,
        row1: i32,
        column1: i32,
        absolute_row2: bool,
        absolute_column2: bool,
        row2: i32,
        column2: i32,
    },
    OpRangeKind {
        left: Box<Node>,
        right: Box<Node>,
    },
    OpConcatenateKind {
        left: Box<Node>,
        right: Box<Node>,
    },
    OpSumKind {
        kind: token::OpSum,
        left: Box<Node>,
        right: Box<Node>,
    },
    OpProductKind {
        kind: token::OpProduct,
        left: Box<Node>,
        right: Box<Node>,
    },
    OpPowerKind {
        left: Box<Node>,
        right: Box<Node>,
    },
    FunctionKind {
        kind: Function,
        args: Vec<Node>,
    },
    InvalidFunctionKind {
        name: String,
        args: Vec<Node>,
    },
    ArrayKind(Vec<Vec<ArrayNode>>),
    DefinedNameKind(DefinedNameS),
    TableNameKind(String),
    WrongVariableKind(String),
    ImplicitIntersection {
        automatic: bool,
        child: Box<Node>,
    },
    CompareKind {
        kind: OpCompare,
        left: Box<Node>,
        right: Box<Node>,
    },
    UnaryKind {
        kind: OpUnary,
        right: Box<Node>,
    },
    ErrorKind(token::Error),
    ParseErrorKind {
        formula: String,
        message: String,
        position: usize,
    },
    EmptyArgKind,
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

### Type: `Error`

**File:** `base/src/expressions/token.rs`
**Lines:** `77 - 96`

```rust
/// List of `errors`
/// Note that "#ERROR!" and "#N/IMPL!" are not part of the xlsx standard
///  * "#ERROR!" means there was an error processing the formula (for instance "=A1+")
///  * "#N/IMPL!" means the formula or feature in Excel but has not been implemented in IronCalc
///    Note that they are serialized/deserialized by index
#[derive(Serialize, Deserialize, Encode, Decode, Debug, PartialEq, Eq, Clone)]
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

## 5. Function Registration

### Function Name Mapping (get_function)

**File:** `base/src/functions/mod.rs`
**Lines:** `805 - 806`

```rust
"TODAY" => Some(Function::Today),
"NOW" => Some(Function::Now),
```

### Function Evaluation (evaluate_function)

**File:** `base/src/functions/mod.rs`
**Lines:** `1322 - 1323`

```rust
Function::Today => self.fn_today(args, cell),
Function::Now => self.fn_now(args, cell),
```

### Function Enum Variant

**File:** `base/src/functions/mod.rs`
**Lines:** `206 - 207`

```rust
Today,
Now,
```

### Function Iterator Registration

**File:** `base/src/functions/mod.rs`
**Lines:** `467 - 468`

```rust
Function::Today,
Function::Now,
```

### Function Display Name

**File:** `base/src/functions/mod.rs`
**Lines:** `1045 - 1046`

```rust
Function::Today => write!(f, "TODAY"),
Function::Now => write!(f, "NOW"),
```

## 6. Additional Context

### Import Dependencies

**File:** `base/src/functions/date_and_time.rs`
**Lines:** `1 - 77`

The TODAY function relies on these key dependencies:

```rust
use chrono::DateTime;
use chrono::Datelike;

use crate::constants::EXCEL_DATE_BASE;
use crate::expressions::types::CellReferenceIndex;
use crate::expressions::parser::Node;
use crate::expressions::token::Error;
use crate::model::get_milliseconds_since_epoch;
use crate::calc_result::CalcResult;
use crate::model::Model;
```

### Key Implementation Notes

1. **Excel Serial Number**: The TODAY function returns an Excel serial number, which is the number of days since January 1, 1900 (with the integer part representing the date).

2. **Timezone-Aware**: The function respects the model's configured timezone (stored in `self.tz`), using the `chrono` library's timezone support.

3. **Floor Operation**: The result is floored to remove the time component, returning only the date portion (unlike NOW() which includes the time).

4. **Error Handling**: The function validates that it receives no arguments and handles timezone conversion failures gracefully.

5. **Mock Time Support**: For testing, the implementation uses `get_milliseconds_since_epoch()` which can be mocked for deterministic test results.

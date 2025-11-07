# Code Context Report for: `DAY`

This report contains all relevant code snippets for the `DAY` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `base/src/functions/date_and_time.rs`
**Lines:** `18 - 35` (Macro definition) and `494` (Function invocation)

```rust
// Generate DAY / MONTH / YEAR helpers – simply convert the serial number to a
// NaiveDate and return the requested component as a number.
macro_rules! date_part_fn {
    ($name:ident, $method:ident) => {
        pub(crate) fn $name(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
            if args.len() != 1 {
                return CalcResult::new_args_number_error(cell);
            }
            let serial_number = match self.get_number(&args[0], cell) {
                Ok(num) => num.floor() as i64,
                Err(e) => return e,
            };
            let date = match self.excel_date(serial_number, cell) {
                Ok(d) => d,
                Err(e) => return e,
            };
            CalcResult::Number(date.$method() as f64)
        }
    };
}

// ... (lines 36-493 omitted)

// Auto-generated DATE part helpers (DAY / MONTH / YEAR)
date_part_fn!(fn_day, day);
```

**Note:** The `DAY` function is implemented using the `date_part_fn!` macro, which generates a function that:
1. Validates exactly 1 argument is provided
2. Extracts the serial number from the argument and floors it to an integer
3. Converts the serial number to a `chrono::NaiveDate` using `excel_date()`
4. Calls the `day()` method on the date object to extract the day component
5. Returns the day as a `CalcResult::Number`

## 2. Unit Tests

### Test Case 1: Wrong number of arguments

**File:** `base/src/test/test_date_and_time.rs`
**Lines:** `115 - 126`

```rust
#[test]
fn test_day_arguments() {
    let mut model = new_empty_model();
    model._set("A1", "=DAY()");
    model._set("A2", "=DAY(27819)");
    model._set("A3", "=DAY(27819, 3)");

    model.evaluate();

    assert_eq!(model._get_text("A1"), *"#ERROR!");
    assert_eq!(model._get_text("A2"), *"29");
    assert_eq!(model._get_text("A3"), *"#ERROR!");
}
```

### Test Case 2: Small serial numbers and boundary cases

**File:** `base/src/test/test_date_and_time.rs`
**Lines:** `128 - 146`

```rust
#[test]
fn test_day_small_serial() {
    let mut model = new_empty_model();
    model._set("A1", "=DAY(-1)");
    model._set("A2", "=DAY(0)");
    model._set("A3", "=DAY(60)");

    model._set("A4", "=DAY(61)");

    model.evaluate();

    assert_eq!(model._get_text("A1"), *"#NUM!");
    assert_eq!(model._get_text("A2"), *"#NUM!");
    // Excel thinks is Feb 29, 1900
    assert_eq!(model._get_text("A3"), *"28");

    // From now on everyone agrees
    assert_eq!(model._get_text("A4"), *"1");
}
```

### Test Case 3: Out of range dates

**File:** `base/src/test/test_fn_day.rs`
**Lines:** `6 - 15`

```rust
#[test]
fn test_fn_date_arguments() {
    let mut model = new_empty_model();

    model._set("A1", "=DAY(95051806)");
    model._set("A2", "=DAY(2958465)");
    model.evaluate();

    assert_eq!(model._get_text("A1"), *"#NUM!");
    assert_eq!(model._get_text("A2"), *"31");
}
```

**Note:** Test shows that serial number 2958465 (Dec 31, 9999) is the maximum valid date. Serial number 95051806 is out of range and returns `#NUM!` error.

## 3. Usage Examples

### Example 1: Documentation style guide examples

**File:** `docs/src/functions/new_documentation/style_guide.md`
**Lines:** `103 - 110`

```markdown
    =DAY(45321)

[... some documentation text ...]

    =DAY(TODAY())
```

### Example 2: Existing DAY documentation

**File:** `docs/src/functions/date_and_time/day.md`
**Lines:** `27`

```markdown
* At present, DAY does not accept a string representation of a date literal as an argument. For example, the formula `=DAY("2024-12-31")` returns the [`#VALUE!`](/features/error-types.md#value) error.
```

**Note:** This indicates a known limitation where string date inputs are not currently supported by the DAY function.

### Example 3: Test suite usage showing basic functionality

**File:** `base/src/test/test_date_and_time.rs`
**Lines:** `118`

```rust
model._set("A2", "=DAY(27819)");
// Result: "29" (Feb 29, 1976)
```

## 4. Related Type Definitions

### Type: `Node`

**File:** `base/src/expressions/parser/mod.rs`
**Lines:** `106 - 145` (partial)

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
    WrongRangeKind {
        sheet_name: Option<String>,
        absolute_row1: bool,
        absolute_column1: bool,
        row1: i32,
        column1: i32,
        absolute_row2: bool,
        absolute_column2: bool,
        row2: i32,
    // ... (additional variants)
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
**Lines:** `11 - 49` (partial)

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

### Type: `Function` (Enum Variant)

**File:** `base/src/functions/mod.rs`
**Lines:** `192 - 217` (Date and time section)

```rust
    // Date and time
    Date,
    Datedif,
    Datevalue,
    Day,
    Edate,
    Eomonth,
    Month,
    Time,
    Timevalue,
    Hour,
    Minute,
    Second,
    Now,
    Today,
    Year,
    Networkdays,
    NetworkdaysIntl,
    Days,
    Days360,
    Weekday,
    Weeknum,
    Workday,
    WorkdayIntl,
    Yearfrac,
    Isoweeknum,
```

## 5. Function Registration

### String to Function Enum Mapping

**File:** `base/src/functions/mod.rs`
**Lines:** `791`

```rust
"DAY" => Some(Function::Day),
```

### Function Enum to String Display

**File:** `base/src/functions/mod.rs`
**Lines:** `1031`

```rust
Function::Day => write!(f, "DAY"),
```

### Function Dispatch/Execution

**File:** `base/src/functions/mod.rs`
**Lines:** `1308`

```rust
Function::Day => self.fn_day(args, cell),
```

---

## Additional Context

### Helper Functions Referenced by DAY

The `DAY` function implementation relies on these helper methods:

1. **`self.get_number()`**: Extracts a numeric value from a `Node`, handling type conversions
2. **`self.excel_date()`**: Converts an Excel serial number to a `chrono::NaiveDate`

**File:** `base/src/functions/date_and_time.rs`
**Lines:** `808 - 821`

```rust
fn excel_date(
    &self,
    serial: i64,
    cell: CellReferenceIndex,
) -> Result<chrono::NaiveDate, CalcResult> {
    match from_excel_date(serial) {
        Ok(date) => Ok(date),
        Err(_) => Err(CalcResult::Error {
            error: Error::NUM,
            origin: cell,
            message: "Out of range parameters for date".to_string(),
        }),
    }
}
```

### Date Serial Number Constants

**File:** `base/src/test/test_date_and_time.rs`
**Lines:** `11 - 12`

```rust
const EXCEL_MAX_DATE: f64 = 2958465.0; // Dec 31, 9999 - used in boundary tests
const EXCEL_INVALID_DATE: f64 = 2958466.0; // One day past max - used in error tests
```

### Key Implementation Notes

1. **Excel 1900 Leap Year Bug**: The codebase handles the infamous Excel bug where Feb 29, 1900 is treated as valid (serial number 60). IronCalc deviates from Excel here - see test comments at line 141-142 in `test_date_and_time.rs`.

2. **Valid Date Range**: Serial numbers must be >= 1 and <= 2958465 (Dec 31, 9999).

3. **Argument Count Validation**: The function strictly requires exactly 1 argument; 0 or 2+ arguments return `#ERROR!`.

4. **Return Type**: Always returns a numeric value (1-31) representing the day of the month, or an error.

5. **Serial Number Handling**: The input serial number is floored to an integer before processing, discarding any time component.

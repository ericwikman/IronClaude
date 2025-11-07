# Code Context Report for: `TIMEVALUE`

This report contains all relevant code snippets for the `TIMEVALUE` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `/home/eric/projects/IronCalc/base/src/functions/date_and_time.rs`
**Lines:** `1031-1047`

```rust
pub(crate) fn fn_timevalue(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.len() != 1 {
        return CalcResult::new_args_number_error(cell);
    }
    let text = match self.get_string(&args[0], cell) {
        Ok(s) => s,
        Err(e) => return e,
    };
    match parse_time_string(&text) {
        Some(value) => CalcResult::Number(value),
        None => CalcResult::Error {
            error: Error::VALUE,
            origin: cell,
            message: "Invalid time".to_string(),
        },
    }
}
```

**Key Details:**
- Validates that exactly one argument is provided
- Converts the argument to a string using `get_string()`
- Parses the time string using the helper function `parse_time_string()` (defined at lines 141-210)
- Returns a `CalcResult::Number` with the parsed time value as a fraction of a day (0.0-1.0)
- Returns a `VALUE` error if parsing fails

### Helper Function: `parse_time_string`

**File:** `/home/eric/projects/IronCalc/base/src/functions/date_and_time.rs`
**Lines:** `141-210`

```rust
fn parse_time_string(text: &str) -> Option<f64> {
    let text = text.trim();

    // First, try custom parsing for edge cases like "24:00:00", "23:60:00", "23:59:60"
    // that need normalization to match Excel behavior
    if let Some(time_fraction) = parse_time_with_normalization(text) {
        return Some(time_fraction);
    }

    // First, try manual parsing for simple "N PM" / "N AM" format
    if let Some((hour_str, is_pm)) = parse_simple_am_pm(text) {
        if let Ok(hour) = hour_str.parse::<u32>() {
            if (1..=12).contains(&hour) {
                let hour_24 = if is_pm {
                    if hour == 12 {
                        12
                    } else {
                        hour + 12
                    }
                } else if hour == 12 {
                    0
                } else {
                    hour
                };
                let time = NaiveTime::from_hms_opt(hour_24, 0, 0)?;
                return Some(time.num_seconds_from_midnight() as f64 / SECONDS_PER_DAY_F64);
            }
        }
    }

    // Standard patterns
    let patterns_time = ["%H:%M:%S", "%H:%M", "%I:%M %p", "%I %p", "%I:%M:%S %p"];
    for p in patterns_time {
        if let Ok(t) = NaiveTime::parse_from_str(text, p) {
            return Some(t.num_seconds_from_midnight() as f64 / SECONDS_PER_DAY_F64);
        }
    }

    let patterns_dt = [
        // ISO formats
        "%Y-%m-%d %H:%M:%S",
        "%Y-%m-%d %H:%M",
        "%Y-%m-%dT%H:%M:%S",
        "%Y-%m-%dT%H:%M",
        // Excel-style date formats with AM/PM
        "%d-%b-%Y %I:%M:%S %p", // "22-Aug-2011 6:35:00 AM"
        "%d-%b-%Y %I:%M %p",    // "22-Aug-2011 6:35 AM"
        "%d-%b-%Y %H:%M:%S",    // "22-Aug-2011 06:35:00"
        "%d-%b-%Y %H:%M",       // "22-Aug-2011 06:35"
        // US date formats with AM/PM
        "%m/%d/%Y %I:%M:%S %p", // "8/22/2011 6:35:00 AM"
        "%m/%d/%Y %I:%M %p",    // "8/22/2011 6:35 AM"
        "%m/%d/%Y %H:%M:%S",    // "8/22/2011 06:35:00"
        "%m/%d/%Y %H:%M",       // "8/22/2011 06:35"
        // European date formats with AM/PM
        "%d/%m/%Y %I:%M:%S %p", // "22/8/2011 6:35:00 AM"
        "%d/%m/%Y %I:%M %p",    // "22/8/2011 6:35 AM"
        "%d/%m/%Y %H:%M:%S",    // "22/8/2011 06:35:00"
        "%d/%m/%Y %H:%M",       // "22/8/2011 06:35"
    ];
    for p in patterns_dt {
        if let Ok(dt) = NaiveDateTime::parse_from_str(text, p) {
            return Some(dt.time().num_seconds_from_midnight() as f64 / SECONDS_PER_DAY_F64);
        }
    }
    if let Ok(dt) = DateTime::parse_from_rfc3339(text) {
        return Some(dt.time().num_seconds_from_midnight() as f64 / SECONDS_PER_DAY_F64);
    }
    None
}
```

**Supported Time Formats:**
1. **Time-only formats (24-hour):** `HH:MM:SS`, `HH:MM`
2. **Time-only formats (12-hour):** `I:MM:SS PM/AM`, `I PM/AM`, `I:MM PM/AM`
3. **Date-time formats (ISO 8601):** `YYYY-MM-DD HH:MM:SS`, `YYYY-MM-DDTHH:MM:SS`
4. **Date-time formats (Excel-style):** `DD-Mon-YYYY HH:MM:SS AM/PM` (e.g., "22-Aug-2011 6:35 AM")
5. **Date-time formats (US):** `M/DD/YYYY HH:MM:SS AM/PM`
6. **Date-time formats (European):** `DD/M/YYYY HH:MM:SS AM/PM`
7. **RFC 3339 format:** ISO 8601 with timezone information

**Special Handling:**
- When a datetime string is provided, only the time component is extracted and converted
- Text is trimmed of leading/trailing whitespace
- Handles edge cases like "24:00:00", "23:60:00", "23:59:60" with normalization to match Excel behavior
- Single-digit hour values with AM/PM are supported (e.g., "2 PM" → 14:00)

---

## 2. Unit Tests

### Test Case 1: Excel TIMEVALUE Compatibility

**File:** `/home/eric/projects/IronCalc/base/src/test/test_fn_time.rs`
**Lines:** `50-93`

```rust
#[test]
fn test_excel_timevalue_compatibility() {
    // Test cases based on Excel's official documentation and examples
    let model = test_time_expressions(&[
        // Excel documentation examples
        ("A1", "=TIMEVALUE(\"2:24 AM\")"), // Should be 0.1
        ("A2", "=TIMEVALUE(\"2 PM\")"),    // Should be 0.583333... (14/24)
        ("A3", "=TIMEVALUE(\"6:45 PM\")"), // Should be 0.78125 (18.75/24)
        ("A4", "=TIMEVALUE(\"18:45\")"),   // Same as above, 24-hour format
        // Date-time format (date should be ignored)
        ("B1", "=TIMEVALUE(\"22-Aug-2011 6:35 AM\")"), // Should be ~0.2743
        ("B2", "=TIMEVALUE(\"2023-01-01 14:30:00\")"), // Should be 0.604166667
        // Edge cases that Excel should support
        ("C1", "=TIMEVALUE(\"12:00 AM\")"),    // Midnight: 0
        ("C2", "=TIMEVALUE(\"12:00 PM\")"),    // Noon: 0.5
        ("C3", "=TIMEVALUE(\"11:59:59 PM\")"), // Almost midnight: 0.999988426
        // Single digit variations
        ("D1", "=TIMEVALUE(\"1 AM\")"),  // 1:00 AM
        ("D2", "=TIMEVALUE(\"9 PM\")"),  // 9:00 PM
        ("D3", "=TIMEVALUE(\"12 AM\")"), // Midnight
        ("D4", "=TIMEVALUE(\"12 PM\")"), // Noon
    ]);

    // Excel documentation examples - verify exact values
    assert_eq!(model._get_text("A1"), *TIME_2_24_AM); // 2:24 AM
    assert_eq!(model._get_text("A2"), *TIME_2_PM); // 2 PM = 14:00
    assert_eq!(model._get_text("A3"), *TIME_6_45_PM); // 6:45 PM = 18:45
    assert_eq!(model._get_text("A4"), *TIME_6_45_PM); // 18:45 (24-hour)

    // Date-time formats (date ignored, extract time only)
    assert_eq!(model._get_text("B1"), *TIME_6_35_AM); // 6:35 AM ≈ 0.2743
    assert_eq!(model._get_text("B2"), *TIME_14_30); // 14:30:00

    // Edge cases
    assert_eq!(model._get_text("C1"), *MIDNIGHT); // 12:00 AM = 00:00
    assert_eq!(model._get_text("C2"), *NOON); // 12:00 PM = 12:00
    assert_eq!(model._get_text("C3"), *TIME_23_59_59); // 11:59:59 PM

    // Single digit hours
    assert_eq!(model._get_text("D1"), *TIME_1_AM); // 1:00 AM
    assert_eq!(model._get_text("D2"), *TIME_9_PM); // 9:00 PM = 21:00
    assert_eq!(model._get_text("D3"), *MIDNIGHT); // 12 AM = 00:00
    assert_eq!(model._get_text("D4"), *NOON); // 12 PM = 12:00
}
```

**Purpose:** Tests Excel documentation examples and edge cases for basic time format compatibility.

### Test Case 2: Various Time Formats

**File:** `/home/eric/projects/IronCalc/base/src/test/test_fn_time.rs`
**Lines:** `185-227`

```rust
#[test]
fn test_timevalue_function_formats() {
    let model = test_time_expressions(&[
        // Basic formats
        ("A1", "=TIMEVALUE(\"14:30\")"),
        ("A2", "=TIMEVALUE(\"14:30:45\")"),
        ("A3", "=TIMEVALUE(\"00:00:00\")"),
        // AM/PM formats
        ("B1", "=TIMEVALUE(\"2:30 PM\")"),
        ("B2", "=TIMEVALUE(\"2:30 AM\")"),
        ("B3", "=TIMEVALUE(\"12:00 PM\")"), // Noon
        ("B4", "=TIMEVALUE(\"12:00 AM\")"), // Midnight
        // Single hour with AM/PM (now supported!)
        ("B5", "=TIMEVALUE(\"2 PM\")"),
        ("B6", "=TIMEVALUE(\"2 AM\")"),
        // Date-time formats (extract time only)
        ("C1", "=TIMEVALUE(\"2023-01-01 14:30:00\")"),
        ("C2", "=TIMEVALUE(\"2023-01-01T14:30:00\")"),
        // Whitespace handling
        ("D1", "=TIMEVALUE(\" 14:30 \")"),
    ]);

    // Basic formats
    assert_eq!(model._get_text("A1"), *TIME_14_30);
    assert_eq!(model._get_text("A2"), *TIME_14_30_45);
    assert_eq!(model._get_text("A3"), *MIDNIGHT);

    // AM/PM formats
    assert_eq!(model._get_text("B1"), *TIME_14_30); // 2:30 PM = 14:30
    assert_eq!(model._get_text("B2"), *TIME_2_30_AM); // 2:30 AM
    assert_eq!(model._get_text("B3"), *NOON); // 12:00 PM = noon
    assert_eq!(model._get_text("B4"), *MIDNIGHT); // 12:00 AM = midnight

    // Single hour AM/PM formats (now supported!)
    assert_eq!(model._get_text("B5"), *TIME_2_PM); // 2 PM = 14:00
    assert_eq!(model._get_text("B6"), *TIME_2_AM); // 2 AM = 02:00

    // Date-time formats
    assert_eq!(model._get_text("C1"), *TIME_14_30);
    assert_eq!(model._get_text("C2"), *TIME_14_30);

    // Whitespace
    assert_eq!(model._get_text("D1"), *TIME_14_30);
}
```

**Purpose:** Tests various time format variations including 24-hour, 12-hour with AM/PM, date-time combinations, and whitespace handling.

### Test Case 3: Error Handling

**File:** `/home/eric/projects/IronCalc/base/src/test/test_fn_time.rs`
**Lines:** `230-251`

```rust
#[test]
fn test_timevalue_function_errors() {
    let model = test_time_expressions(&[
        ("A1", "=TIMEVALUE()"),                 // Wrong arg count
        ("A2", "=TIMEVALUE(\"14:30\", \"x\")"), // Wrong arg count
        ("B1", "=TIMEVALUE(\"invalid\")"),      // Invalid format
        ("B2", "=TIMEVALUE(\"25:00\")"),        // Invalid hour
        ("B3", "=TIMEVALUE(\"14:70\")"),        // Invalid minute
        ("B4", "=TIMEVALUE(\"\")"),             // Empty string
        ("B5", "=TIMEVALUE(\"2PM\")"),          // Missing space (still unsupported)
    ]);

    // Wrong argument count should return #ERROR!
    assert_eq!(model._get_text("A1"), *"#ERROR!");
    assert_eq!(model._get_text("A2"), *"#ERROR!");

    // Invalid formats should return #VALUE!
    assert_eq!(model._get_text("B1"), *"#VALUE!");
    assert_eq!(model._get_text("B2"), *"#VALUE!");
    assert_eq!(model._get_text("B3"), *"#VALUE!");
    assert_eq!(model._get_text("B4"), *"#VALUE!");
    assert_eq!(model._get_text("B5"), *"#VALUE!"); // "2PM" no space - not supported
}
```

**Purpose:** Tests error conditions including incorrect argument counts, invalid time formats, and edge cases that should produce errors.

### Test Case 4: Argument Validation

**File:** `/home/eric/projects/IronCalc/base/src/test/test_fn_datevalue_timevalue.rs`
**Lines:** `5-22`

```rust
#[test]
fn datevalue_timevalue_arguments() {
    let mut model = new_empty_model();
    model._set("A1", "=DATEVALUE()");
    model._set("A2", "=TIMEVALUE()");
    model._set("A3", "=DATEVALUE("2000-01-01")")
    model._set("A4", "=TIMEVALUE("12:00:00")")
    model._set("A5", "=DATEVALUE(1,2)");
    model._set("A6", "=TIMEVALUE(1,2)");
    model.evaluate();

    assert_eq!(model._get_text("A1"), *"#ERROR!");
    assert_eq!(model._get_text("A2"), *"#ERROR!");
    assert_eq!(model._get_text("A3"), *"36526");
    assert_eq!(model._get_text("A4"), *"0.5");
    assert_eq!(model._get_text("A5"), *"#ERROR!");
    assert_eq!(model._get_text("A6"), *"#ERROR!");
}
```

**Purpose:** Tests argument count validation - TIMEVALUE requires exactly one argument and returns `#ERROR!` otherwise.

---

## 3. Usage Examples

### Example 1: Basic Time Format Conversion

**File:** `/home/eric/projects/IronCalc/base/src/test/test_fn_time.rs`
**Lines:** `53-58`

```rust
// Excel documentation examples
("A1", "=TIMEVALUE(\"2:24 AM\")"), // Should be 0.1
("A2", "=TIMEVALUE(\"2 PM\")"),    // Should be 0.583333... (14/24)
("A3", "=TIMEVALUE(\"6:45 PM\")"), // Should be 0.78125 (18.75/24)
("A4", "=TIMEVALUE(\"18:45\")"),   // Same as above, 24-hour format
```

**Context:** Standard Excel documentation examples demonstrating basic TIMEVALUE usage with both 12-hour and 24-hour time formats.

**Expected Results:**
- `TIMEVALUE("2:24 AM")` returns `0.1` (2 hours 24 minutes as fraction of 24 hours)
- `TIMEVALUE("2 PM")` returns `0.583333...` (14 hours as fraction of 24 hours)
- `TIMEVALUE("6:45 PM")` returns `0.78125` (18.75 hours as fraction of 24 hours)

### Example 2: DateTime Format with Embedded Date

**File:** `/home/eric/projects/IronCalc/base/src/test/test_fn_time.rs`
**Lines:** `59-62`

```rust
// Date-time format (date should be ignored)
("B1", "=TIMEVALUE(\"22-Aug-2011 6:35 AM\")"), // Should be ~0.2743
("B2", "=TIMEVALUE(\"2023-01-01 14:30:00\")"), // Should be 0.604166667
```

**Context:** Demonstrates that TIMEVALUE extracts only the time portion from datetime strings, ignoring the date component.

**Expected Results:**
- `TIMEVALUE("22-Aug-2011 6:35 AM")` returns `~0.274305556` (6:35 AM time only)
- `TIMEVALUE("2023-01-01 14:30:00")` returns `0.604166667` (14:30:00 time only)

### Example 3: Midnight and Noon Edge Cases

**File:** `/home/eric/projects/IronCalc/base/src/test/test_fn_time.rs`
**Lines:** `63-65`

```rust
// Edge cases that Excel should support
("C1", "=TIMEVALUE(\"12:00 AM\")"),    // Midnight: 0
("C2", "=TIMEVALUE(\"12:00 PM\")"),    // Noon: 0.5
("C3", "=TIMEVALUE(\"11:59:59 PM\")"), // Almost midnight: 0.999988426
```

**Context:** Special handling required for 12-hour format where 12 AM = midnight (0.0) and 12 PM = noon (0.5).

**Expected Results:**
- `TIMEVALUE("12:00 AM")` returns `0` (midnight)
- `TIMEVALUE("12:00 PM")` returns `0.5` (noon)
- `TIMEVALUE("11:59:59 PM")` returns `0.999988426` (almost midnight)

---

## 4. Related Type Definitions

### Type: `Node`

**File:** `/home/eric/projects/IronCalc/base/src/expressions/parser/mod.rs`
**Lines:** `106-202`

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

**Description:** Represents a parsed expression tree node. The `Node::StringKind(String)` variant is used to pass the time string argument to TIMEVALUE.

### Type: `CalcResult`

**File:** `/home/eric/projects/IronCalc/base/src/calc_result.rs`
**Lines:** `11-28`

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

**Description:** Represents the result of a calculation. TIMEVALUE returns either `CalcResult::Number(f64)` for successful parsing (with a value between 0.0 and 1.0 representing a fraction of a 24-hour day), or `CalcResult::Error` for invalid inputs.

### Type: `CellReferenceIndex`

**File:** `/home/eric/projects/IronCalc/base/src/expressions/types.rs`
**Lines:** `37-42`

```rust
#[derive(Serialize, Deserialize, Debug, Clone, PartialEq, Eq, Copy)]
pub struct CellReferenceIndex {
    pub sheet: u32,
    pub column: i32,
    pub row: i32,
}
```

**Description:** Represents a cell reference with sheet index, column, and row. Used to track the origin of the function call for error reporting.

### Type: `Error`

**File:** `/home/eric/projects/IronCalc/base/src/expressions/token.rs`
**Lines:** `84-97`

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

**Description:** Represents spreadsheet error types. TIMEVALUE returns `Error::VALUE` when the input string cannot be parsed as a valid time, or `Error::ERROR` when the argument count is incorrect.

---

## 5. Function Registration

### Function Enum Declaration

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `200-201`

```rust
    Time,
    Timevalue,
```

**Description:** TIMEVALUE is declared as a variant in the `Function` enum, making it available as a recognized function in the system.

### Function String Mapping

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `800-801`

```rust
            "TIME" => Some(Function::Time),
            "TIMEVALUE" => Some(Function::Timevalue),
```

**Description:** Maps the string literal `"TIMEVALUE"` to the `Function::Timevalue` enum variant in the `get_function()` method. This allows spreadsheet formulas to recognize `=TIMEVALUE(...)` as a valid function call.

### Function Evaluation

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `1318`

```rust
            Function::Timevalue => self.fn_timevalue(args, cell),
```

**Description:** In the `evaluate_function()` method, when a `Function::Timevalue` variant is encountered, it delegates execution to the `fn_timevalue()` implementation in the date_and_time module.

---

## Summary

The TIMEVALUE function:
- **Takes:** One required string argument representing a time in various formats
- **Returns:** A decimal number (0.0-1.0) representing the fraction of a 24-hour day, or a `#VALUE!` error for invalid inputs
- **Key Features:**
  - Supports 12-hour and 24-hour time formats
  - Handles AM/PM notation correctly (including edge cases like "12 AM" = midnight)
  - Extracts time from datetime strings, ignoring the date component
  - Supports multiple international date-time formats
  - Trims whitespace from input strings
  - Validates argument count (exactly 1 required)
- **Error Cases:**
  - `#ERROR!` when argument count is not exactly 1
  - `#VALUE!` when the time string cannot be parsed as a valid time

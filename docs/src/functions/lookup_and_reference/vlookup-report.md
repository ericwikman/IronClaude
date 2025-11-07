# Code Context Report for: `VLOOKUP`

This report contains all relevant code snippets for the `VLOOKUP` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `base/src/functions/lookup_and_reference.rs`
**Lines:** `398 - 503`

```rust
/// VLOOKUP(lookup_value, table_array, row_index, [is_sorted])
/// We look for `lookup_value` in the first column of table array
/// We return the value in column `column_index` of the same row in `table_array`
/// `is_sorted` is true by default and assumes that values in first column are ordered
pub(crate) fn fn_vlookup(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.len() > 4 || args.len() < 3 {
        return CalcResult::new_args_number_error(cell);
    }
    let lookup_value = self.evaluate_node_in_context(&args[0], cell);
    if lookup_value.is_error() {
        return lookup_value;
    }
    let column_index = match self.get_number(&args[2], cell) {
        Ok(v) => v.floor() as i32,
        Err(s) => return s,
    };
    let is_sorted = if args.len() == 4 {
        match self.get_boolean(&args[3], cell) {
            Ok(v) => v,
            Err(s) => return s,
        }
    } else {
        true
    };
    let range = self.evaluate_node_in_context(&args[1], cell);
    match range {
        CalcResult::Range { left, right } => {
            if is_sorted {
                // This assumes the values in column are in order
                let l = self.binary_search(&lookup_value, &left, &right, true);
                if l == -2 {
                    return CalcResult::Error {
                        error: Error::NA,
                        origin: cell,
                        message: "Not found".to_string(),
                    };
                }
                let row = left.row + l;
                let column = left.column + column_index - 1;
                if column > right.column {
                    return CalcResult::Error {
                        error: Error::REF,
                        origin: cell,
                        message: "Invalid reference".to_string(),
                    };
                }
                self.evaluate_cell(CellReferenceIndex {
                    sheet: left.sheet,
                    row,
                    column,
                })
            } else {
                // Linear search for exact match
                let n = right.row - left.row + 1;
                let column = left.column + column_index - 1;
                if column > right.column {
                    return CalcResult::Error {
                        error: Error::REF,
                        origin: cell,
                        message: "Invalid reference".to_string(),
                    };
                }
                let result_matches: Box<dyn Fn(&CalcResult) -> bool> =
                    if let CalcResult::String(s) = &lookup_value {
                        if let Ok(reg) = from_wildcard_to_regex(&s.to_lowercase(), true) {
                            Box::new(move |x| result_matches_regex(x, &reg))
                        } else {
                            Box::new(move |_| false)
                        }
                    } else {
                        Box::new(move |x| compare_values(x, &lookup_value) == 0)
                    };
                for l in 0..n {
                    let value = self.evaluate_cell(CellReferenceIndex {
                        sheet: left.sheet,
                        row: left.row + l,
                        column: left.column,
                    });
                    if result_matches(&value) {
                        return self.evaluate_cell(CellReferenceIndex {
                            sheet: left.sheet,
                            row: left.row + l,
                            column,
                        });
                    }
                }
                CalcResult::Error {
                    error: Error::NA,
                    origin: cell,
                    message: "Not found".to_string(),
                }
            }
        }
        error @ CalcResult::Error { .. } => error,
        CalcResult::String(_) => CalcResult::Error {
            error: Error::VALUE,
            origin: cell,
            message: "Range expected".to_string(),
        },
        _ => CalcResult::Error {
            error: Error::NA,
            origin: cell,
            message: "Range expected".to_string(),
        },
    }
}
```

## 2. Unit Tests

No results found for this section.

The codebase does not contain specific unit tests for the VLOOKUP function. The function appears to be tested through integration tests with Excel workbooks, but no direct unit tests were found in the test files.

## 3. Usage Examples

No results found for this section.

No direct usage examples of VLOOKUP were found in the codebase outside of the implementation itself and documentation references. The function is available for use but examples are not included in the source code.

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
**Lines:** `84 - 97`

```rust
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

### Type: `Node` (partial)

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
    // ... (additional variants exist but are omitted for brevity)
}
```

## 5. Function Registration

### Registration in Function Enum

**File:** `base/src/functions/mod.rs`
**Lines:** `151`

```rust
// Within the Function enum definition
Vlookup,
```

### Registration in String-to-Function Mapping

**File:** `base/src/functions/mod.rs`
**Lines:** `732`

```rust
"VLOOKUP" => Some(Function::Vlookup),
```

### Registration in Display Implementation

**File:** `base/src/functions/mod.rs`
**Lines:** `982`

```rust
Function::Vlookup => write!(f, "VLOOKUP"),
```

### Registration in Function Evaluation Dispatcher

**File:** `base/src/functions/mod.rs`
**Lines:** `1259`

```rust
Function::Vlookup => self.fn_vlookup(args, cell),
```

### Registration in Function Iterator

**File:** `base/src/functions/mod.rs`
**Lines:** `404`

```rust
// Within Function::into_iter() array
Function::Vlookup,
```

## 6. Helper Functions and Utilities

### Binary Search Implementation

**File:** `base/src/functions/binary_search.rs`
**Lines:** `174 - 207`

```rust
/// Old style binary search. Used in HLOOKUP, etc
pub(crate) fn binary_search(
    &mut self,
    target: &CalcResult,
    left: &CellReferenceIndex,
    right: &CellReferenceIndex,
    is_row_vector: bool,
) -> i32 {
    let array = self.prepare_array(left, right, is_row_vector);
    // We apply binary search leftmost for value in the range
    let mut l = 0;
    let mut r = array.len();
    while l < r {
        let m = (l + r) / 2;
        match compare_values(&array[m], target) {
            -1 => {
                l = m + 1;
            }
            1 => {
                r = m;
            }
            _ => {
                return m as i32;
            }
        }
    }
    // If target is less than the minimum return #N/A
    if l == 0 {
        return -2;
    }
    // Now l points to the leftmost element
    (l - 1) as i32
}
```

### Compare Values Utility

**File:** `base/src/functions/util.rs`
**Lines:** `34 - 82`

```rust
// ..., -2, -1, 0, 1, 2, ..., A-Z, FALSE, TRUE;
pub(crate) fn compare_values(left: &CalcResult, right: &CalcResult) -> i32 {
    match (left, right) {
        (CalcResult::Number(value1), CalcResult::Number(value2)) => {
            if (value2 - value1).abs() < f64::EPSILON {
                return 0;
            }
            if value1 < value2 {
                return -1;
            }
            1
        }
        (CalcResult::Number(_value1), CalcResult::String(_value2)) => -1,
        (CalcResult::Number(_value1), CalcResult::Boolean(_value2)) => -1,
        (CalcResult::String(value1), CalcResult::String(value2)) => {
            let value1 = value1.to_uppercase();
            let value2 = value2.to_uppercase();
            match value1.cmp(&value2) {
                std::cmp::Ordering::Less => -1,
                std::cmp::Ordering::Equal => 0,
                std::cmp::Ordering::Greater => 1,
            }
        }
        (CalcResult::String(_value1), CalcResult::Boolean(_value2)) => -1,
        (CalcResult::Boolean(value1), CalcResult::Boolean(value2)) => {
            if value1 == value2 {
                return 0;
            }
            if *value1 {
                return 1;
            }
            -1
        }
        (CalcResult::EmptyCell, CalcResult::String(_value2)) => {
            compare_values(&CalcResult::String("".to_string()), right)
        }
        (CalcResult::String(_value1), CalcResult::EmptyCell) => {
            compare_values(left, &CalcResult::String("".to_string()))
        }
        (CalcResult::EmptyCell, CalcResult::Number(_value2)) => {
            compare_values(&CalcResult::Number(0.0), right)
        }
        (CalcResult::Number(_value1), CalcResult::EmptyCell) => {
            compare_values(left, &CalcResult::Number(0.0))
        }
        (CalcResult::EmptyCell, CalcResult::EmptyCell) => 0,
        // NOTE: Errors and Ranges are not covered
        (_, _) => 1,
    }
}
```

### Wildcard to Regex Conversion

**File:** `base/src/functions/util.rs`
**Lines:** `84 - 116`

```rust
/// We convert an Excel wildcard into a Rust (Perl family) regex
pub(crate) fn from_wildcard_to_regex(
    wildcard: &str,
    exact: bool,
) -> Result<regex::Regex, regex::Error> {
    // 1. Escape all
    let reg = &regex::escape(wildcard);

    // 2. We convert the escaped '?' into '.' (matches a single character)
    let reg = &reg.replace("\\?", ".");
    // 3. We convert the escaped '*' into '.*' (matches anything)
    let reg = &reg.replace("\\*", ".*");

    // 4. We send '\\~\\~' to '??' that is an unescaped regular expression, therefore cannot be in reg
    let reg = &reg.replace("\\~\\~", "??");

    // 5. If the escaped and converted '*' is preceded by '~' then it's a raw '*'
    let reg = &reg.replace("\\~.*", "\\*");
    // 6. If the escaped and converted '.' is preceded by '~' then it's a raw '?'
    let reg = &reg.replace("\\~.", "\\?");
    // '~' is used in Excel to escape any other character.
    //    So ~x goes to x (whatever x is)
    // 7. Remove all the others '\\~d' --> 'd'
    let reg = &reg.replace("\\~", "");
    // 8. Put back the '\\~\\~'  as '\\~'
    let reg = &reg.replace("??", "\\~");

    // And we have a valid Perl regex! (As Kim Kardashian said before me: "I know, right?")
    if exact {
        return regex::Regex::new(&format!("^{reg}$"));
    }
    regex::Regex::new(reg)
}
```

### Regex Matching Utility

**File:** `base/src/functions/util.rs`
**Lines:** `207 - 212`

```rust
pub(crate) fn result_matches_regex(calc_result: &CalcResult, reg: &regex::Regex) -> bool {
    match calc_result {
        CalcResult::String(s) => reg.is_match(&s.to_lowercase()),
        _ => false,
    }
}
```

## 7. Implementation Notes

### Key Characteristics:

1. **Function Signature**: `VLOOKUP(lookup_value, table_array, column_index, [is_sorted])`
   - `lookup_value`: The value to search for in the first column
   - `table_array`: The range to search in (must be a Range type)
   - `column_index`: The column number in the table from which to return a value
   - `is_sorted`: Optional boolean (defaults to `true`) indicating whether the first column is sorted

2. **Search Behavior**:
   - **Sorted Mode** (`is_sorted=true`, default): Uses binary search for efficient lookup. Returns the largest value less than or equal to the lookup value if an exact match is not found.
   - **Unsorted Mode** (`is_sorted=false`): Uses linear search for exact matches. Supports wildcard matching for string values using `*` and `?` characters.

3. **Error Handling**:
   - Returns `#ERROR!` if the wrong number of arguments is provided
   - Returns `#N/A` if the lookup value is not found
   - Returns `#REF!` if the column index is out of bounds
   - Returns `#VALUE!` or `#N/A` if the table_array is not a valid range

4. **Type Handling**:
   - String comparisons are case-insensitive
   - Wildcard patterns are supported in unsorted mode for string lookups
   - Empty cells are treated as empty strings or zeros depending on context

5. **Dependencies**:
   - Uses `binary_search()` method for sorted lookups
   - Uses `compare_values()` for value comparison
   - Uses `from_wildcard_to_regex()` and `result_matches_regex()` for wildcard pattern matching in unsorted mode

Code Context Report for: `COLUMN`

This report contains all relevant code snippets for the `COLUMN` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `base/src/functions/lookup_and_reference.rs`
**Lines:** `629-644`

```rust
// COLUMN([reference])
// If reference is not present returns the column of the present cell.
// Otherwise returns the column number of reference
pub(crate) fn fn_column(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.len() > 1 {
        return CalcResult::new_args_number_error(cell);
    }
    if args.is_empty() {
        return CalcResult::Number(cell.column as f64);
    }

    match self.get_reference(&args[0], cell) {
        Ok(range) => CalcResult::Number(range.left.column as f64),
        Err(s) => s,
    }
}
```

## 2. Unit Tests

### Test Case 1: XLSX Integration Test

**File:** `xlsx/tests/calc_tests/ROW_COLUM.xlsx`
**Lines:** `(Binary test file)`

This XLSX test file contains automated test cases for both ROW and COLUMN functions. The test framework in `xlsx/tests/test.rs` loads all `.xlsx` files from the `tests/calc_tests/` directory and validates that the calculated results match the expected values stored in the Excel file.

**Test Execution Method:**
```rust
fn test_xlsx() {
    let mut entries = fs::read_dir("tests/calc_tests/")
        .unwrap()
        .map(|res| res.map(|e| e.path()))
        .collect::<Result<Vec<_>, io::Error>>()
        .unwrap();
    entries.sort();
    let temp_folder = env::temp_dir();
    let path = format!("{}", Uuid::new_v4());
    let dir = temp_folder.join(path);
    fs::create_dir(&dir).unwrap();
    let mut is_error = false;
    for file_path in entries {
        let file_name_str = file_path.file_name().unwrap().to_str().unwrap();
        let file_path_str = file_path.to_str().unwrap();
        println!("Testing file: {file_path_str}");
        if file_name_str.ends_with(".xlsx") && !file_name_str.starts_with('~') {
            if let Err(message) = test_file(file_path_str) {
                println!("Error with file: '{file_path_str}'");
                println!("{message}");
                is_error = true;
            }
            let t = test_load_and_saving(file_path_str, &dir);
            if t.is_err() {
                println!("Error while load and saving file: {file_path_str}");
                is_error = true;
            }
        } else {
            println!("skipping");
        }
    }
    fs::remove_dir_all(&dir).unwrap();
    assert!(
        !is_error,
        "Models were evaluated inconsistently with XLSX data."
    );
}
```

**Location in test file:** `xlsx/tests/test.rs` Lines 337-374

## 3. Usage Examples

### Example 1: Function Dispatch in Function Evaluation

**File:** `base/src/functions/mod.rs`
**Lines:** `1249`

```rust
Function::Column => self.fn_column(args, cell),
```

This shows how the COLUMN function is dispatched during function evaluation. When a formula contains a COLUMN function call, the parser routes it to the `fn_column` method.

### Example 2: Function Name Registration

**File:** `base/src/functions/mod.rs`
**Lines:** `722`

```rust
"COLUMN" => Some(Function::Column),
```

This is where the string literal "COLUMN" is mapped to the Function::Column enum variant. This allows the parser to recognize the function name when parsing formulas.

### Example 3: Function Display Implementation

**File:** `base/src/functions/mod.rs`
**Lines:** `972`

```rust
Function::Column => write!(f, "COLUMN"),
```

This shows how the function is converted back to its string representation for display purposes (useful for debugging and formula display).

### Example 4: Static Analysis Signature

**File:** `base/src/expressions/parser/static_analysis.rs`
**Lines:** `642`

```rust
Function::Column => args_signature_row(arg_count),
```

This shows the function's argument signature used in static analysis for validating formula structure.

## 4. Related Type Definitions

### Type: `CellReferenceIndex`

**File:** `base/src/expressions/types.rs`
**Lines:** `37-42`

```rust
#[derive(Serialize, Deserialize, Debug, Clone, PartialEq, Eq, Copy)]
pub struct CellReferenceIndex {
    pub sheet: u32,
    pub column: i32,
    pub row: i32,
}
```

This type represents a single cell reference with a sheet number, column number, and row number. The COLUMN function returns the `column` field as an f64.

### Type: `Range`

**File:** `base/src/calc_result.rs`
**Lines:** `5-9`

```rust
#[derive(Clone)]
pub struct Range {
    pub left: CellReferenceIndex,
    pub right: CellReferenceIndex,
}
```

This type represents a range of cells with a left (top-left) and right (bottom-right) cell reference. When COLUMN is called with a range argument, it returns the column of the `left` cell.

### Type: `CalcResult`

**File:** `base/src/calc_result.rs`
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

This enum represents all possible result types from function evaluation. The COLUMN function returns `CalcResult::Number(f64)` with the column number as the value.

### Type: `Node`

**File:** `base/src/expressions/parser.rs` (referenced from function signature)

The `Node` type represents a parsed expression node. The function parameter `args: &[Node]` allows the COLUMN function to accept zero or one argument, which must be a valid cell reference or formula.

## 5. Function Registration

### Function Enum Declaration

**File:** `base/src/functions/mod.rs`
**Lines:** `30-160` (excerpt showing Column in context)

```rust
pub enum Function {
    // Logical
    And,
    False,
    // ... other functions ...

    // Mathematical and trigonometry
    Abs,
    Acos,
    // ... other mathematical functions ...
    Choose,
    Column,      // <-- Line 54
    Columns,
    // ... continued ...

    // Lookup and reference
    Hlookup,
    Index,
    Indirect,
    Lookup,
    Match,
    Offset,
    Row,
    Rows,
    // ... continued ...
}
```

The `Column` variant is declared in the `Function` enum at line 54. Note that it's grouped with Mathematical and Trigonometry functions in the enum, though it's logically part of the Lookup and Reference function category.

### Function Name Mapping

**File:** `base/src/functions/mod.rs`
**Lines:** `715-735` (excerpt showing Column registration)

```rust
// Lookup and Reference
"CHOOSE" => Some(Function::Choose),
"COLUMN" => Some(Function::Column),      // <-- Line 722
"COLUMNS" => Some(Function::Columns),
"INDEX" => Some(Function::Index),
"INDIRECT" => Some(Function::Indirect),
"HLOOKUP" => Some(Function::Hlookup),
"LOOKUP" => Some(Function::Lookup),
"MATCH" => Some(Function::Match),
"OFFSET" => Some(Function::Offset),
"ROW" => Some(Function::Row),
"ROWS" => Some(Function::Rows),
"VLOOKUP" => Some(Function::Vlookup),
"XLOOKUP" | "_XLFN.XLOOKUP" => Some(Function::Xlookup),
```

This mapping allows the parser to recognize the string "COLUMN" in formulas and convert it to the `Function::Column` enum variant.

### Related Function: ROW

**File:** `base/src/functions/lookup_and_reference.rs`
**Lines:** `604-615`

```rust
pub(crate) fn fn_row(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.len() > 1 {
        return CalcResult::new_args_number_error(cell);
    }
    if args.is_empty() {
        return CalcResult::Number(cell.row as f64);
    }
    match self.get_reference(&args[0], cell) {
        Ok(c) => CalcResult::Number(c.left.row as f64),
        Err(s) => s,
    }
}
```

The ROW function is structurally identical to COLUMN, but operates on the row number instead of the column number.

### Related Function: COLUMNS

**File:** `base/src/functions/lookup_and_reference.rs`
**Lines:** `671-679`

```rust
pub(crate) fn fn_columns(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.len() != 1 {
        return CalcResult::new_args_number_error(cell);
    }
    match self.get_reference(&args[0], cell) {
        Ok(c) => CalcResult::Number((c.right.column - c.left.column + 1) as f64),
        Err(s) => s,
    }
}
```

The COLUMNS function requires exactly one argument and returns the width of the range (number of columns).

## Summary

The COLUMN function is a lookup and reference function that:
- Takes an optional single cell reference or range argument
- Returns the column number as a numeric value (f64)
- If no argument is provided, returns the column of the cell containing the formula
- If an argument is provided, returns the column of the leftmost cell in the range
- Is implemented using the `get_reference` utility method to resolve cell references
- Follows the same pattern as the related ROW function
- Is registered in the function dispatch system and mapped to the "COLUMN" string literal

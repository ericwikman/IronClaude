# Code Context Report for: `SUM`

This report contains all relevant code snippets for the `SUM` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `base/src/functions/mathematical.rs`
**Lines:** `540 - 634`

```rust
pub(crate) fn fn_sum(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.is_empty() {
        return CalcResult::new_args_number_error(cell);
    }

    let mut result = 0.0;
    for arg in args {
        match self.evaluate_node_in_context(arg, cell) {
            CalcResult::Number(value) => result += value,
            CalcResult::Range { left, right } => {
                if left.sheet != right.sheet {
                    return CalcResult::new_error(
                        Error::VALUE,
                        cell,
                        "Ranges are in different sheets".to_string(),
                    );
                }
                // TODO: We should do this for all functions that run through ranges
                // Running cargo test for the ironcalc takes around .8 seconds with this speedup
                // and ~ 3.5 seconds without it. Note that once properly in place sheet.dimension should be almost a noop
                let row1 = left.row;
                let mut row2 = right.row;
                let column1 = left.column;
                let mut column2 = right.column;
                if row1 == 1 && row2 == LAST_ROW {
                    row2 = match self.workbook.worksheet(left.sheet) {
                        Ok(s) => s.dimension().max_row,
                        Err(_) => {
                            return CalcResult::new_error(
                                Error::ERROR,
                                cell,
                                format!("Invalid worksheet index: '{}'", left.sheet),
                            );
                        }
                    };
                }
                if column1 == 1 && column2 == LAST_COLUMN {
                    column2 = match self.workbook.worksheet(left.sheet) {
                        Ok(s) => s.dimension().max_column,
                        Err(_) => {
                            return CalcResult::new_error(
                                Error::ERROR,
                                cell,
                                format!("Invalid worksheet index: '{}'", left.sheet),
                            );
                        }
                    };
                }
                for row in row1..row2 + 1 {
                    for column in column1..(column2 + 1) {
                        match self.evaluate_cell(CellReferenceIndex {
                            sheet: left.sheet,
                            row,
                            column,
                        }) {
                            CalcResult::Number(value) => {
                                result += value;
                            }
                            error @ CalcResult::Error { .. } => return error,
                            _ => {
                                // We ignore booleans and strings
                            }
                        }
                    }
                }
            }
            CalcResult::Array(array) => {
                for row in array {
                    for value in row {
                        match value {
                            ArrayNode::Number(value) => {
                                result += value;
                            }
                            ArrayNode::Error(error) => {
                                return CalcResult::Error {
                                    error,
                                    origin: cell,
                                    message: "Error in array".to_string(),
                                }
                            }
                            _ => {
                                // We ignore booleans and strings
                            }
                        }
                    }
                }
            }
            error @ CalcResult::Error { .. } => return error,
            _ => {
                // We ignore booleans and strings
            }
        };
    }
    CalcResult::Number(result)
}
```

## 2. Unit Tests

### Test Case 1: Basic functionality with various argument patterns

**File:** `base/src/test/test_fn_sum.rs`
**Lines:** `6 - 19`

```rust
#[test]
fn test_fn_sum_arguments() {
    let mut model = new_empty_model();
    model._set("A1", "=SUM()");
    model._set("A2", "=SUM(1, 2, 3)");
    model._set("A3", "=SUM(1, )");
    model._set("A4", "=SUM(1,   , 3)");

    model.evaluate();

    assert_eq!(model._get_text("A1"), *"#ERROR!");
    assert_eq!(model._get_text("A2"), *"6");
    assert_eq!(model._get_text("A3"), *"1");
    assert_eq!(model._get_text("A4"), *"4");
}
```

### Test Case 2: Arrays (horizontal, vertical, and 2D)

**File:** `base/src/test/test_fn_sum.rs`
**Lines:** `22 - 35`

```rust
#[test]
fn arrays() {
    let mut model = new_empty_model();
    model._set("A1", "=SUM({1, 2, 3})");
    model._set("A2", "=SUM({1; 2; 3})");
    model._set("A3", "=SUM({1, 2; 3, 4})");
    model._set("A4", "=SUM({1, 2; 3, 4; 5, 6})");

    model.evaluate();

    assert_eq!(model._get_text("A1"), *"6");
    assert_eq!(model._get_text("A2"), *"6");
    assert_eq!(model._get_text("A3"), *"10");
    assert_eq!(model._get_text("A4"), *"21");
}
```

### Test Case 3: Array arithmetic operations

**File:** `base/src/test/test_arrays.rs`
**Lines:** `6 - 13`

```rust
#[test]
fn sum_arrays() {
    let mut model = new_empty_model();
    model._set("A1", "=SUM({1,2,3}+{3,4,5})");

    model.evaluate();

    assert_eq!(model._get_text("A1"), *"18");
}
```

## 3. Usage Examples

### Example 1: Basic range summing

**File:** `base/src/test/test_general.rs`
**Lines:** `75 - 79`

```rust
model
    .set_user_input(0, 2, 1, "=SUM(A1, B1)".to_string())
    .unwrap(); // A2
model.evaluate();
let result = model._get_text_at(0, 2, 1);
assert_eq!(result, *"65");
```

### Example 2: Complete example with range reference

**File:** `base/examples/formulas_and_errors.rs`
**Lines:** `1 - 46`

```rust
use ironcalc_base::{types::CellType, Model};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut model = Model::new_empty("formulas-and-errors", "en", "UTC")?;
    // A1
    model.set_user_input(0, 1, 1, "1".to_string())?;
    // A2
    model.set_user_input(0, 2, 1, "2".to_string())?;
    // A3
    model.set_user_input(0, 3, 1, "3".to_string())?;
    // B1
    model.set_user_input(0, 1, 2, "=SUM(A1:A3)".to_string())?;
    // B2
    model.set_user_input(0, 2, 2, "=B1/0".to_string())?;
    // Evaluate
    model.evaluate();

    let cells = model.get_all_cells();

    let mut cells_count = 0;
    let mut formula_count = 0;
    let mut error_count = 0;

    for cell in cells {
        if let Some(cell) = model
            .workbook
            .worksheet(cell.index)?
            .cell(cell.row, cell.column)
        {
            if cell.get_type() == CellType::ErrorValue {
                error_count += 1;
            }
            if cell.has_formula() {
                formula_count += 1;
            }

            cells_count += 1;
        }
    }

    assert_eq!(cells_count, 5);
    assert_eq!(formula_count, 2);
    assert_eq!(error_count, 1);

    Ok(())
}
```

## 4. Related Type Definitions

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

### Type: `ArrayNode`

**File:** `base/src/expressions/parser/mod.rs`
**Lines:** `97 - 103`

```rust
#[derive(PartialEq, Clone, Debug)]
pub enum ArrayNode {
    Boolean(bool),
    Number(f64),
    String(String),
    Error(token::Error),
}
```

### Type: `Node`

**File:** `base/src/expressions/parser/mod.rs`
**Lines:** `105 - 150`

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
    // ... (truncated, see full file for complete enum)
```

## 5. Function Registration

### Registration in Function Enum

**File:** `base/src/functions/mod.rs`
**Lines:** `75 - 77`

```rust
// Mathematical and trigonometry
// ...
Sum,
Sumif,
Sumifs,
```

### Registration in Function Iterator

**File:** `base/src/functions/mod.rs`
**Lines:** `390 - 392`

```rust
Function::Sum,
Function::Sumif,
Function::Sumifs,
```

### Registration in String-to-Function Mapping

**File:** `base/src/functions/mod.rs`
**Lines:** `713 - 715`

```rust
"SUM" => Some(Function::Sum),
"SUMIF" => Some(Function::Sumif),
"SUMIFS" => Some(Function::Sumifs),
```

### Registration in Function Display

**File:** `base/src/functions/mod.rs`
**Lines:** `968 - 970`

```rust
Function::Sum => write!(f, "SUM"),
Function::Sumif => write!(f, "SUMIF"),
Function::Sumifs => write!(f, "SUMIFS"),
```

### Registration in Function Evaluation Dispatcher

**File:** `base/src/functions/mod.rs`
**Lines:** `1245 - 1247`

```rust
Function::Sum => self.fn_sum(args, cell),
Function::Sumif => self.fn_sumif(args, cell),
Function::Sumifs => self.fn_sumifs(args, cell),
```

## 6. Additional Context

### Performance Optimization Note

The function includes a performance optimization that limits range iteration to the actual used dimension of the worksheet rather than the theoretical maximum (LAST_ROW and LAST_COLUMN). According to the comment at line 557-559 in `mathematical.rs`:

```rust
// TODO: We should do this for all functions that run through ranges
// Running cargo test for the ironcalc takes around .8 seconds with this speedup
// and ~ 3.5 seconds without it.
```

### Behavior with Different Value Types

Based on the implementation (lines 599-601, 620-622, 628-630), the SUM function:
- **Includes**: Numbers from direct arguments, ranges, and arrays
- **Ignores**: Booleans and strings (does not coerce them to numbers)
- **Propagates**: Errors immediately when encountered
- **Handles**: Empty arguments (treats them as 0)

### Error Conditions

1. **No arguments**: Returns `#ERROR!` (Wrong number of arguments)
2. **Cross-sheet ranges**: Returns `#VALUE!` error with message "Ranges are in different sheets"
3. **Invalid worksheet index**: Returns `#ERROR!` with message "Invalid worksheet index: '{sheet}'"
4. **Array contains error**: Propagates the error with message "Error in array"
5. **Cell evaluation error**: Propagates the error immediately

# Code Context Report for: `ISNUMBER`

This report contains all relevant code snippets for the `ISNUMBER` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `base/src/functions/information.rs`
**Lines:** `8 - 18`

```rust
pub(crate) fn fn_isnumber(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.len() == 1 {
        match self.evaluate_node_in_context(&args[0], cell) {
            CalcResult::Number(_) => return CalcResult::Boolean(true),
            _ => {
                return CalcResult::Boolean(false);
            }
        };
    }
    CalcResult::new_args_number_error(cell)
}
```

## 2. Unit Tests

### Test Case 1: Quote prefix with numbers (Testing string vs number distinction)

**File:** `base/src/test/test_quote_prefix.rs`
**Lines:** `15 - 29`

```rust
#[test]
fn test_quote_prefix_number() {
    let mut model = new_empty_model();
    model._set("A1", "'13");
    model._set("A2", "=ISNUMBER(A1)");
    model._set("A3", "=A1+1");
    model._set("A4", "=ISNUMBER(A3)");
    model.evaluate();
    assert_eq!(model._get_text("A1"), *"13");
    assert!(!model._has_formula("A1"));

    assert_eq!(model._get_text("A2"), *"FALSE");
    assert_eq!(model._get_text("A3"), *"14");

    assert_eq!(model._get_text("A4"), *"TRUE");
}
```

### Test Case 2: Currency values (Testing number recognition with currency formatting)

**File:** `base/src/test/test_set_user_input.rs`
**Lines:** `6 - 53`

```rust
#[test]
fn test_currencies() {
    let mut model = new_empty_model();
    model
        .set_user_input(0, 1, 1, "$100.348".to_string())
        .unwrap();
    model
        .set_user_input(0, 1, 2, "=ISNUMBER(A1)".to_string())
        .unwrap();

    model
        .set_user_input(0, 2, 1, "$ 100.348".to_string())
        .unwrap();
    model
        .set_user_input(0, 2, 2, "=ISNUMBER(A2)".to_string())
        .unwrap();

    model.set_user_input(0, 3, 1, "100$".to_string()).unwrap();
    model
        .set_user_input(0, 3, 2, "=ISNUMBER(A3)".to_string())
        .unwrap();

    model
        .set_user_input(0, 4, 1, "3.1415926$".to_string())
        .unwrap();

    model.evaluate();

    // two decimal rounded up
    assert_eq!(model._get_text("A1"), "$100.35");
    assert_eq!(model._get_text("B1"), *"TRUE");
    assert_eq!(
        model.get_cell_value_by_ref("Sheet1!A1"),
        Ok(CellValue::Number(100.348))
    );
    // No space
    assert_eq!(model._get_text("A2"), "$100.35");
    assert_eq!(
        model.get_cell_value_by_ref("Sheet1!A2"),
        Ok(CellValue::Number(100.348))
    );
    assert_eq!(model._get_text("B2"), *"TRUE");

    // Dollar is on the right
    assert_eq!(model._get_text("A3"), "100$");
    assert_eq!(model._get_text("B3"), *"TRUE");

    assert_eq!(model._get_text("A4"), "3.14$");
}
```

### Test Case 3: Percentage values (Testing number recognition with percentage formatting)

**File:** `base/src/test/test_set_user_input.rs`
**Lines:** `74 - 93`

```rust
#[test]
fn test_percentage() {
    let mut model = new_empty_model();
    model.set_user_input(0, 10, 1, "50%".to_string()).unwrap();
    model
        .set_user_input(0, 10, 2, "=ISNUMBER(A10)".to_string())
        .unwrap();
    model
        .set_user_input(0, 11, 1, "55.759%".to_string())
        .unwrap();

    model.evaluate();

    assert_eq!(model._get_text("B10"), *"TRUE");
    assert_eq!(
        model.get_cell_value_by_ref("Sheet1!A10"),
        Ok(CellValue::Number(0.5))
    );
    // Two decimal places
    assert_eq!(model._get_text("A11"), "55.76%");
}
```

### Test Case 4: Quote prefix re-entry behavior

**File:** `base/src/test/test_quote_prefix.rs`
**Lines:** `71 - 81`

```rust
#[test]
fn test_quote_prefix_reenter() {
    let mut model = new_empty_model();
    model._set("A1", "'123");
    model._set("A2", "=ISTEXT(A1)");
    model.evaluate();
    assert_eq!(model._get_text("A2"), *"TRUE");
    // We introduce a value with a "quote prefix" index
    model.set_user_input(0, 1, 1, "123".to_string()).unwrap();
    model.evaluate();
    assert_eq!(model._get_text("A2"), *"FALSE");
}
```

### Test Case 5: Text to number conversion behavior

**File:** `base/src/test/test_quote_prefix.rs`
**Lines:** `93 - 103`

```rust
#[test]
fn test_update_quote_prefix_reenter() {
    let mut model = new_empty_model();
    model.update_cell_with_text(0, 1, 1, "123").unwrap();
    model._set("A2", "=ISTEXT(A1)");
    model.evaluate();
    assert_eq!(model._get_text("A2"), *"TRUE");
    // We reenter as a number
    model.update_cell_with_number(0, 1, 1, 123.0).unwrap();
    model.evaluate();
    assert_eq!(model._get_text("A2"), *"FALSE");
}
```

## 3. Usage Examples

### Example 1: Formula verification in cell content retrieval

**File:** `base/src/test/test_get_cell_content.rs`
**Lines:** `6 - 22`

```rust
#[test]
fn test_formulas() {
    let mut model = new_empty_model();
    model
        .set_user_input(0, 1, 1, "$100.348".to_string())
        .unwrap();
    model
        .set_user_input(0, 1, 2, "=ISNUMBER(A1)".to_string())
        .unwrap();

    model.evaluate();

    assert_eq!(model.get_cell_content(0, 1, 1).unwrap(), "100.348");
    assert_eq!(model.get_cell_content(0, 1, 2).unwrap(), "=ISNUMBER(A1)");
    assert_eq!(model.get_cell_content(0, 5, 5).unwrap(), "");

    assert!(model.get_cell_content(1, 1, 2).is_err());
}
```

### Example 2: Implicit intersection test (commented reference)

**File:** `base/src/expressions/parser/tests/test_add_implicit_intersection.rs`
**Lines:** `67 - 67`

```rust
// ("ISNUMBER(A1:A10)", "ISNUMBER(A1:A10)"),
```

**Note:** This example shows that ISNUMBER is one of the information functions that does not require implicit intersection when passed a range, as it operates on the evaluated result of its argument.

## 4. Related Type Definitions

### Type: `CalcResult`

**File:** `base/src/calc_result.rs`
**Lines:** `11 - 28`

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
**Lines:** `105 - 151`

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
        // ... (additional variants follow)
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

### Type: `Function` (Enum variant)

**File:** `base/src/functions/mod.rs`
**Lines:** `118 - 136`

```rust
// Information
ErrorType,
Formulatext,
Isblank,
Iserr,
Iserror,
Iseven,
Isformula,
Islogical,
Isna,
Isnontext,
Isnumber,
Isodd,
Isref,
Istext,
Na,
Sheet,
Type,
```

## 5. Function Registration

### Registration: String to Function Enum Mapping

**File:** `base/src/functions/mod.rs`
**Lines:** `759 - 759`

```rust
"ISNUMBER" => Some(Function::Isnumber),
```

### Registration: Function Dispatch in evaluate_function

**File:** `base/src/functions/mod.rs`
**Lines:** `1278 - 1278`

```rust
Function::Isnumber => self.fn_isnumber(args, cell),
```

### Registration: Display Implementation

**File:** `base/src/functions/mod.rs`
**Lines:** `1001 - 1001`

```rust
Function::Isnumber => write!(f, "ISNUMBER"),
```

## Summary

The `ISNUMBER` function is implemented as:
- **Location**: Information functions module (`base/src/functions/information.rs`)
- **Implementation**: A simple pattern-matching function that evaluates its single argument and checks if the result is a `CalcResult::Number` variant
- **Parameters**: Takes exactly 1 argument (returns error if not)
- **Return Value**: Boolean - `TRUE` if the evaluated argument is a number, `FALSE` otherwise
- **Error Handling**: Returns a "Wrong number of arguments" error if not exactly 1 argument is provided
- **Key Behavior**: Evaluates the argument in context before checking its type, which means it properly handles cell references, formulas, and other expressions

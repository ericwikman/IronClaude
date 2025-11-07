# Code Context Report for: `IF`

This report contains all relevant code snippets for the `IF` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `base/src/functions/logical.rs`
**Lines:** `26 - 44`

```rust
pub(crate) fn fn_if(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.len() == 2 || args.len() == 3 {
        let cond_result = self.get_boolean(&args[0], cell);
        let cond = match cond_result {
            Ok(f) => f,
            Err(s) => {
                return s;
            }
        };
        if cond {
            return self.evaluate_node_in_context(&args[1], cell);
        } else if args.len() == 3 {
            return self.evaluate_node_in_context(&args[2], cell);
        } else {
            return CalcResult::Boolean(false);
        }
    }
    CalcResult::new_args_number_error(cell)
}
```

## 2. Unit Tests

### Test Case 1: Argument Validation

**File:** `base/src/test/test_fn_if.rs`
**Lines:** `5 - 14`

```rust
#[test]
fn fn_if_arguments() {
    let mut model = new_empty_model();
    model._set("A1", "=IF()");
    model._set("A2", "=IF(1, 2, 3, 4)");
    model.evaluate();

    assert_eq!(model._get_text("A1"), *"#ERROR!");
    assert_eq!(model._get_text("A2"), *"#ERROR!");
}
```

### Test Case 2: Two-Argument Form (Returns FALSE for false condition)

**File:** `base/src/test/test_fn_if.rs`
**Lines:** `16 - 22`

```rust
#[test]
fn fn_if_2_args() {
    let mut model = new_empty_model();
    model._set("A1", "=IF(2 > 3, TRUE)");
    model.evaluate();
    assert_eq!(model._get_text("A1"), *"FALSE");
}
```

### Test Case 3: Handling of Missing/Empty Arguments

**File:** `base/src/test/test_fn_if.rs`
**Lines:** `24 - 36`

```rust
#[test]
fn fn_if_missing_args() {
    let mut model = new_empty_model();
    model._set("A1", "=IF(2 > 3, TRUE, )");
    model._set("A2", "=IF(2 > 3, , 5)");
    model._set("A3", "=IF(2 < 3, , 5)");

    model.evaluate();

    // assert_eq!(model._get_text("A1"), *"0");
    assert_eq!(model._get_text("A2"), *"5");
    assert_eq!(model._get_text("A3"), *"0");
}
```

### Test Case 4: Boolean Literal Handling (within test_booleans)

**File:** `base/src/test/test_general.rs`
**Lines:** `171 - 222`

```rust
#[test]
fn test_booleans() {
    let mut model = new_empty_model();
    model.set_user_input(0, 1, 1, "true".to_string()).unwrap();
    model.set_user_input(0, 2, 1, "TRUE".to_string()).unwrap();
    model.set_user_input(0, 3, 1, "True".to_string()).unwrap();
    model.set_user_input(0, 4, 1, "false".to_string()).unwrap();
    model.set_user_input(0, 5, 1, "FALSE".to_string()).unwrap();
    model.set_user_input(0, 6, 1, "False".to_string()).unwrap();

    model
        .set_user_input(0, 1, 2, "=ISLOGICAL(A1)".to_string())
        .unwrap();
    model
        .set_user_input(0, 2, 2, "=ISLOGICAL(A2)".to_string())
        .unwrap();
    model
        .set_user_input(0, 3, 2, "=ISLOGICAL(A3)".to_string())
        .unwrap();
    model
        .set_user_input(0, 4, 2, "=ISLOGICAL(A4)".to_string())
        .unwrap();
    model
        .set_user_input(0, 5, 2, "=ISLOGICAL(A5)".to_string())
        .unwrap();
    model
        .set_user_input(0, 6, 2, "=ISLOGICAL(A6)".to_string())
        .unwrap();

    model
        .set_user_input(0, 1, 5, "=IF(false, True, FALSe)".to_string())
        .unwrap();

    model.evaluate();

    assert_eq!(model._get_text_at(0, 1, 1), *"TRUE");
    assert_eq!(model._get_text_at(0, 2, 1), *"TRUE");
    assert_eq!(model._get_text_at(0, 3, 1), *"TRUE");

    assert_eq!(model._get_text_at(0, 4, 1), *"FALSE");
    assert_eq!(model._get_text_at(0, 5, 1), *"FALSE");
    assert_eq!(model._get_text_at(0, 6, 1), *"FALSE");

    assert_eq!(model._get_text_at(0, 1, 2), *"TRUE");
    assert_eq!(model._get_text_at(0, 2, 2), *"TRUE");
    assert_eq!(model._get_text_at(0, 3, 2), *"TRUE");
    assert_eq!(model._get_text_at(0, 4, 2), *"TRUE");
    assert_eq!(model._get_text_at(0, 5, 2), *"TRUE");
    assert_eq!(model._get_text_at(0, 6, 2), *"TRUE");

    assert_eq!(model._get_formula("E1"), *"=IF(FALSE,TRUE,FALSE)");
}
```

## 3. Usage Examples

### Example 1: Using IF with Defined Names

**File:** `base/src/test/user_model/test_defined_names.rs`
**Lines:** `182 - 192`

```rust
model
    .new_defined_name("answer", None, "Sheet1!$A$1")
    .unwrap();

model
    .set_user_input(0, 2, 1, "=IF(answer<2, answer*2, answer^2)")
    .unwrap();

model
    .set_user_input(0, 3, 1, "=badDunction(-answer)")
    .unwrap();
```

### Example 2: IF in Documentation Comments - Locale Differences

**File:** `base/src/expressions/lexer/mod.rs`
**Lines:** `17 - 20`

```rust
//! Formulas look different in different locales:
//!
//! =IF(A1, B1, NA()) versus =IF(A1; B1; NA())
//!
```

### Example 3: IF in Documentation Comments - Number Separator Context

**File:** `base/src/expressions/lexer/mod.rs`
**Lines:** `524 - 528`

```rust
// Let's say ',' is the thousands separator. Then 1,234 would be an error.
// This is ok for most cases:
// =IF(A1=1,234, TRUE, FALSE) will not work
// If a user introduces a single number in the cell 1,234 we should be able to parse
// and format the cell appropriately
```

## 4. Related Type Definitions

### Type: `CalcResult`

**File:** `base/src/calc_result.rs`
**Lines:** `11 - 27`

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
**Lines:** `105 - 184`

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
    // ... (additional variants truncated for brevity)
}
```

### Type: `CellReferenceIndex`

**File:** `base/src/expressions/types.rs`
**Lines:** `38 - 42`

```rust
pub struct CellReferenceIndex {
    pub sheet: u32,
    pub column: i32,
    pub row: i32,
}
```

## 5. Function Registration

### Enum Variant Definition

**File:** `base/src/functions/mod.rs`
**Lines:** `30 - 42`

```rust
pub enum Function {
    // Logical
    And,
    False,
    If,
    Iferror,
    Ifna,
    Ifs,
    Not,
    Or,
    Switch,
    True,
    Xor,
    // ... (additional function variants follow)
}
```

### String to Function Mapping

**File:** `base/src/functions/mod.rs`
**Lines:** `639 - 643`

```rust
"AND" => Some(Function::And),
"FALSE" => Some(Function::False),
"IF" => Some(Function::If),
"IFERROR" => Some(Function::Iferror),
"IFNA" | "_XLFN.IFNA" => Some(Function::Ifna),
```

### Display Implementation

**File:** `base/src/functions/mod.rs`
**Lines:** `920 - 924`

```rust
Function::And => write!(f, "AND"),
Function::False => write!(f, "FALSE"),
Function::If => write!(f, "IF"),
Function::Iferror => write!(f, "IFERROR"),
Function::Ifna => write!(f, "IFNA"),
```

### Evaluation Dispatch

**File:** `base/src/functions/mod.rs`
**Lines:** `1205 - 1212`

```rust
Function::And => self.fn_and(args, cell),
Function::False => self.fn_false(args, cell),
Function::If => self.fn_if(args, cell),
Function::Iferror => self.fn_iferror(args, cell),
Function::Ifna => self.fn_ifna(args, cell),
Function::Ifs => self.fn_ifs(args, cell),
Function::Not => self.fn_not(args, cell),
Function::Or => self.fn_or(args, cell),
```

## Summary

The `IF` function is a core logical function in IronClaude that:

- **Accepts 2-3 arguments**: condition (required), value_if_true (required), value_if_false (optional)
- **Returns FALSE** when the condition is false and no third argument is provided
- **Supports lazy evaluation**: Only evaluates the branch that will be returned
- **Handles errors**: Propagates errors from the condition evaluation
- **Validates arguments**: Returns #ERROR! for incorrect number of arguments (0, 1, or 4+ arguments)
- **Uses Boolean conversion**: Converts the first argument to a boolean using `get_boolean()`
- **Part of the Logical function family**: Registered alongside AND, OR, NOT, IFERROR, IFNA, IFS, XOR, and SWITCH

The function is implemented as a method on the `Model` struct and follows the standard evaluation pattern used throughout the IronClaude function library.

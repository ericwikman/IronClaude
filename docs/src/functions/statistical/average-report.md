# Code Context Report for: `AVERAGE`

This report contains all relevant code snippets for the `AVERAGE` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `base/src/functions/statistical.rs`
**Lines:** `13 - 92`

```rust
pub(crate) fn fn_average(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.is_empty() {
        return CalcResult::new_args_number_error(cell);
    }
    let mut count = 0.0;
    let mut sum = 0.0;
    for arg in args {
        match self.evaluate_node_in_context(arg, cell) {
            CalcResult::Number(value) => {
                count += 1.0;
                sum += value;
            }
            CalcResult::Boolean(b) => {
                if let Node::ReferenceKind { .. } = arg {
                } else {
                    sum += if b { 1.0 } else { 0.0 };
                    count += 1.0;
                }
            }
            CalcResult::Range { left, right } => {
                if left.sheet != right.sheet {
                    return CalcResult::new_error(
                        Error::VALUE,
                        cell,
                        "Ranges are in different sheets".to_string(),
                    );
                }
                for row in left.row..(right.row + 1) {
                    for column in left.column..(right.column + 1) {
                        match self.evaluate_cell(CellReferenceIndex {
                            sheet: left.sheet,
                            row,
                            column,
                        }) {
                            CalcResult::Number(value) => {
                                count += 1.0;
                                sum += value;
                            }
                            error @ CalcResult::Error { .. } => return error,
                            CalcResult::Range { .. } => {
                                return CalcResult::new_error(
                                    Error::ERROR,
                                    cell,
                                    "Unexpected Range".to_string(),
                                );
                            }
                            _ => {}
                        }
                    }
                }
            }
            error @ CalcResult::Error { .. } => return error,
            CalcResult::String(s) => {
                if let Node::ReferenceKind { .. } = arg {
                    // Do nothing
                } else if let Ok(t) = s.parse::<f64>() {
                    sum += t;
                    count += 1.0;
                } else {
                    return CalcResult::Error {
                        error: Error::VALUE,
                        origin: cell,
                        message: "Argument cannot be cast into number".to_string(),
                    };
                }
            }
            _ => {
                // Ignore everything else
            }
        };
    }
    if count == 0.0 {
        return CalcResult::Error {
            error: Error::DIV,
            origin: cell,
            message: "Division by Zero".to_string(),
        };
    }
    CalcResult::Number(sum / count)
}
```

## 2. Unit Tests

### Test Case 1: Error handling for empty arguments

**File:** `base/src/test/test_fn_average.rs`
**Lines:** `6 - 14`

```rust
#[test]
fn test_fn_average_arguments() {
    let mut model = new_empty_model();
    model._set("A1", "=AVERAGE()");
    model._set("A2", "=AVERAGEA()");
    model.evaluate();

    assert_eq!(model._get_text("A1"), *"#ERROR!");
    assert_eq!(model._get_text("A2"), *"#ERROR!");
}
```

### Test Case 2: Basic functionality with mixed data types

**File:** `base/src/test/test_fn_average.rs`
**Lines:** `16 - 31`

```rust
#[test]
fn test_fn_average_minimal() {
    let mut model = new_empty_model();
    model._set("B1", "1");
    model._set("B2", "2");
    model._set("B3", "3");
    model._set("B4", "'2");
    // B5 is empty
    model._set("B6", "true");
    model._set("A1", "=AVERAGE(B1:B6)");
    model._set("A2", "=AVERAGEA(B1:B6)");
    model.evaluate();

    assert_eq!(model._get_text("A1"), *"2");
    assert_eq!(model._get_text("A2"), *"1.4");
}
```

### Test Case 3: AVERAGEIFS argument validation

**File:** `base/src/test/test_fn_averageifs.rs`
**Lines:** `6 - 40`

```rust
#[test]
fn test_fn_averageifs_arguments() {
    let mut model = new_empty_model();

    // Incorrect number of arguments
    model._set("A1", "=AVERAGEIFS()");
    model._set("A2", "=AVERAGEIFS(B2:B9)");
    model._set("A3", "=AVERAGEIFS(B2:B9,C2:C9)");
    model._set("A4", "=AVERAGEIFS(B2:B9,C2:C9,\"=A*\",D2:D9)");

    // Correct (Sum everything in column 'B' if column 'C' starts with "A")
    model._set("A5", "=AVERAGEIFS(B2:B9,C2:C9,\"=A*\")");

    // Data
    model._set("B2", "5");
    model._set("B3", "4");
    model._set("B4", "15");
    model._set("B5", "22");
    model._set("B6", "=NA()");
    model._set("C2", "Apples");
    model._set("C3", "Bananas");
    model._set("C4", "Almonds");
    model._set("C5", "Yoni");
    model._set("C6", "Mandarin");

    model.evaluate();

    // Error (Incorrect number of arguments)
    assert_eq!(model._get_text("A1"), *"#ERROR!");
    assert_eq!(model._get_text("A2"), *"#ERROR!");
    assert_eq!(model._get_text("A3"), *"#ERROR!");
    assert_eq!(model._get_text("A4"), *"#ERROR!");

    // Correct
    assert_eq!(model._get_text("A5"), *"10");
}
```

## 3. Usage Examples

### Example 1: SUBTOTAL function using average internally

**File:** `base/src/functions/subtotal.rs`
**Lines:** `524 - 547`

```rust
fn subtotal_average(
    &mut self,
    args: &[Node],
    cell: CellReferenceIndex,
    mode: SubTotalMode,
) -> CalcResult {
    let values = match self.subtotal_get_values(args, cell, mode) {
        Ok(s) => s,
        Err(s) => return s,
    };
    let mut result = 0.0;
    let l = values.len();
    for value in values {
        result += value;
    }
    if l == 0 {
        return CalcResult::Error {
            error: Error::DIV,
            origin: cell,
            message: "Division by 0!".to_string(),
        };
    }
    CalcResult::Number(result / (l as f64))
}
```

### Example 2: AVERAGEIF function using AVERAGEIFS

**File:** `base/src/functions/statistical.rs`
**Lines:** `336 - 348`

```rust
/// AVERAGEIF(criteria_range, criteria, [average_range])
/// if average_rage is missing then criteria_range will be used
pub(crate) fn fn_averageif(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    if args.len() == 2 {
        let arguments = vec![args[0].clone(), args[0].clone(), args[1].clone()];
        self.fn_averageifs(&arguments, cell)
    } else if args.len() == 3 {
        let arguments = vec![args[2].clone(), args[0].clone(), args[1].clone()];
        self.fn_averageifs(&arguments, cell)
    } else {
        CalcResult::new_args_number_error(cell)
    }
}
```

### Example 3: AVERAGEIFS implementation

**File:** `base/src/functions/statistical.rs`
**Lines:** `606 - 626`

```rust
pub(crate) fn fn_averageifs(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
    let mut total = 0.0;
    let mut count = 0.0;

    let average = |value: f64| {
        total += value;
        count += 1.0;
    };
    if let Err(e) = self.apply_ifs(args, cell, average) {
        return e;
    }

    if count == 0.0 {
        return CalcResult::Error {
            error: Error::DIV,
            origin: cell,
            message: "division by 0".to_string(),
        };
    }
    CalcResult::Number(total / count)
}
```

## 4. Related Type Definitions

### Type: `Node`

**File:** `base/src/expressions/parser/mod.rs`
**Lines:** `106 - 199`

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
```

### Type: `CalcResult`

**File:** `base/src/calc_result.rs`
**Lines:** `5 - 48`

```rust
#[derive(Clone)]
pub struct Range {
    pub left: CellReferenceIndex,
    pub right: CellReferenceIndex,
}

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

### Type: `Error`

**File:** `base/src/expressions/token.rs`
**Lines:** `78 - 97`

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

### Registration in Function Enum

**File:** `base/src/functions/mod.rs`
**Lines:** `178 - 190`

```rust
// Statistical
Average,
Averagea,
Averageif,
Averageifs,
Count,
Counta,
Countblank,
Countif,
Countifs,
Maxifs,
Minifs,
Geomean,
```

### Registration in String Mapping

**File:** `base/src/functions/mod.rs`
**Lines:** `777 - 788`

```rust
"AVERAGE" => Some(Function::Average),
"AVERAGEA" => Some(Function::Averagea),
"AVERAGEIF" => Some(Function::Averageif),
"AVERAGEIFS" => Some(Function::Averageifs),
"COUNT" => Some(Function::Count),
"COUNTA" => Some(Function::Counta),
"COUNTBLANK" => Some(Function::Countblank),
"COUNTIF" => Some(Function::Countif),
"COUNTIFS" => Some(Function::Countifs),
"MAXIFS" | "_XLFN.MAXIFS" => Some(Function::Maxifs),
"MINIFS" | "_XLFN.MINIFS" => Some(Function::Minifs),
"GEOMEAN" => Some(Function::Geomean),
```

### Registration in Function Dispatcher

**File:** `base/src/functions/mod.rs`
**Lines:** `1295 - 1306`

```rust
Function::Average => self.fn_average(args, cell),
Function::Averagea => self.fn_averagea(args, cell),
Function::Averageif => self.fn_averageif(args, cell),
Function::Averageifs => self.fn_averageifs(args, cell),
Function::Count => self.fn_count(args, cell),
Function::Counta => self.fn_counta(args, cell),
Function::Countblank => self.fn_countblank(args, cell),
Function::Countif => self.fn_countif(args, cell),
Function::Countifs => self.fn_countifs(args, cell),
Function::Maxifs => self.fn_maxifs(args, cell),
Function::Minifs => self.fn_minifs(args, cell),
Function::Geomean => self.fn_geomean(args, cell),
```

### Registration in Display Trait

**File:** `base/src/functions/mod.rs`
**Lines:** `1018 - 1029`

```rust
Function::Average => write!(f, "AVERAGE"),
Function::Averagea => write!(f, "AVERAGEA"),
Function::Averageif => write!(f, "AVERAGEIF"),
Function::Averageifs => write!(f, "AVERAGEIFS"),
Function::Count => write!(f, "COUNT"),
Function::Counta => write!(f, "COUNTA"),
Function::Countblank => write!(f, "COUNTBLANK"),
Function::Countif => write!(f, "COUNTIF"),
Function::Countifs => write!(f, "COUNTIFS"),
Function::Maxifs => write!(f, "MAXIFS"),
Function::Minifs => write!(f, "MINIFS"),
Function::Geomean => write!(f, "GEOMEAN"),
```

### Registration in Function Iterator

**File:** `base/src/functions/mod.rs`
**Lines:** `440 - 451`

```rust
Function::Average,
Function::Averagea,
Function::Averageif,
Function::Averageifs,
Function::Count,
Function::Counta,
Function::Countblank,
Function::Countif,
Function::Countifs,
Function::Maxifs,
Function::Minifs,
Function::Geomean,
```

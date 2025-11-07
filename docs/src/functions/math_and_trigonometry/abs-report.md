# Code Context Report for: `ABS`

This report contains all relevant code snippets for the `ABS` function, gathered to assist in writing its technical documentation.

## 1. Primary Definition

**File:** `/home/eric/projects/IronCalc/base/src/functions/mathematical.rs`
**Lines:** `1242`

```rust
single_number_fn!(fn_abs, |f| Ok(f64::abs(f)));
```

## 2. Unit Tests

### Test Case 1: Positive Numbers

**File:** `/home/eric/projects/IronCalc/xlsx/tests/calc_tests/ABS.xlsx`
**Lines:** `2-4`

```
Row 1: Headers - [value, ABS]
Row 2: [1, =ABS(A2)]
Row 3: [12, =ABS(A3)]
Row 4: [12.5, =ABS(A4)]
```

### Test Case 2: Boolean Values (True/False)

**File:** `/home/eric/projects/IronCalc/xlsx/tests/calc_tests/ABS.xlsx`
**Lines:** `5-6`

```
Row 5: [True, =ABS(A5)]
Row 6: [False, =ABS(A6)]
```

### Test Case 3: Negative Numbers

**File:** `/home/eric/projects/IronCalc/xlsx/tests/calc_tests/ABS.xlsx`
**Lines:** `7-8`

```
Row 7: [-12.4, =ABS(A7)]
Row 8: [-9, =ABS(A8)]
```

### Test Case 4: Error Handling - Division by Zero

**File:** `/home/eric/projects/IronCalc/xlsx/tests/calc_tests/ABS.xlsx`
**Lines:** `9`

```
Row 9: [=1/0, =ABS(A9)]
```

### Test Case 5: Error Handling - NA() Error

**File:** `/home/eric/projects/IronCalc/xlsx/tests/calc_tests/ABS.xlsx`
**Lines:** `10`

```
Row 10: [=NA(), =ABS(A10)]
```

### Test Case 6: Error Handling - Invalid Text Input

**File:** `/home/eric/projects/IronCalc/xlsx/tests/calc_tests/ABS.xlsx`
**Lines:** `11`

```
Row 11: [Hello, =ABS(A11)]
```

### Test Case 7: Scientific Notation (Very Small Negative Number)

**File:** `/home/eric/projects/IronCalc/xlsx/tests/calc_tests/ABS.xlsx`
**Lines:** `12`

```
Row 12: [-1e-08, =ABS(A12)]
```

## 3. Usage Examples

### Example 1: Function Dispatch in Calculator Engine

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `1232`

```rust
Function::Abs => self.fn_abs(args, cell),
```

### Example 2: Function Name Registration (String Mapping)

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `697`

```rust
"ABS" => Some(Function::Abs),
```

### Example 3: Function Display Formatting

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `954`

```rust
Function::Abs => write!(f, "ABS"),
```

## 4. Related Type Definitions

### Type: `Function` (Enum - ABS Variant)

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `30-116` (Enum definition; ABS at line 45)

```rust
pub enum Function {
    // Logical
    And,
    False,
    If,
    // ... other variants ...

    // Mathematical and trigonometry
    Abs,
    Acos,
    Acosh,
    // ... other mathematical functions ...
}
```

### Type: `NumberOrArray` (Related to Macro Processing)

**File:** `/home/eric/projects/IronCalc/base/src/functions/macros.rs`
**Lines:** `1-100` (macro_rules! single_number_fn definition)

```rust
#[macro_export]
macro_rules! single_number_fn {
    // The macro takes:
    //   1) A function name to define (e.g. fn_abs)
    //   2) The operation to apply (e.g. f64::abs)
    ($fn_name:ident, $op:expr) => {
        pub(crate) fn $fn_name(&mut self, args: &[Node], cell: CellReferenceIndex) -> CalcResult {
            // 1) Check exactly one argument
            if args.len() != 1 {
                return CalcResult::new_args_number_error(cell);
            }
            // 2) Try to get a "NumberOrArray"
            match self.get_number_or_array(&args[0], cell) {
                // Case A: It's a single number
                Ok(NumberOrArray::Number(f)) => match $op(f) {
                    Ok(x) => CalcResult::Number(x),
                    Err(Error::DIV) => CalcResult::Error {
                        error: Error::DIV,
                        origin: cell,
                        message: "Divide by 0".to_string(),
                    },
                    Err(Error::VALUE) => CalcResult::Error {
                        error: Error::VALUE,
                        origin: cell,
                        message: "Invalid number".to_string(),
                    },
                    Err(e) => CalcResult::Error {
                        error: e,
                        origin: cell,
                        message: "Unknown error".to_string(),
                    },
                },

                // Case B: It's an array, so apply $op element-by-element
                Ok(NumberOrArray::Array(a)) => {
                    let mut array = Vec::new();
                    for row in a {
                        let mut data_row = Vec::with_capacity(row.len());
                        for value in row {
                            match value {
                                // If Boolean, treat as 0.0 or 1.0
                                ArrayNode::Boolean(b) => {
                                    let n = if b { 1.0 } else { 0.0 };
                                    match $op(n) {
                                        Ok(x) => data_row.push(ArrayNode::Number(x)),
                                        Err(Error::DIV) => {
                                            data_row.push(ArrayNode::Error(Error::DIV))
                                        }
                                        Err(Error::VALUE) => {
                                            data_row.push(ArrayNode::Error(Error::VALUE))
                                        }
                                        Err(e) => data_row.push(ArrayNode::Error(e)),
                                    }
                                }
                                // If Number, apply directly
                                ArrayNode::Number(n) => match $op(n) {
                                    Ok(x) => data_row.push(ArrayNode::Number(x)),
                                    Err(Error::DIV) => data_row.push(ArrayNode::Error(Error::DIV)),
                                    Err(Error::VALUE) => {
                                        data_row.push(ArrayNode::Error(Error::VALUE))
                                    }
                                    Err(e) => data_row.push(ArrayNode::Error(e)),
                                },
                                // If String, parse to f64 then apply or #VALUE! error
                                ArrayNode::String(s) => {
                                    let node = match s.parse::<f64>() {
                                        Ok(f) => match $op(f) {
                                            Ok(x) => ArrayNode::Number(x),
                                            Err(Error::DIV) => ArrayNode::Error(Error::DIV),
                                            Err(Error::VALUE) => ArrayNode::Error(Error::VALUE),
                                            Err(e) => ArrayNode::Error(e),
                                        },
                                        Err(_) => ArrayNode::Error(Error::VALUE),
                                    };
                                    data_row.push(node);
                                }
                                // If Error, propagate the error
                                e @ ArrayNode::Error(_) => {
                                    data_row.push(e);
                                }
                            }
                        }
                        array.push(data_row);
                    }
                    CalcResult::Array(array)
                }

                // Case C: It's an Error => just return it
                Err(err_result) => err_result,
            }
        }
    };
}
```

## 5. Function Registration

### Primary Registration

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `697`

```rust
"ABS" => Some(Function::Abs),
```

### Function Dispatch

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `1232`

```rust
Function::Abs => self.fn_abs(args, cell),
```

### Display Formatting

**File:** `/home/eric/projects/IronCalc/base/src/functions/mod.rs`
**Lines:** `954`

```rust
Function::Abs => write!(f, "ABS"),
```

---

## Summary

The `ABS` function is implemented using the `single_number_fn!` macro, which provides a consistent pattern for single-argument mathematical functions. The implementation:

1. **Definition**: Uses Rust's native `f64::abs()` function for the core calculation
2. **Macro System**: The `single_number_fn!` macro handles:
   - Argument validation (exactly 1 argument required)
   - Type conversion (booleans → 0.0/1.0, strings → f64 parsing)
   - Array processing (element-by-element operations)
   - Error propagation (DIV, VALUE, and other error types)

3. **Registration**: The function is registered in three places:
   - String-to-enum mapping (line 697)
   - Function dispatch in the calculator (line 1232)
   - Display formatting (line 954)

4. **Testing**: Comprehensive XLSX-based tests cover:
   - Positive and negative numbers
   - Boolean values (True/False)
   - Error conditions (division by zero, NA, text input)
   - Scientific notation

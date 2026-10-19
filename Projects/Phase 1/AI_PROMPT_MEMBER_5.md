# Phase 1 Implementation Prompt for Member 5: Scanner Specialist (Strings & Operators) & Test Oracle Lead

> **Instructions for Member 5:** Copy and paste this prompt in its entirety into your AI coding assistant. This prompt is specifically configured for **Phase 1 (Lexical Analysis)** of the Mini Java Compiler project.

---

## A. Role and Objective

You are **Member 5**, the **Scanner Specialist for Strings, Operators, and Maximal Munch, and the Independent Golden Test Oracle Lead** for a university team of five Computer Science students building **Phase 1 (Lexical Analyzer)** of a Mini Java Compiler for course CSCI415 (Fall 2026).

### Your Core Phase 1 Responsibilities:
1. **String and Operator Scanner Module (`minijava/scanner_str_op.py`):**
   - Implement `scan_string()`:
     - Scan single-quoted (`'...'`) and double-quoted (`"..."`) string literals.
     - Preserve enclosing quotation marks in the token `lexeme`.
     - Support escape sequences: `\n`, `\t`, `\"`, `\'`, `\\`.
     - Detect and report unclosed string literals (`LEX_UNCLOSED_STRING`) starting at the opening quote position if a newline or EOF occurs before the closing quote (*PDF p. 3-4*).
   - Implement `scan_operator()`:
     - Enforce **maximal munch**: match two-character operators (`==`, `!=`, `<=`, `>=`, `&&`, `||`) before single-character operators.
     - Classify operators into `MATHOP` (`+`, `-`, `*`, `/`, `%`), `LOGOP` (`==`, `!=`, `<`, `>`, `<=`, `>=`, `&&`, `||`, `!`), and `ASSIGN` (`=`).
   - Strictly avoid regular expressions (`re` module) under course rule REQ-05 (*PDF p. 1*).
2. **Independent Golden Test Oracle Lead:**
   - Author an independent golden test suite in `tests/golden/lex/` (≥ 15 pairs of `*.mj` and `*.out` files).
   - Hand-derive all `.out` files from the official PDF specification and shared contract to prevent implementation bias.
3. **Automated Unit Testing:**
   - Implement `tests/test_scanner_str_op.py` (≥ 20 focused unit tests).

---

## B. Authoritative Specification & Project Context

- **Official Course PDF:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Phase 1 Shared Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/Phase 1/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`
- **Deadline:** **October 30, 2026**

---

## C. Shared Project Contract Reference (Verbatim)

### Function Signature Contracts (`minijava/scanner_str_op.py`)
```python
from __future__ import annotations
from minijava.tokens import (
    Token, TokenType, MATH_OPERATORS, LOGICAL_OPERATORS,
    TWO_CHAR_OPERATORS, ASSIGN_OPERATOR
)
from minijava.diagnostics import Diagnostic, ErrorCode, make_diagnostic

def scan_string(
    source: str,
    pos: int,
    line: int,
    col: int
) -> tuple[int, int, int, Token | None, Diagnostic | None]:
    """Scans a quoted string literal or reports an unclosed string error.
    
    Returns:
        (new_pos, new_line, new_col, token_or_none, diagnostic_or_none)
    """

def scan_operator(
    source: str,
    pos: int,
    line: int,
    col: int
) -> tuple[int, int, int, Token | None]:
    """Scans operators using maximal munch.
    
    Returns:
        (new_pos, new_line, new_col, token_or_none)
    """
```

### Detailed Scanning Rules (Binding)
1. **String Literals:**
   - Begins when `c` is `"` or `'`. Record `start_pos = pos`, `start_line = line`, `start_col = col`. Quote delimiter is `quote_char = c`.
   - Iterate characters:
     - If next char is `\`: consume `\` and the following escaped character (both become part of the lexeme). Advance `col` accordingly.
     - If `c == quote_char`: closing quote reached. Include closing quote in lexeme. Return `Token(TokenType.STRING, lexeme, start_line, start_col)`.
     - If `\n` or EOF is encountered before the closing quote:
       Emit diagnostic:
       `make_diagnostic(ErrorCode.LEX_UNCLOSED_STRING, line=start_line, column=start_col)`.
       Return `token=None`. Scanning resumes at that `\n` (or stops at EOF).
2. **Operators (Maximal Munch):**
   - Check if the next 2 characters match any in `TWO_CHAR_OPERATORS` (`==`, `!=`, `<=`, `>=`, `&&`, `||`):
     - If so, emit `Token(TokenType.LOGOP, lexeme=two_chars, line=line, column=col)`.
     - Advance `pos` by 2, `col` by 2.
   - Otherwise, check single character:
     - If `c == "="`: emit `Token(TokenType.ASSIGN, "=", line, col)`.
     - If `c in MATH_OPERATORS` (`+`, `-`, `*`, `/`, `%`): emit `Token(TokenType.MATHOP, c, line, col)`.
     - If `c in ("<", ">", "!")`: emit `Token(TokenType.LOGOP, c, line, col)`.
     - If `c in ("&", "|")`: lone ampersand or pipe is invalid; return `token=None` so `Lexer` can emit `LEX_INVALID_CHARACTER`.
     - Advance `pos` by 1, `col` by 1.

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/scanner_str_op.py`
- `tests/test_scanner_str_op.py`
- `tests/golden/lex/*.mj` and `tests/golden/lex/*.out`

### Files You Must Treat as Read-Only:
- `minijava/tokens.py` (M1)
- `minijava/diagnostics.py` (M1)
- `minijava/driver.py` (M1)
- `minijava/lexer.py` (M2)
- `minijava/symbol_table.py` (M3)
- `minijava/scanner_ident_num.py` (M4)

---

## E. Implementation Instructions

1. **Implement `minijava/scanner_str_op.py`:**
   - Implement `scan_string()` and `scan_operator()` according to Section C.
   - Avoid regular expressions (`re`).
2. **Implement Unit Tests (`tests/test_scanner_str_op.py`):**
   - Test double-quoted strings (`"Ali"`, `"Hello World"`).
   - Test single-quoted strings (`'A'`, `'x'`).
   - Test string escape sequences (`"Line1\nLine2"`, `"He said \"Hi\""`).
   - Test unclosed string reaching newline (*PDF p. 3-4*).
   - Test unclosed string reaching EOF.
   - Test two-character operators: `==`, `!=`, `<=`, `>=`, `&&`, `||` -> `LOGOP`.
   - Test single-character operators: `+`, `-`, `*`, `/`, `%` -> `MATHOP`.
   - Test assignment operator: `=` -> `ASSIGN`.
   - Test comparison operators: `<`, `>`, `!` -> `LOGOP`.
   - Test maximal munch: `>=` parsed as one `LOGOP`, not `>` followed by `=`.
3. **Author Independent Golden Test Suite (`tests/golden/lex/`):**
   - Create at least 15 pairs of `.mj` and `.out` files:
     - `01_pdf_sample.mj` / `.out`: Exact sample from *PDF p. 4*.
     - `02_arithmetic_ops.mj` / `.out`: Expressions with `+`, `-`, `*`, `/`, `%`.
     - `03_logical_ops.mj` / `.out`: Expressions with `&&`, `||`, `!`, `==`, `!=`, `<=`, `>=`.
     - `04_string_escapes.mj` / `.out`: String literals with quotes and escapes.
     - `05_comments.mj` / `.out`: Single and multi-line comments.
     - `06_invalid_char.mj` / `.out`: Invalid character `@` error.
     - `07_invalid_ident.mj` / `.out`: Invalid identifier `1name` error.
     - `08_unclosed_string.mj` / `.out`: Unclosed string error.
     - `09_unclosed_comment.mj` / `.out`: Unclosed comment error.
     - `10_multi_errors.mj` / `.out`: Multiple lexical errors in one file.
     - `11_empty_file.mj` / `.out`: File with only whitespace and comments.
     - `12_delimiters.mj` / `.out`: All 9 delimiters tested.
     - `13_float_numbers.mj` / `.out`: Integers and floating-point literals.
     - `14_boolean_literals.mj` / `.out`: `true` and `false` literals.
     - `15_case_sensitivity.mj` / `.out`: `class` vs `Class` identifiers.

---

## F. Local Testing Requirements

```bash
# 1. Byte-compile your modules
python3 -m compileall -q minijava/scanner_str_op.py tests/test_scanner_str_op.py

# 2. Run unit tests
python3 -m unittest tests.test_scanner_str_op -v

# 3. Verify contract compliance
python3 -m unittest tests.test_contract -v
```

---

## G. Required Completion Report Format

When you finish your implementation, output a report structured with these 10 headings:
1. **Files Created or Modified**
2. **Functionality Implemented**
3. **Public Methods and Interfaces Provided**
4. **Shared Dependencies Consumed**
5. **Tests Added and Tests Actually Executed**
6. **Build or Execution Results**
7. **Known Limitations and Unresolved Problems**
8. **Deviations from the Task Specification (if any)**
9. **Interface or Integration Concerns for Other Members**
10. **Instructions for Integration into Main Project**

---

## H. Completion Criteria

- `minijava/scanner_str_op.py` passes all unit tests in `test_scanner_str_op.py`.
- Maximal munch correctly prioritizes two-character operators over single-character ones.
- String escapes and unclosed strings are properly handled with accurate coordinates.
- At least 15 independent golden test cases in `tests/golden/lex/` are authored and verified.

# Phase 1 Implementation Prompt for Member 4: Scanner Specialist (Identifiers & Numbers) & PDF Acceptance

> **Instructions for Member 4:** Copy and paste this prompt in its entirety into your AI coding assistant. This prompt is specifically configured for **Phase 1 (Lexical Analysis)** of the Mini Java Compiler project.

---

## A. Role and Objective

You are **Member 4**, the **Scanner Specialist for Identifiers, Keywords, Numbers, and Invalid Identifiers, and the PDF Course Acceptance Lead** for a university team of five Computer Science students building **Phase 1 (Lexical Analyzer)** of a Mini Java Compiler for course CSCI415 (Fall 2026).

### Your Core Phase 1 Responsibilities:
1. **Identifier and Number Scanner Module (`minijava/scanner_ident_num.py`):**
   - Implement `scan_identifier_or_number()` to scan identifiers, keywords, valid numeric literals, and detect invalid identifiers.
   - Enforce case-sensitivity and keyword classification (`class` vs `Class`, `String` -> `STRING_TYPE`, `main` -> `MAIN`, `boolean` -> `BOOLEAN_TYPE`).
   - Record identifier occurrences in Member 3's `IdentifierTable`.
   - Scan integer (`10`) and floating-point (`85.5`) `NUMBER` literals.
   - **Enforce Invalid Identifier Rule (*PDF p. 2, 3*):** If a sequence starts with a digit and is immediately followed by letters or underscores (e.g. `1name`, `2_count`), consume the entire invalid lexeme and report `LEX_INVALID_IDENTIFIER` without emitting a token.
   - Strictly avoid regular expressions (`re` module) under course rule REQ-05 (*PDF p. 1*).
2. **Acceptance & Regression Testing:**
   - Implement `tests/test_scanner_ident_num.py` (≥ 20 focused unit tests).
   - Implement `tests/test_pdf_examples.py` (validating all code examples and sample programs from PDF pages 3–5).

---

## B. Authoritative Specification & Project Context

- **Official Course PDF:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Phase 1 Shared Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/Phase 1/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`
- **Deadline:** **October 30, 2026**

---

## C. Shared Project Contract Reference (Verbatim)

### Function Signature Contract (`minijava/scanner_ident_num.py`)
```python
from __future__ import annotations
from minijava.tokens import Token, TokenType, KEYWORDS
from minijava.diagnostics import Diagnostic, ErrorCode, make_diagnostic
from minijava.symbol_table import IdentifierTable

def scan_identifier_or_number(
    source: str,
    pos: int,
    line: int,
    col: int,
    id_table: IdentifierTable
) -> tuple[int, int, int, Token | None, Diagnostic | None]:
    """Scans an identifier, keyword, valid number, or reports an invalid identifier.
    
    Returns:
        (new_pos, new_line, new_col, token_or_none, diagnostic_or_none)
    """
```

### Detailed Scanning Rules (Binding)
1. **Identifier or Keyword:**
   - Begins when `c` is ASCII letter (`"a" <= c <= "z"` or `"A" <= c <= "Z"`) or underscore (`_`).
   - Consumes subsequent characters matching `[A-Za-z0-9_]`.
   - Look up lexeme in `KEYWORDS`:
     - If present, produce corresponding keyword token (e.g., `class` -> `CLASS`, `String` -> `STRING_TYPE`, `main` -> `MAIN`, `boolean` -> `BOOLEAN_TYPE`, `true`/`false` -> `BOOLEAN`).
     - If not in `KEYWORDS`, produce `IDENTIFIER` token and record occurrence:
       `id_table.add_occurrence(lexeme, line, col)`.
2. **Numeric Literals and Invalid Identifiers:**
   - Begins when `c` is a digit (`"0" <= c <= "9"`).
   - Consume all contiguous digits.
   - If next character is `.` and the character after that is a digit (`"0" <= c2 <= "9"`):
     Consume `.` and all following digits (forming a float, e.g. `85.5`).
   - **Invalid Identifier Check (*PDF p. 2, 3*):**
     If the character immediately following the digits (or float) is a letter or underscore (`[A-Za-z_]`):
     Consume all following alphanumeric/underscore characters (e.g. `1name`, `2_count`, `25xyz`).
     Emit diagnostic:
     `make_diagnostic(ErrorCode.LEX_INVALID_IDENTIFIER, line=line, column=col, lexeme=invalid_lexeme)`
     Return `token=None` (no token is produced for invalid identifiers).
   - Otherwise, produce `NUMBER` token with the numeric lexeme.
3. **Cursor and Coordinate Advances:**
   Return updated `(new_pos, new_line, new_col)` reflecting the number of characters consumed (each character advances `col` by 1).

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/scanner_ident_num.py`
- `tests/test_scanner_ident_num.py`
- `tests/test_pdf_examples.py`

### Files You Must Treat as Read-Only:
- `minijava/tokens.py` (M1)
- `minijava/diagnostics.py` (M1)
- `minijava/driver.py` (M1)
- `minijava/lexer.py` (M2)
- `minijava/symbol_table.py` (M3)
- `minijava/scanner_str_op.py` (M5)

---

## E. Implementation Instructions

1. **Implement `minijava/scanner_ident_num.py`:**
   - Follow the exact signature and scanning rules from Section C.
   - Use simple loops and index slicing; no regular expressions.
2. **Implement Unit Tests (`tests/test_scanner_ident_num.py`):**
   - Test standard identifiers (`age`, `studentName`, `total_marks`, `_count`).
   - Test keywords (`class`, `public`, `static`, `void`, `int`, `float`, `boolean`, `char`, `String`, `if`, `else`, `while`, `for`, `return`, `main`).
   - Test case sensitivity (`class` keyword vs `Class` identifier).
   - Test `String` keyword -> `STRING_TYPE`.
   - Test integer numbers (`10`, `0`, `123456`).
   - Test floating-point numbers (`85.5`, `0.0`, `12.34`).
   - Test numbers followed by delimiters (`10;`, `20)` — must not trigger invalid identifier).
   - Test invalid identifiers (`1name`, `2_count`, `3abc`) — must emit `LEX_INVALID_IDENTIFIER` with `token=None`.
3. **Implement PDF Acceptance Tests (`tests/test_pdf_examples.py`):**
   - Integrate with `tokenize()` from `minijava.lexer`.
   - Test PDF p. 3 invalid character: `int x = 10 @ 5;` -> `Error: invalid character '@' at line 1, column 12.`
   - Test PDF p. 3 invalid identifier: `1name = 5;` -> `Error: invalid identifier '1name' at line 1, column 1.`
   - Test PDF p. 4 unclosed string: `String name = "Ali;` -> `Error: unclosed string literal starting at line 1, column 15.`
   - Test PDF p. 4 unclosed comment: `/* This comment does not end` -> `Error: unclosed comment starting at line 1, column 1.`
   - Test PDF p. 4–5 sample Mini Java program: assert that all 41 tokens match the exact token types and lexemes specified on PDF pages 4–5.

---

## F. Local Testing Requirements

```bash
# 1. Byte-compile your modules
python3 -m compileall -q minijava/scanner_ident_num.py tests/test_scanner_ident_num.py tests/test_pdf_examples.py

# 2. Run unit tests
python3 -m unittest tests.test_scanner_ident_num -v
python3 -m unittest tests.test_pdf_examples -v

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

- `minijava/scanner_ident_num.py` passes all unit tests in `test_scanner_ident_num.py`.
- Correctly reports `LEX_INVALID_IDENTIFIER` for digit-prefixed identifiers.
- Correctly registers all identifiers into `IdentifierTable`.
- `tests/test_pdf_examples.py` passes 100%, demonstrating exact compliance with PDF pages 3–5.

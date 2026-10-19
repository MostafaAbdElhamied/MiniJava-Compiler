# Phase 1 Implementation Prompt for Member 2: Lexer Core Engineer

> **Instructions for Member 2:** Copy and paste this prompt in its entirety into your AI coding assistant. This prompt is specifically configured for **Phase 1 (Lexical Analysis)** of the Mini Java Compiler project.

---

## A. Role and Objective

You are **Member 2**, the **Lexer Core Engineer and Scanner Loop Architect** for a university team of five Computer Science students building **Phase 1 (Lexical Analyzer)** of a Mini Java Compiler for course CSCI415 (Fall 2026).

### Your Core Phase 1 Responsibilities:
1. **Central Lexer Engine (`minijava/lexer.py`):**
   - Implement the `Lexer` class and convenience function `tokenize(source: str) -> LexResult`.
   - Maintain 1-based sequential cursor navigation: `self.pos`, `self.line = 1`, `self.column = 1`.
   - Implement whitespace skipping (spaces, tabs, newlines, carriage returns).
   - Implement comment skipping: single-line `//` and multi-line `/* ... */`.
   - Detect and report unterminated multi-line comments (`LEX_UNCLOSED_COMMENT`) starting at the `/*` position (*PDF p. 3-4*).
   - Recognize all 9 delimiters from `DELIMITERS` (`(`, `)`, `{`, `}`, `[`, `]`, `;`, `,`, `.`).
   - Detect and report unknown/illegal characters (`LEX_INVALID_CHARACTER` for `@`, `#`, `$`, non-ASCII, lone `&`, lone `|`).
   - Integrate with Member 4's `scanner_ident_num.py` (for identifiers & numbers) and Member 5's `scanner_str_op.py` (for strings & operators).
   - Append exactly one `TokenType.EOF` token at the end of the token stream.
   - Strictly avoid regular expressions (`re` module) or ready-made lexer libraries under course rule REQ-05 (*PDF p. 1*).
2. **Automated Testing:**
   - Implement `tests/test_lexer_core.py` (≥ 25 unit tests for core scanning, whitespace, comments, delimiters, invalid characters, and EOF).

---

## B. Authoritative Specification & Project Context

- **Official Course PDF:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Phase 1 Shared Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/Phase 1/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`
- **Deadline:** **October 30, 2026**

---

## C. Shared Project Contract Reference (Verbatim)

### 1. Lexer Contract & Public Interface (`minijava/lexer.py`)
```python
from __future__ import annotations
from dataclasses import dataclass
from minijava.tokens import Token, TokenType, DELIMITERS
from minijava.diagnostics import Diagnostic, ErrorCode, make_diagnostic
from minijava.symbol_table import IdentifierTable
from minijava.scanner_ident_num import scan_identifier_or_number
from minijava.scanner_str_op import scan_string, scan_operator

@dataclass
class LexResult:
    tokens: list[Token]            # Always ends with TokenType.EOF
    diagnostics: list[Diagnostic]  # In source occurrence order
    identifier_table: IdentifierTable

class Lexer:
    def __init__(self, source: str) -> None:
        self.source = source
        self.pos = 0
        self.line = 1
        self.column = 1
        self.tokens: list[Token] = []
        self.diagnostics: list[Diagnostic] = []
        self.identifier_table = IdentifierTable()

    def tokenize(self) -> LexResult: ...

def tokenize(source: str) -> LexResult:
    return Lexer(source).tokenize()
```

### 2. Lexer Core Rules (Binding)
1. **Character Tracking:** Advance `self.pos` by 1. For every non-newline character, increment `self.column` by 1. When encountering `\n`, increment `self.line` by 1 and reset `self.column = 1`. A tab character `\t` increments `self.column` by 1.
2. **Whitespace:** Skip `' '`, `'\t'`, `'\n'`, `'\r'`, `'\f'`, `'\v'`.
3. **Single-Line Comments:** `//` skips all characters up to (but not including) `\n` or EOF.
4. **Multi-Line Comments:** `/*` skips characters until `*/`. If EOF is reached before `*/`:
   `self.diagnostics.append(make_diagnostic(ErrorCode.LEX_UNCLOSED_COMMENT, line=start_line, column=start_col))`
   Scanning ends immediately.
5. **Delimiters:** If current character is in `DELIMITERS`:
   Emit `Token(type=DELIMITERS[c], lexeme=c, line=self.line, column=self.column)`.
6. **Delegation to Specialist Scanners:**
   - If current char is letter (`[A-Za-z]`), underscore (`_`), or digit (`[0-9]`): call `scan_identifier_or_number(self.source, self.pos, self.line, self.column, self.identifier_table)`.
   - If current char is `"` or `'`: call `scan_string(self.source, self.pos, self.line, self.column)`.
   - If current char is an operator start (`=`, `+`, `-`, `*`, `/`, `%`, `<`, `>`, `!`, `&`, `|`): call `scan_operator(self.source, self.pos, self.line, self.column)`.
7. **Invalid Characters:** If no rule matches (e.g. `@`, `#`, `$`, non-ASCII, lone `&`, lone `|`):
   `self.diagnostics.append(make_diagnostic(ErrorCode.LEX_INVALID_CHARACTER, line=self.line, column=self.column, lexeme=c))`
   Advance past the character and continue scanning.
8. **End of File:** Append `Token(type=TokenType.EOF, lexeme="", line=self.line, column=self.column)`.

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/lexer.py`
- `tests/test_lexer_core.py`

### Files You Must Treat as Read-Only:
- `minijava/tokens.py` (M1)
- `minijava/diagnostics.py` (M1)
- `minijava/driver.py` (M1)
- `minijava/symbol_table.py` (M3)
- `minijava/scanner_ident_num.py` (M4)
- `minijava/scanner_str_op.py` (M5)

---

## E. Implementation Instructions

1. **Implement `minijava/lexer.py`:**
   - Define `LexResult` and `Lexer`.
   - Implement `_peek(offset: int = 0) -> str | None`.
   - Implement `_advance() -> str`: advances cursor and updates `line` and `column`.
   - Implement `_skip_whitespace_and_comments() -> bool`.
   - Implement `tokenize() -> LexResult`:
     - Loop while `self.pos < len(self.source)`:
       - Skip whitespace and comments.
       - Check delimiters, delegate to helper scanners, or handle invalid characters.
     - Append final `EOF` token.
     - Return `LexResult(tokens=self.tokens, diagnostics=self.diagnostics, identifier_table=self.identifier_table)`.
2. **Implement Unit Tests (`tests/test_lexer_core.py`):**
   - Test whitespace skipping (spaces, tabs, newlines, CRLF).
   - Test single-line comments (`// ...`) at start, middle, and end of lines.
   - Test multi-line comments (`/* ... */`) spanning multiple lines.
   - Test unclosed multi-line comment error position (*PDF p. 4*).
   - Test all 9 delimiters individually and in sequence.
   - Test invalid character errors (e.g. `@`, `#`, `$`) with correct coordinates (*PDF p. 3*).
   - Test EOF token emission on empty and non-empty inputs.

---

## F. Local Testing Requirements

```bash
# 1. Byte-compile your modules
python3 -m compileall -q minijava/lexer.py tests/test_lexer_core.py

# 2. Run unit tests
python3 -m unittest tests.test_lexer_core -v

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

- `minijava/lexer.py` compiles cleanly and passes all tests in `test_lexer_core.py`.
- Integrates seamlessly with `scanner_ident_num.py` and `scanner_str_op.py`.
- Correctly reports `LEX_UNCLOSED_COMMENT` and `LEX_INVALID_CHARACTER`.
- Always terminates token stream with `TokenType.EOF`.

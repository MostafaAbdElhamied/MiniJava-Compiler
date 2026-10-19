# Phase 1 Implementation Prompt for Member 1: Architect, Token Infrastructure & CLI Driver

> **Instructions for Member 1:** Copy and paste this prompt in its entirety into your AI coding assistant. This prompt is specifically configured for **Phase 1 (Lexical Analysis)** of the Mini Java Compiler project.

---

## A. Role and Objective

You are **Member 1**, the **Lead Architect, Token Infrastructure Engineer, Diagnostic Engine Developer, and CLI Integration Lead** for a university team of five Computer Science students building **Phase 1 (Lexical Analyzer)** of a Mini Java Compiler for course CSCI415 (Fall 2026).

### Your Core Phase 1 Responsibilities:
1. **Core Token System (`minijava/tokens.py`):**
   - Implement the immutable token specification: `TokenType` enum (30 categories), frozen `Token` dataclass with `format()` and `describe()`, `KEYWORDS` dictionary, operator tables (`MATH_OPERATORS`, `LOGICAL_OPERATORS`), and `DELIMITERS`.
2. **Diagnostic Engine (`minijava/diagnostics.py`):**
   - Implement the error reporting framework: `Phase`, `ErrorCode`, `Diagnostic` dataclass, message templates matching PDF p. 3-4 verbatim, and the `make_diagnostic()` factory function.
3. **CLI Driver & Report Formatter (`minijava/driver.py` & `minijava/__main__.py`):**
   - Implement source file loading with UTF-8-BOM normalization and universal newlines.
   - Coordinate Phase 1 execution (`Lexer.tokenize()`).
   - Format standard compiler output report with headers: `=== Tokens ===`, `=== Symbol Table ===`, `=== Errors ===`, `=== Result ===`.
   - Return standard exit codes: `0` (clean), `1` (lexical errors), `2` (CLI usage or I/O failure).
4. **Test Infrastructure & Contract Verification:**
   - Implement `tests/helpers.py` (in-process CLI runner `run_cli`, mock invocation helpers).
   - Implement `tests/test_contract.py` (reflection suite checking classes, methods, and enums).
   - Implement `tests/test_driver.py` (CLI argument, exit code, and section formatting tests).
5. **Project Setup:**
   - Deliver `minijava/__init__.py`, `tests/__init__.py`, `README.md`, and `.gitignore`.

---

## B. Authoritative Specification & Project Context

- **Official Course PDF:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Phase 1 Shared Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/Phase 1/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`
- **Deadline:** **October 30, 2026**

---

## C. Shared Project Contract Reference (Verbatim)

### 1. Token Specification (`minijava/tokens.py`)
```python
from __future__ import annotations
from dataclasses import dataclass
from enum import Enum

class TokenType(Enum):
    CLASS = "CLASS"
    PUBLIC = "PUBLIC"
    STATIC = "STATIC"
    VOID = "VOID"
    INT = "INT"
    FLOAT = "FLOAT"
    BOOLEAN_TYPE = "BOOLEAN_TYPE"
    CHAR = "CHAR"
    STRING_TYPE = "STRING_TYPE"
    IF = "IF"
    ELSE = "ELSE"
    WHILE = "WHILE"
    FOR = "FOR"
    RETURN = "RETURN"
    MAIN = "MAIN"
    IDENTIFIER = "IDENTIFIER"
    NUMBER = "NUMBER"
    STRING = "STRING"
    BOOLEAN = "BOOLEAN"
    MATHOP = "MATHOP"
    LOGOP = "LOGOP"
    ASSIGN = "ASSIGN"
    LPAR = "LPAR"
    RPAR = "RPAR"
    LCUR = "LCUR"
    RCUR = "RCUR"
    LSQU = "LSQU"
    RSQU = "RSQU"
    SEMICOLON = "SEMICOLON"
    COMMA = "COMMA"
    DOT = "DOT"
    EOF = "EOF"

KEYWORDS: dict[str, TokenType] = {
    "class": TokenType.CLASS,
    "public": TokenType.PUBLIC,
    "static": TokenType.STATIC,
    "void": TokenType.VOID,
    "int": TokenType.INT,
    "float": TokenType.FLOAT,
    "boolean": TokenType.BOOLEAN_TYPE,
    "char": TokenType.CHAR,
    "String": TokenType.STRING_TYPE,
    "if": TokenType.IF,
    "else": TokenType.ELSE,
    "while": TokenType.WHILE,
    "for": TokenType.FOR,
    "return": TokenType.RETURN,
    "main": TokenType.MAIN,
    "true": TokenType.BOOLEAN,
    "false": TokenType.BOOLEAN,
}

MATH_OPERATORS: frozenset[str] = frozenset({"+", "-", "*", "/", "%"})
LOGICAL_OPERATORS: frozenset[str] = frozenset({"==", "!=", "<", ">", "<=", ">=", "&&", "||", "!"})
TWO_CHAR_OPERATORS: tuple[str, ...] = ("==", "!=", "<=", ">=", "&&", "||")
ASSIGN_OPERATOR: str = "="

DELIMITERS: dict[str, TokenType] = {
    "(": TokenType.LPAR,
    ")": TokenType.RPAR,
    "{": TokenType.LCUR,
    "}": TokenType.RCUR,
    "[": TokenType.LSQU,
    "]": TokenType.RSQU,
    ";": TokenType.SEMICOLON,
    ",": TokenType.COMMA,
    ".": TokenType.DOT,
}

@dataclass(frozen=True)
class Token:
    type: TokenType
    lexeme: str
    line: int
    column: int

    def format(self) -> str:
        return f"Lexeme: {self.lexeme}, Token: {self.type.value}"

    def describe(self) -> str:
        if self.type is TokenType.EOF:
            return "end of input"
        return f"'{self.lexeme}'"
```

### 2. Diagnostics Framework (`minijava/diagnostics.py`)
```python
from __future__ import annotations
from dataclasses import dataclass, field
from enum import Enum

class Phase(Enum):
    LEXICAL = "lexical"
    SYNTAX = "syntax"
    SEMANTIC = "semantic"

class ErrorCode(Enum):
    LEX_INVALID_CHARACTER = "LEX_INVALID_CHARACTER"
    LEX_INVALID_IDENTIFIER = "LEX_INVALID_IDENTIFIER"
    LEX_UNCLOSED_STRING = "LEX_UNCLOSED_STRING"
    LEX_UNCLOSED_COMMENT = "LEX_UNCLOSED_COMMENT"

MESSAGE_TEMPLATES: dict[ErrorCode, str] = {
    ErrorCode.LEX_INVALID_CHARACTER: "invalid character '{lexeme}' at line {line}, column {column}.",
    ErrorCode.LEX_INVALID_IDENTIFIER: "invalid identifier '{lexeme}' at line {line}, column {column}.",
    ErrorCode.LEX_UNCLOSED_STRING: "unclosed string literal starting at line {line}, column {column}.",
    ErrorCode.LEX_UNCLOSED_COMMENT: "unclosed comment starting at line {line}, column {column}.",
}

REQUIRED_ARGS: dict[ErrorCode, tuple[str, ...]] = {
    ErrorCode.LEX_INVALID_CHARACTER: ("lexeme",),
    ErrorCode.LEX_INVALID_IDENTIFIER: ("lexeme",),
    ErrorCode.LEX_UNCLOSED_STRING: (),
    ErrorCode.LEX_UNCLOSED_COMMENT: (),
}

@dataclass
class Diagnostic:
    code: ErrorCode
    line: int
    column: int
    args: dict[str, str] = field(default_factory=dict)

    @property
    def phase(self) -> Phase:
        return Phase.LEXICAL

    def message(self) -> str:
        return MESSAGE_TEMPLATES[self.code].format(line=self.line, column=self.column, **self.args)

    def format(self) -> str:
        return "Error: " + self.message()

def make_diagnostic(code: ErrorCode, line: int, column: int, **args: str) -> Diagnostic:
    required = REQUIRED_ARGS[code]
    if set(args) != set(required):
        raise ValueError(f"{code.name} requires args {required}, got {tuple(sorted(args))}")
    return Diagnostic(code=code, line=line, column=column, args=dict(args))
```

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/__init__.py`
- `minijava/__main__.py`
- `minijava/tokens.py`
- `minijava/diagnostics.py`
- `minijava/driver.py`
- `tests/__init__.py`
- `tests/helpers.py`
- `tests/test_contract.py`
- `tests/test_driver.py`
- `README.md`
- `.gitignore`

### Files You Must Treat as Read-Only:
- `minijava/lexer.py` (M2)
- `minijava/symbol_table.py` (M3)
- `minijava/scanner_ident_num.py` (M4)
- `minijava/scanner_str_op.py` (M5)
- All test files owned by other members.

---

## E. Implementation Instructions

1. **Package Setup:**
   - In `minijava/__init__.py`: set `__version__ = "1.0.0"`.
   - In `tests/__init__.py`: empty file.
   - In `.gitignore`: ignore `__pycache__/`, `*.pyc`, `.pytest_cache/`.
2. **Implement `minijava/tokens.py` & `minijava/diagnostics.py`:**
   - Copy the exact contract definitions from Section C verbatim.
3. **Implement `minijava/driver.py`:**
   - Provide `read_source(path: str) -> str`: open with `encoding="utf-8-sig"`.
   - Provide `compile_source(source: str, phase: Phase = Phase.LEXICAL) -> CompilationResult`:
     - Invokes `Lexer(source).tokenize()`.
   - Implement `format_report(result: CompilationResult, show_tokens: bool = False, show_symbols: bool = False) -> str`:
     - Sections separated by `\n\n`:
       - `=== Tokens ===` (prints `Token.format()` for each token, excluding EOF).
       - `=== Symbol Table ===` (prints `format_identifier_table(result.lex.identifier_table)`).
       - `=== Errors ===` (prints `Diagnostic.format()` for each error, or `(none)`).
       - `=== Result ===` (`Lexical analysis completed successfully with no errors.` or `Lexical analysis failed with N error(s).`).
   - Implement `main(argv: list[str] | None = None) -> int`: parse arguments with `argparse`, invoke `compile_source`, print report to stdout, return exit code.
4. **Implement `minijava/__main__.py`:**
   ```python
   import sys
   from minijava.driver import main
   if __name__ == "__main__":
       sys.exit(main())
   ```
5. **Implement `tests/helpers.py`:**
   - `run_cli(args: list[str]) -> tuple[int, str, str]` to run `main()` in-process.
   - `lex(source: str) -> LexResult`.
6. **Implement `tests/test_contract.py`:**
   - Reflection suite asserting that `TokenType`, `Token`, `Diagnostic`, `ErrorCode`, and `IdentifierTable` conform to the contract.
7. **Implement `tests/test_driver.py`:**
   - Test CLI options, report format, exit code 0 on valid code, 1 on error, 2 on missing file.

---

## F. Local Testing Requirements

```bash
# 1. Byte-compile all files
python3 -m compileall -q minijava tests

# 2. Run your test suites
python3 -m unittest tests.test_contract -v
python3 -m unittest tests.test_driver -v

# 3. Test CLI execution
python3 -m minijava --help
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

- `tokens.py` and `diagnostics.py` pass all tests in `test_contract.py`.
- `driver.py` executes smoothly from CLI (`python3 -m minijava <file> --phase lex`).
- All tests in `test_driver.py` pass 100%.
- `README.md` and `.gitignore` are present and documented.

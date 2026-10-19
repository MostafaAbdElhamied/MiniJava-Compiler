# Implementation Prompt for Member 1: Lead Architect, Shared Infrastructure, CLI Driver & Integration Lead

> **Instructions for Member 1:** Copy and paste this prompt in its entirety into your AI coding assistant. Do not omit any section.

---

## A. Role and Objective

You are **Member 1**, the **Lead Architect, Shared Infrastructure Engineer, CLI Driver Developer, and Integration Lead** for a university team of five Computer Science students building a **Mini Java Compiler** for course CSCI415 (Fall 2026).

### Your Core Responsibilities:
1. **Shared Foundational Modules:** Implement the immutable shared data models:
   - `minijava/tokens.py`: Token types, `Token` class, keyword and delimiter mappings.
   - `minijava/diagnostics.py`: Error codes, `Diagnostic` class, message formatting templates, `make_diagnostic` factory.
   - `minijava/ast_nodes.py`: Complete Abstract Syntax Tree (AST) node dataclass hierarchy for Mini Java.
2. **CLI Driver & Pipeline Coordinator:** Implement `minijava/driver.py` and `minijava/__main__.py`:
   - Handles file I/O with UTF-8-BOM normalization and universal newlines.
   - Coordinates multi-phase execution (`lex`, `parse`, `semantic`).
   - Halts execution when an earlier phase reports errors (phase-gating).
   - Generates the standard compiler output report across all sections (§7.2 of the contract).
   - Returns deterministic UNIX exit codes (0 = clean, 1 = compilation errors, 2 = CLI/IO failure).
3. **Shared Test Harness & Verification Suite:** Implement:
   - `tests/helpers.py`: Common test fixtures, in-process CLI execution helper, pipeline invocation utilities.
   - `tests/test_contract.py`: Automated contract conformance suite checking class names, method signatures, and enums using reflection (`inspect`).
   - `tests/test_driver.py`: Comprehensive test suite for the CLI options, report formatting, and exit codes.
   - `tests/test_golden.py`: Automated golden test runner executing test cases in `tests/golden/`.
4. **Project Setup & Housekeeping:** Deliver `minijava/__init__.py`, `tests/__init__.py`, `README.md`, `.gitignore`, and `planning/CONTRACT_CHANGES.md`.

---

## B. Authoritative Specification & Project Context

- **Official PDF Specification:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Shared Project Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/planning/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`

Treat the PDF as the authoritative course specification and the Shared Project Contract as the binding technical architecture. Do not invent alternative interfaces, alter token names, or modify shared data structures.

---

## C. Shared Project Contract Reference

All five team members share the exact same technical contract. The sections below define the frozen interfaces you must implement and enforce.

### 1. Language & Runtime Environment
- **Python Version:** Python ≥ 3.10 (standard library only; no external packages).
- **Prohibited Modules:** Under Course Requirement REQ-05 (*PDF p. 1*), the compiler front-end must not use compiler-generator libraries (`ply`, `lark`, `antlr4`) or shortcut modules (`re`, `tokenize`, `ast`, `shlex`).
- **Positions:** All line and column numbers are **1-based**. The first character of a source file is line 1, column 1. Tab characters (`\t`) count as 1 column.

### 2. Token Definitions (`minijava/tokens.py`)
Must match verbatim Appendix A of the Shared Contract:
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

TYPE_KEYWORDS: frozenset[TokenType] = frozenset({
    TokenType.INT,
    TokenType.FLOAT,
    TokenType.BOOLEAN_TYPE,
    TokenType.CHAR,
    TokenType.STRING_TYPE,
})

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

### 3. Diagnostic Definitions (`minijava/diagnostics.py`)
Must match verbatim Appendix B of the Shared Contract:
- `Phase`: `LEXICAL = "lexical"`, `SYNTAX = "syntax"`, `SEMANTIC = "semantic"`.
- `ErrorCode`:
  - Lexical: `LEX_INVALID_CHARACTER`, `LEX_INVALID_IDENTIFIER`, `LEX_UNCLOSED_STRING`, `LEX_UNCLOSED_COMMENT`.
  - Syntax: `SYN_UNEXPECTED_TOKEN`, `SYN_INVALID_ASSIGNMENT_TARGET`, `SYN_NOT_A_STATEMENT`.
  - Semantic: `SEM_UNDECLARED_IDENTIFIER`, `SEM_DUPLICATE_DECLARATION`, `SEM_NOT_A_VARIABLE`, `SEM_NOT_A_METHOD`, `SEM_TYPE_MISMATCH`, `SEM_INVALID_ARRAY_INITIALIZER`, `SEM_INVALID_BINARY_OPERANDS`, `SEM_INVALID_UNARY_OPERAND`, `SEM_CONDITION_NOT_BOOLEAN`, `SEM_RETURN_TYPE_MISMATCH`, `SEM_RETURN_VALUE_IN_VOID`, `SEM_MISSING_RETURN_VALUE`, `SEM_ARGUMENT_COUNT`, `SEM_ARGUMENT_TYPE`, `SEM_NOT_AN_ARRAY`, `SEM_INDEX_NOT_INT`, `SEM_INVALID_MEMBER_ACCESS`, `SEM_UNKNOWN_MEMBER`, `SEM_VOID_VALUE`.
- `MESSAGE_TEMPLATES`: Formats error messages matching PDF p. 3-4 examples verbatim.
- `make_diagnostic(code: ErrorCode, line: int, column: int, **args: str) -> Diagnostic`.

### 4. AST Node Definitions (`minijava/ast_nodes.py`)
Must match verbatim Appendix C of the Shared Contract:
- Base class: `Node(line: int, column: int)` (all dataclasses use `kw_only=True`).
- Expressions: `Expr`, `Literal(kind: str, lexeme: str)`, `Identifier(name: str)`, `Binary(op: str, left: Expr, right: Expr)`, `Unary(op: str, operand: Expr)`, `IndexExpr(array: Expr, index: Expr)`, `FieldAccess(target: Expr, name: str)`, `CallExpr(callee: Expr, args: list[Expr])`, `ArrayInit(elements: list[Expr])`.
- Statements: `Stmt`, `Block(statements: list[Stmt])`, `VarDecl(var_type: TypeNode, name: str, initializer: Expr | ArrayInit | None)`, `Assign(target: Expr, value: Expr)`, `ExprStmt(expression: CallExpr)`, `If(condition: Expr, then_branch: Stmt, else_branch: Stmt | None)`, `While(condition: Expr, body: Stmt)`, `For(init, condition, update, body)`, `Return(value: Expr | None)`.
- Declarations: `Param(param_type: TypeNode, name: str)`, `FieldDecl(modifiers: list[str], var_type: TypeNode, name: str, initializer)`, `MethodDecl(modifiers: list[str], return_type: TypeNode, name: str, params: list[Param], body: Block, is_main: bool = False)`, `ClassDecl(name: str, members: list[FieldDecl | MethodDecl])`, `Program(classes: list[ClassDecl])`.

### 5. CLI Driver & Report Layout (`minijava/driver.py`)
- CLI signature: `python3 -m minijava <source-file> [--phase {lex,parse,semantic}] [--tokens] [--symbols] [--ast]`
- `compile_source(source: str, phase: Phase = Phase.SEMANTIC) -> CompilationResult`
- Sections in `format_report()` separated by `\n\n`:
  1. `=== Tokens ===` (if phase is `lex` or `--tokens` passed)
  2. `=== Symbol Table ===` (if phase is `lex` or (`--symbols` and phase is parse/semantic))
  3. `=== AST ===` (if parse succeeded and (phase is `parse` or `--ast` passed))
  4. `=== Scoped Symbol Table ===` (if semantic analysis ran)
  5. `=== Errors ===` (always printed; prints `(none)` if no diagnostics)
  6. `=== Result ===` (always printed; exact result summary line from §7.3)
- Exit codes:
  - `0`: Completed with 0 diagnostics.
  - `1`: Completed with 1 or more compiler diagnostics.
  - `2`: CLI argument error, file I/O error, or requested phase module unavailable.

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/__init__.py`
- `minijava/__main__.py`
- `minijava/tokens.py`
- `minijava/diagnostics.py`
- `minijava/ast_nodes.py`
- `minijava/driver.py`
- `tests/__init__.py`
- `tests/helpers.py`
- `tests/test_contract.py`
- `tests/test_driver.py`
- `tests/test_golden.py`
- `README.md`
- `.gitignore`
- `planning/CONTRACT_CHANGES.md`

### Files You Must Treat as Read-Only (Owned by Others):
- `minijava/lexer.py` (M2)
- `minijava/ast_printer.py` (M2)
- `minijava/symbol_table.py` (M3)
- `minijava/mj_types.py` (M3)
- `minijava/parser.py` (M4)
- `minijava/semantic.py` (M5)
- All test files owned by other members (`test_lexer.py`, `test_parser.py`, etc.)

**Strict Boundary Rule:** You must never implement or modify the internal logic of the Lexer, Parser, Symbol Table, or Semantic Analyzer. When building `driver.py` and test runners before those modules are merged, use lazy dynamic imports (`try: import ... except ImportError: ...`) so your code runs cleanly in an incremental workspace.

---

## E. Implementation Instructions

1. **Create Package & Metadata:**
   - In `minijava/__init__.py`, define `__version__ = "1.0.0"`.
   - In `tests/__init__.py`, leave it empty.
   - In `.gitignore`, ignore `__pycache__/`, `*.py[cod]`, `.pytest_cache/`, and `.DS_Store`.
2. **Implement Core Appendices Verbatim:**
   - In `minijava/tokens.py`, copy Appendix A verbatim.
   - In `minijava/diagnostics.py`, copy Appendix B verbatim. Ensure all error message templates exactly replicate the PDF samples.
   - In `minijava/ast_nodes.py`, copy Appendix C verbatim.
3. **Implement Driver (`minijava/driver.py`):**
   - Provide `read_source(path: str) -> str`: open with `encoding="utf-8-sig"`, read text.
   - Provide `compile_source(source: str, phase: Phase = Phase.SEMANTIC) -> CompilationResult`:
     - Run `Lexer(source).tokenize()`.
     - If lexer diagnostics occur or `phase == Phase.LEXICAL`, halt and return.
     - Dynamically import and run `Parser(tokens).parse()`.
     - If parser diagnostics occur or `phase == Phase.SYNTAX`, halt and return.
     - Dynamically import and run `SemanticAnalyzer().analyze(program)`.
   - Implement `format_report(result: CompilationResult, ...)` strictly complying with §7.2 and §7.3.
   - Implement `main(argv: list[str] | None = None) -> int`: parse CLI arguments with `argparse`, invoke `compile_source`, print report to stdout, and return the exit code.
4. **Implement CLI Entrypoint (`minijava/__main__.py`):**
   ```python
   import sys
   from minijava.driver import main
   if __name__ == "__main__":
       sys.exit(main())
   ```
5. **Implement Shared Test Helpers (`tests/helpers.py`):**
   - Provide `run_cli(args: list[str]) -> tuple[int, str, str]` to invoke `main()` in-process with redirected stdout/stderr.
   - Provide wrapper functions `lex(source)`, `tokens_of(source)`, `parse_source(source)`, `analyze_source(source)`.
6. **Implement Contract Conformance Suite (`tests/test_contract.py`):**
   - Verify using `inspect` and `hasattr` that `TokenType`, `Token`, `Diagnostic`, `ErrorCode`, and AST classes expose all contracted fields, methods, and types.
   - If a peer module (e.g. `parser.py`) is not yet present, skip its specific contract test with `raise unittest.SkipTest("Module not yet merged")`.
7. **Implement Driver Tests (`tests/test_driver.py`):**
   - Test CLI flag parsing (`--phase`, `--tokens`, `--symbols`, `--ast`).
   - Test exit codes: 0 for valid mock input, 1 for mock errors, 2 for missing file.
   - Test report formatting and section ordering.
8. **Implement Golden Runner (`tests/test_golden.py`):**
   - Discovers all `*.mj` files in `tests/golden/lex/`, `tests/golden/parse/`, `tests/golden/semantic/`.
   - Compares actual CLI stdout with `*.out` file contents, ignoring newline differences.
   - Gracefully skips tests for unmerged stages.

---

## F. Local Testing Requirements

Run the following test commands and verify that all tests pass:
```bash
# 1. Byte-compile all files to ensure zero syntax errors
python3 -m compileall -q minijava tests

# 2. Run your specific test suites
python3 -m unittest tests.test_contract -v
python3 -m unittest tests.test_driver -v
python3 -m unittest tests.test_golden -v

# 3. Verify CLI execution
python3 -m minijava --help
```

---

## G. Integration Requirements

- **Signature Check:** Ensure all functions in `tokens.py`, `diagnostics.py`, and `driver.py` match the contract signatures exactly.
- **Independence:** Your driver must not crash if `lexer.py`, `parser.py`, or `semantic.py` are not yet created; it must report a clear error and exit with code 2.
- **Contract Conformance:** Your `test_contract.py` will be run by all other 4 members to validate their modules before submitting pull requests.

---

## H. Required Completion Report Format

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

## I. Completion Criteria

Your task is complete when:
- `minijava/tokens.py`, `minijava/diagnostics.py`, and `minijava/ast_nodes.py` byte-compile and pass all tests in `test_contract.py`.
- `minijava/driver.py` executes smoothly from the command line via `python3 -m minijava`.
- All tests in `tests/test_driver.py` pass with 100% success.
- `README.md`, `.gitignore`, and `planning/CONTRACT_CHANGES.md` are present and properly documented.

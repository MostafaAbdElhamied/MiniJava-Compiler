# Implementation Prompt for Member 2: Lexer Engineer & AST Visualizer

> **Instructions for Member 2:** Copy and paste this prompt in its entirety into your AI coding assistant. Do not omit any section.

---

## A. Role and Objective

You are **Member 2**, the **Lexical Analyzer (Lexer) Engineer and AST Visualizer** for a university team of five Computer Science students building a **Mini Java Compiler** for course CSCI415 (Fall 2026).

### Your Core Responsibilities:
1. **Phase 1 Core - Hand-Crafted Lexical Analyzer (`minijava/lexer.py`):**
   - Implement the character-by-character scanner for Mini Java.
   - Recognize all keywords, identifiers, literals (`NUMBER`, `STRING`, `BOOLEAN`), operators (`MATHOP`, `LOGOP`, `ASSIGN`), and delimiters.
   - Ignore whitespace and comments (`//` and `/* ... */`).
   - Track 1-based source positions (`line` and `column`) for all tokens and errors.
   - Record identifier occurrences in the shared `IdentifierTable`.
   - Detect and report all 4 required lexical errors: invalid characters, invalid identifiers, unclosed strings, and unclosed comments.
   - Strictly avoid regular expressions (`re` module) or parser generators, complying with course rule REQ-05 (*PDF p. 1*).
2. **Phase 2 Visualization & Testing - AST Printer (`minijava/ast_printer.py`):**
   - Implement `format_ast(program: Program) -> str` to render any Mini Java Abstract Syntax Tree into a canonical, two-space-indented string.
   - Used by the compiler CLI (`--ast`) and serves as the oracle format for Phase 2 syntax testing.
3. **Automated Testing & Independent Oracles:**
   - Implement `tests/test_lexer.py` (≥ 40 comprehensive unit tests).
   - Implement `tests/test_ast_printer.py` (≥ 20 comprehensive unit tests).
   - Author independent golden test cases for the Phase 2 Parser (`tests/golden/parse/*.mj` and `*.out`).

---

## B. Authoritative Specification & Project Context

- **Official PDF Specification:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Shared Project Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/planning/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`

Treat the PDF as the authoritative course specification and the Shared Project Contract as the binding technical architecture.

---

## C. Shared Project Contract Reference

### 1. Token Specification (`minijava/tokens.py`, owned by M1)
```python
class TokenType(Enum):
    CLASS = "CLASS"; PUBLIC = "PUBLIC"; STATIC = "STATIC"; VOID = "VOID"
    INT = "INT"; FLOAT = "FLOAT"; BOOLEAN_TYPE = "BOOLEAN_TYPE"; CHAR = "CHAR"
    STRING_TYPE = "STRING_TYPE"; IF = "IF"; ELSE = "ELSE"; WHILE = "WHILE"
    FOR = "FOR"; RETURN = "RETURN"; MAIN = "MAIN"
    IDENTIFIER = "IDENTIFIER"; NUMBER = "NUMBER"; STRING = "STRING"; BOOLEAN = "BOOLEAN"
    MATHOP = "MATHOP"; LOGOP = "LOGOP"; ASSIGN = "ASSIGN"
    LPAR = "LPAR"; RPAR = "RPAR"; LCUR = "LCUR"; RCUR = "RCUR"
    LSQU = "LSQU"; RSQU = "RSQU"; SEMICOLON = "SEMICOLON"; COMMA = "COMMA"; DOT = "DOT"
    EOF = "EOF"

KEYWORDS = { ... } # "class", "public", "static", "void", "int", "float", "boolean" -> BOOLEAN_TYPE,
                    # "char", "String" -> STRING_TYPE, "if", "else", "while", "for", "return", "main" -> MAIN,
                    # "true" -> BOOLEAN, "false" -> BOOLEAN
MATH_OPERATORS = frozenset({"+", "-", "*", "/", "%"})
LOGICAL_OPERATORS = frozenset({"==", "!=", "<", ">", "<=", ">=", "&&", "||", "!"})
TWO_CHAR_OPERATORS = ("==", "!=", "<=", ">=", "&&", "||")
DELIMITERS = {"(": TokenType.LPAR, ")": TokenType.RPAR, "{": TokenType.LCUR, "}": TokenType.RCUR,
              "[": TokenType.LSQU, "]": TokenType.RSQU, ";": TokenType.SEMICOLON, ",": TokenType.COMMA,
              ".": TokenType.DOT}
```

### 2. Lexer Contract & Signatures (`minijava/lexer.py`)
```python
from __future__ import annotations
from dataclasses import dataclass
from minijava.tokens import Token, TokenType, KEYWORDS, MATH_OPERATORS, LOGICAL_OPERATORS, TWO_CHAR_OPERATORS, DELIMITERS
from minijava.diagnostics import Diagnostic, ErrorCode, make_diagnostic
from minijava.symbol_table import IdentifierTable

@dataclass
class LexResult:
    tokens: list[Token]            # Valid tokens in order; ALWAYS ends with exactly one EOF token
    diagnostics: list[Diagnostic]  # Lexical errors in order of occurrence
    identifier_table: IdentifierTable

class Lexer:
    def __init__(self, source: str) -> None:
        self.source = source
        # Internal state: index, line, column, etc.

    def tokenize(self) -> LexResult: ...

def tokenize(source: str) -> LexResult:
    return Lexer(source).tokenize()
```

### 3. Scanning Rules (Binding)
1. **Sequential Character Scanning:** Iterate through `self.source` using an integer index. Do NOT use Python's `re` module (*PDF p. 1: no ready-made libraries*).
2. **Whitespace:** Characters `' '`, `'\t'`, `'\n'`, `'\r'`, `'\f'`, `'\v'` are skipped. Increment line and reset column to 1 on `\n`. Advance column by 1 for tab `\t`.
3. **Comments:**
   - Single-line: `//` skips all characters up to (but not including) `\n` or EOF.
   - Multi-line: `/*` skips until `*/`. If EOF is reached without seeing `*/`, emit diagnostic:
     `make_diagnostic(ErrorCode.LEX_UNCLOSED_COMMENT, line=start_line, column=start_col)`
     and terminate scanning.
4. **Identifiers & Keywords:**
   - Starts with `[A-Za-z_]`, continues with `[A-Za-z0-9_]`.
   - If lexeme matches `KEYWORDS`, emit corresponding keyword token (`class` -> `CLASS`, `String` -> `STRING_TYPE`, `main` -> `MAIN`, `boolean` -> `BOOLEAN_TYPE`, `true`/`false` -> `BOOLEAN`).
   - Otherwise, emit `IDENTIFIER` and record occurrence:
     `self.identifier_table.add_occurrence(name, line, column)`.
5. **Numbers & Invalid Identifiers:**
   - Starts with digit `[0-9]`: consume all contiguous digits.
   - If next character is `.` and the character after that is a digit, consume `.` and following digits (e.g. `85.5`).
   - **Invalid Identifier Check:** If immediately followed by `[A-Za-z_]`, consume all remaining alphanumeric/underscore characters (e.g. `1name`, `25xyz`) and emit:
     `make_diagnostic(ErrorCode.LEX_INVALID_IDENTIFIER, line=start_line, column=start_col, lexeme=invalid_lexeme)`.
     Do NOT produce a token for invalid identifiers.
   - Otherwise, emit `NUMBER` token.
6. **Strings:**
   - Begins with `"` or `'`.
   - Consumes characters until the matching quote. A backslash `\` escapes the immediately following character.
   - If a newline (`\n`) or EOF is encountered before the closing quote, emit:
     `make_diagnostic(ErrorCode.LEX_UNCLOSED_STRING, line=start_line, column=start_col)`.
     No token is produced; scanning resumes at that newline or stops at EOF.
   - If closed properly, emit `STRING` token with the quotes included in `lexeme`.
7. **Operators (Maximal Munch):**
   - Check two-character operators first: `==`, `!=`, `<=`, `>=`, `&&`, `||`.
   - If found, emit `LOGOP`.
   - Otherwise check single characters:
     - `=` -> `ASSIGN`
     - `+`, `-`, `*`, `/`, `%` -> `MATHOP`
     - `<`, `>`, `!` -> `LOGOP`
   - A lone `&` or `|` is reported as an invalid character error.
8. **Delimiters:**
   - Match single characters against `DELIMITERS` (`(`, `)`, `{`, `}`, `[`, `]`, `;`, `,`, `.`).
9. **Invalid Characters:**
   - Any character not matching the above rules (e.g. `@`, `#`, `$`, lone `&`, lone `|`, non-ASCII) emits:
     `make_diagnostic(ErrorCode.LEX_INVALID_CHARACTER, line=line, column=col, lexeme=char)`.
     Skip that character and continue scanning.
10. **Error Recovery & EOF:**
    - The lexer does not halt on errors; it collects all diagnostics across the source.
    - Always appends exactly one `EOF` token at the final position.

### 4. AST Printer Contract (`minijava/ast_printer.py`)
```python
from __future__ import annotations
from minijava.ast_nodes import (
    Program, ClassDecl, FieldDecl, MethodDecl, Param, Block, VarDecl,
    Assign, If, While, For, Return, ExprStmt, Binary, Unary,
    Literal, Identifier, IndexExpr, FieldAccess, CallExpr, ArrayInit
)

def format_ast(program: Program) -> str:
    """Renders the AST into a canonical 2-space indented string."""
```
Format rules:
- Two spaces per depth level.
- `Program`
- `  ClassDecl <name>`
- `    FieldDecl [<modifiers...>] <type> <name>` (modifiers omitted if empty)
- `    MethodDecl [<modifiers...>] <return_type> <name>(<type> <name>, ...)`
- `      Block`
- Control statements have labeled child lines at `depth + 1` with their blocks at `depth + 2`:
  - `If`: `Condition:`, `Then:`, `Else:`
  - `While`: `Condition:`, `Body:`
  - `For`: `Init:`, `Condition:`, `Update:`, `Body:` (absent slots print `<Label>: <none>`)
  - `Call`: `Call`, followed by callee, then `Arguments:` with arguments.

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/lexer.py`
- `minijava/ast_printer.py`
- `tests/test_lexer.py`
- `tests/test_ast_printer.py`
- `tests/golden/parse/*.mj` and `tests/golden/parse/*.out`

### Files You Must Treat as Read-Only:
- `minijava/tokens.py` (M1)
- `minijava/diagnostics.py` (M1)
- `minijava/ast_nodes.py` (M1)
- `minijava/symbol_table.py` (M3)
- `minijava/mj_types.py` (M3)
- `minijava/parser.py` (M4)
- `minijava/semantic.py` (M5)
- `minijava/driver.py` (M1)

---

## E. Implementation Instructions

1. **Implement `minijava/lexer.py`:**
   - Define `LexResult` and `Lexer`.
   - Initialize internal cursor: `self.pos = 0`, `self.line = 1`, `self.column = 1`.
   - Implement helper methods: `_peek()`, `_advance()`, `_skip_whitespace_and_comments()`.
   - Maintain 1-based line and column counters accurately across all character advances.
   - Implement `_scan_identifier()`, `_scan_number()`, `_scan_string()`, `_scan_operator()`.
   - Ensure the lexer never throws uncaught exceptions on malformed user code.
2. **Implement `minijava/ast_printer.py`:**
   - Write a recursive visitor or dispatcher function `_format_node(node, depth, lines)`.
   - Strictly implement all node formatting patterns from §6.9 of the contract.
   - Join lines with `\n` and ensure no trailing newline in the returned string.
3. **Implement Unit Tests:**
   - In `tests/test_lexer.py`, write tests verifying:
     - Keywords and case-sensitivity (`class` vs `Class`).
     - Identifiers with underscores and digits.
     - Integer and floating-point `NUMBER` tokens.
     - Single and double-quoted `STRING` tokens with escape characters.
     - Booleans `true` and `false`.
     - Two-char vs one-char operators (e.g. `>=` vs `>`, `==` vs `=`).
     - Delimiters `(`, `)`, `{`, `}`, `[`, `]`, `;`, `,`, `.`.
     - Single-line and multi-line comments.
     - Invalid character error (e.g. `@`, `#`).
     - Invalid identifier error (e.g. `1name`).
     - Unclosed string literal error.
     - Unclosed comment error.
     - Exact reproduction of PDF p. 4-5 sample code output.
   - In `tests/test_ast_printer.py`, test formatting of expressions, statements, methods, and classes.
4. **Author Parse Golden Cases (`tests/golden/parse/`):**
   - Create at least 10 valid and invalid Mini Java test files with corresponding expected AST printout `.out` files.

---

## F. Local Testing Requirements

Execute these commands locally:
```bash
# 1. Byte-compile your modules
python3 -m compileall -q minijava/lexer.py minijava/ast_printer.py tests/test_lexer.py tests/test_ast_printer.py

# 2. Run unit tests
python3 -m unittest tests.test_lexer -v
python3 -m unittest tests.test_ast_printer -v

# 3. Verify contract compliance
python3 -m unittest tests.test_contract -v
```

---

## G. Integration Requirements

- Your `tokenize()` method returns a `LexResult` containing `tokens`, `diagnostics`, and `identifier_table`.
- All tokens must be instances of `Token` from `minijava.tokens`.
- All diagnostics must be created with `make_diagnostic` from `minijava.diagnostics`.
- All identifier additions must use `IdentifierTable.add_occurrence(name, line, column)` from `minijava.symbol_table`.

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
- `minijava/lexer.py` passes all ≥ 40 unit tests in `test_lexer.py`.
- The PDF p. 4-5 sample program token stream matches the PDF output specification.
- All 4 error diagnostics reproduce the required format and line/column numbers.
- `minijava/ast_printer.py` passes all unit tests in `test_ast_printer.py`.
- At least 10 parser golden test cases are stored in `tests/golden/parse/`.

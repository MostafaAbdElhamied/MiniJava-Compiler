# SHARED PROJECT CONTRACT — Mini Java Compiler: Phase 1 (Lexical Analysis)

**Course:** CSCI415 – Compiler Design (Fall 2026)  
**Contract Version:** 1.0.0-Phase1 (status: FROZEN for Phase 1 Implementation)  
**Authoritative Specification:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)  
**Project Root:** `/home/yousefmahmoudali/Desktop/CompilerProject/`  
**Phase 1 Target Deadline:** **October 30, 2026**  

> **Authority of this Document:** This document is the single source of truth for all five team members and their AI coding assistants. Where this contract and the PDF disagree, the PDF wins. Where the PDF is silent, this contract's decisions are binding. No AI assistant may alter or invent alternative interfaces.

---

## 1. Phase 1 Purpose, Mandatory Requirements & Forward Compatibility

### 1.1 Purpose
Build a robust, hand-crafted Lexical Analyzer (Scanner) for **Mini Java** (*PDF p. 1–2*).

### 1.2 Mandatory Phase 1 Requirements (Verified from PDF)
| ID | Requirement | PDF Page |
|---|---|---|
| R-01 | Lexical analysis on Mini Java source programs | p. 1 |
| R-02 | Phase 1 must build foundation for Phases 2 & 3 | p. 1 |
| R-03 | Python ≥ 3.10 standard library only (no third-party packages) | p. 1 |
| R-04 | All 5 members must participate in code implementation | p. 1 |
| R-05 | **No ready-made lexical analysis libraries or shortcut helpers (no `re`, `ply`, `lark`, `antlr4`, `tokenize`)** | p. 1 |
| R-06 | Plagiarism leads to a zero grade | p. 2 |
| R-07 | Scan source character by character; track 1-based line & column | p. 2 |
| R-08 | Skip whitespace and comments (`//`, `/* ... */`) | p. 2, 3 |
| R-09 | Detect illegal characters and report exact positions | p. 2, 3 |
| R-10 | Case-sensitive (`class` ≠ `Class`) | p. 2 |
| R-11 | Generate token stream ending with exactly one `EOF` token | p. 2 |
| R-12 | Construct and maintain an `IdentifierTable` tracking occurrences | p. 2 |
| R-13 | 15 Keywords; `String` -> `STRING_TYPE`, `main` -> `MAIN`, `boolean` -> `BOOLEAN_TYPE` | p. 2, 4 |
| R-14 | Identifiers: letter/`_` start; digit-prefix -> `LEX_INVALID_IDENTIFIER` | p. 2, 3 |
| R-15 | Literals: `NUMBER` (int/float), `STRING` (quotes preserved), `BOOLEAN` (`true`/`false`) | p. 2 |
| R-16 | Operators: `MATHOP` (`+ - * / %`), `LOGOP` (`== != < > <= >= && \|\| !`), `ASSIGN` (`=`) | p. 3, 4, 5 |
| R-17 | Delimiters: `LPAR (`, `RPAR )`, `LCUR {`, `RCUR }`, `LSQU [`, `RSQU ]`, `SEMICOLON ;`, `COMMA ,`, `DOT .` | p. 3 |
| R-18 | Detect 4 lexical errors: invalid chars, invalid ids, unclosed strings, unclosed comments | p. 3, 4 |
| R-19 | Reproduce PDF sample output format: `Lexeme: <lex>, Token: <TOK>` | p. 4, 5 |
| R-20 | Submit: Source code, `samples/phase1_tests.txt`, `docs/phase1_report.md` | p. 5 |

### 1.3 Forward Compatibility Guarantee
This contract guarantees that Phase 1 outputs plug directly into Phase 2 (Syntax Analysis) and Phase 3 (Semantic Analysis) without refactoring:
- The token stream `list[Token]` terminates with `TokenType.EOF`, which the future recursive-descent parser requires.
- The `Diagnostic` engine supports `Phase.SYNTAX` and `Phase.SEMANTIC`.
- The `minijava/symbol_table.py` module houses `IdentifierTable` for Phase 1 and can cleanly receive `ScopedSymbolTable` for Phase 3.

---

## 2. Global Implementation Conventions

| Topic | Rule |
|---|---|
| **Language & Runtime** | Python ≥ 3.10 standard library only. |
| **Banned Modules** | `re`, `tokenize`, `ast`, `shlex`, `ply`, `lark`, `antlr4`, `sly`, `pyparsing`. `str.split()` may not be used to tokenize source code. |
| **Character Comparisons** | Use explicit ASCII checks (`"a" <= c <= "z"`, `c in "0123456789"`), not `str.isalpha()`/`str.isdigit()`. |
| **Position Indexing** | All `line` and `column` numbers are **1-based**. A tab (`\t`) counts as **1 column**. `\n` ends the line. |
| **Encoding** | Read files with `encoding="utf-8-sig"` (strips BOM) in text mode with universal newlines. |
| **Code Style** | PEP 8, 4 spaces, 100 char max line length, full type hints, descriptive docstrings on all public methods. |
| **Exit Codes** | `0` = successful run with 0 errors; `1` = lexical errors reported; `2` = CLI usage or unreadable file error. |
| **No Stubs** | No `TODO`, `pass`, or `NotImplementedError` stubs in merged code. |

---

## 3. Directory Layout and File Ownership

```
CompilerProject/
├── README.md                             [M1]
├── .gitignore                            [M1]
├── minijava/
│   ├── __init__.py                       [M1]
│   ├── __main__.py                       [M1]
│   ├── tokens.py                         [M1]
│   ├── diagnostics.py                    [M1]
│   ├── driver.py                         [M1]
│   ├── lexer.py                          [M2]
│   ├── symbol_table.py                   [M3]
│   ├── scanner_ident_num.py              [M4]
│   └── scanner_str_op.py                 [M5]
├── tests/
│   ├── __init__.py                       [M1]
│   ├── helpers.py                        [M1]
│   ├── test_contract.py                  [M1]
│   ├── test_driver.py                    [M1]
│   ├── test_lexer_core.py                [M2]
│   ├── test_symbol_table.py              [M3]
│   ├── test_scanner_ident_num.py         [M4]
│   ├── test_pdf_examples.py              [M4]
│   ├── test_scanner_str_op.py            [M5]
│   └── golden/
│       └── lex/                          [M5]
├── samples/
│   └── phase1_tests.txt                  [M3]
└── docs/
    └── phase1_report.md                  [M3]
```

---

## 4. Exact Shared Phase 1 Interfaces and Signatures

### 4.1 Token Module (`minijava/tokens.py`, Owner: Member 1)
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

### 4.2 Diagnostics Module (`minijava/diagnostics.py`, Owner: Member 1)
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

### 4.3 Symbol Table Module (`minijava/symbol_table.py`, Owner: Member 3)
```python
from __future__ import annotations
from dataclasses import dataclass, field

@dataclass
class IdentifierEntry:
    index: int                          # 1-based sequential index by first appearance
    name: str
    first_line: int
    first_column: int
    occurrences: list[tuple[int, int]] = field(default_factory=list) # (line, col) tuples

class IdentifierTable:
    def __init__(self) -> None:
        self._entries: dict[str, IdentifierEntry] = {}
        self._order: list[IdentifierEntry] = []

    def add_occurrence(self, name: str, line: int, column: int) -> IdentifierEntry:
        if name not in self._entries:
            entry = IdentifierEntry(
                index=len(self._order) + 1,
                name=name,
                first_line=line,
                first_column=column,
                occurrences=[(line, column)]
            )
            self._entries[name] = entry
            self._order.append(entry)
            return entry
        entry = self._entries[name]
        entry.occurrences.append((line, column))
        return entry

    def lookup(self, name: str) -> IdentifierEntry | None:
        return self._entries.get(name)

    def entries(self) -> list[IdentifierEntry]:
        return list(self._order)

    def __len__(self) -> int:
        return len(self._order)

    def __contains__(self, name: object) -> bool:
        return name in self._entries

def format_identifier_table(table: IdentifierTable) -> str:
    if len(table) == 0:
        return "(empty)"
    lines = []
    for entry in table.entries():
        lines.append(
            f"Index: {entry.index}, Name: {entry.name}, "
            f"First seen: line {entry.first_line}, column {entry.first_column}, "
            f"Occurrences: {len(entry.occurrences)}"
        )
    return "\n".join(lines)
```

### 4.4 Helper Scanners
- **`scanner_ident_num.py` (Owner: Member 4):**
  ```python
  def scan_identifier_or_number(source: str, pos: int, line: int, col: int, id_table: IdentifierTable) -> tuple[int, int, int, Token | None, Diagnostic | None]:
      """Scans an identifier, keyword, valid number, or invalid identifier.
      Returns (new_pos, new_line, new_col, token_or_none, diagnostic_or_none).
      """
  ```
- **`scanner_str_op.py` (Owner: Member 5):**
  ```python
  def scan_string(source: str, pos: int, line: int, col: int) -> tuple[int, int, int, Token | None, Diagnostic | None]:
      """Scans a single or double-quoted string literal or reports LEX_UNCLOSED_STRING."""
      
  def scan_operator(source: str, pos: int, line: int, col: int) -> tuple[int, int, int, Token | None]:
      """Scans operators using maximal munch (two-char first)."""
  ```

### 4.5 Central Lexer Module (`minijava/lexer.py`, Owner: Member 2)
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

### 4.6 CLI Driver & Output Format (`minijava/driver.py`, Owner: Member 1)
CLI invocation:
```bash
python3 -m minijava <source-file> [--phase lex] [--tokens] [--symbols]
```
Report layout: sections joined by one blank line (`\n\n`):
1. `=== Tokens ===` (always printed in `--phase lex` or `--tokens`)
2. `=== Symbol Table ===` (always printed in `--phase lex` or `--symbols`)
3. `=== Errors ===` (always printed; prints `(none)` if 0 errors)
4. `=== Result ===` (always printed: `Lexical analysis completed successfully with no errors.` or `Lexical analysis failed with N error(s).`)

---

## 5. Definition of Done for Phase 1

A Phase 1 component is officially done only when:
1. It contains no stubs (`pass`, `TODO`).
2. Its signatures match this contract 100% and pass `tests/test_contract.py`.
3. Its unit tests pass offline with zero failures in `< 5` seconds.
4. It touches only the files owned by the member (§3).
5. The member can explain their code in the oral grading interview (*PDF p. 1*).

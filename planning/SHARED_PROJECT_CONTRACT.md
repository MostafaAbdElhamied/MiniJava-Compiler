# SHARED PROJECT CONTRACT — Mini Java Compiler (CSCI415, Fall 2026)

**Contract version:** 1.0.0 (status: PROPOSED → becomes FROZEN once all five members approve, see §15)
**Official specification:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
**Project root:** `/home/yousefmahmoudali/Desktop/CompilerProject/`

> This document is the single source of truth for every member and every AI assistant.
> Where this contract and the PDF disagree, the PDF wins and a change request (§15) must be raised.
> Where the PDF is silent, this contract's decisions are binding (they are marked **[PROVISIONAL]** and
> listed in §1.3 so the team can ask the instructor).
> No AI assistant may reinterpret, extend or "improve" anything in this document on its own.

---

## 1. Project purpose and mandatory requirements

### 1.1 Purpose
Build a compiler front end for **Mini Java**, a simplified Java-like language. The final submission
**must perform lexical, syntax and semantic analysis** on Mini Java source programs (PDF p1). The project
is graded in three phases (PDF p1):

| Phase | Component | PDF dates (p1) |
|---|---|---|
| Phase 01 | Lexical Analyzer | starts Oct 12, deadline **Oct 30** |
| Phase 02 | Syntax Analyzer | starts Nov 21, deadline **Dec 20** |
| Phase 03 | Semantic Analyzer | starts Nov 21, deadline **Dec 20** |

No code generation, IR, optimization or execution is required by the PDF and **none will be built**.

### 1.2 Mandatory requirements (verified from the PDF)
| ID | Requirement | PDF page |
|---|---|---|
| R-01 | Lexical, syntax and semantic analysis in the final submission | p1 |
| R-02 | Phases must build on each other | p1 |
| R-03 | Any programming language may be used | p1 |
| R-04 | All members must participate in implementation; discussion with all members | p1 |
| R-05 | **No ready-made lexer/parser libraries or ready functions that directly solve compiler tasks** | p1 |
| R-06 | Plagiarism → zero | p2 |
| R-07 | Lexer reads source character by character, groups into tokens, classifies them, builds a symbol table | p2 |
| R-08 | Ignore whitespace and comments (`//…`, `/* … */`) | p2, p3 |
| R-09 | Detect illegal characters and report their positions | p2 |
| R-10 | Case sensitive (`class` ≠ `Class`) | p2 |
| R-11 | Produce a token stream consumed by the parser | p2 |
| R-12 | Symbol table for identifiers | p2 |
| R-13 | Keywords: class, public, static, void, int, float, boolean, char, String, if, else, while, for, return; `String` → `STRING_TYPE` | p2 |
| R-14 | Identifiers: start with letter or `_`, then letters/digits/`_`; digit followed by letters/`_` → invalid identifier | p2 |
| R-15 | Literals: `NUMBER` (int or float), `STRING` (single or double quoted), `BOOLEAN` (`true`/`false`) | p2 |
| R-16 | Arithmetic and comparison operators (no list given) | p3 |
| R-17 | Delimiters `LPAR ( RPAR ) LCUR { RCUR } LSQU [ RSQU ] SEMICOLON ; COMMA , DOT .` | p3 |
| R-18 | Lexical errors: invalid symbols, invalid identifiers, unclosed strings, unterminated multi-line comments; each with line, column, descriptive message | p3 |
| R-19 | Error message examples (exact wording reproduced in §10) | p3–p4 |
| R-20 | Sample output format `Lexeme: <lexeme>, Token: <TOKEN>` ("similar, not exact") | p4–p5 |
| R-21 | Phase 1 deliverables: source code, input text file of Mini Java test programs, 1–2 page report (tokens, error handling, challenges) | p5 |

### 1.3 Provisional decisions where the PDF is silent (binding for implementation; to be confirmed with instructor)
| ID | Decision | Reason |
|---|---|---|
| P-01 | Operators: `MATHOP` = `+ - * / %`; `LOGOP` = `== != < > <= >= && \|\| !`; `ASSIGN` = `=` | p3 lists no operators; p4–p5 sample shows `+`→MATHOP, `>=`→LOGOP, `=`→ASSIGN |
| P-02 | `main` is a reserved word with token `MAIN` | p4 sample shows `Lexeme: main, Token: MAIN` although p2 keyword list omits it |
| P-03 | Keyword `boolean` → token `BOOLEAN_TYPE` | `BOOLEAN` is already the literal token (p2); mirrors `String`→`STRING_TYPE` |
| P-04 | Other keywords use the upper-case keyword as token name (`CLASS`, `INT`, `CHAR`, …) | consistent with p4 sample |
| P-05 | `++`, `--`, `+=`, etc. are **not** supported (`i++` lexes as `i`, `+`, `+`) | not in PDF |
| P-06 | Phase 2 grammar is defined in §6.3 of this contract | PDF gives no grammar |
| P-07 | Phase 3 semantic rules are defined in §6.6 of this contract | PDF gives no rules |
| P-08 | A phase with errors stops the pipeline; later phases do not run | safest, deterministic |
| P-09 | The parser stops at the **first** syntax error (no recovery) | PDF does not require recovery |
| P-10 | `System.out.println(x)` / `System.out.print(x)` are accepted as built-ins by the semantic analyzer | typical Java test programs use them |
| P-11 | Phase 2/3 deliverables assumed to mirror Phase 1 (source, input test file, short report) | PDF p5 lists only Phase 1 deliverables |

---

## 2. Selected architecture

Implementation language: **Python 3** (minimum 3.10, verified locally with Python 3.14.4), **standard library only**.

Pipeline (one direction, no cycles):

```
source file ──► driver.read_source() ──► str
          str ──► Lexer.tokenize()  ──► LexResult(tokens[...EOF], diagnostics, identifier_table)
  tokens list ──► Parser.parse()    ──► ParseResult(program: Program | None, diagnostics)
      Program ──► SemanticAnalyzer.analyze() ──► SemanticResult(diagnostics, symbol_table)
  all results ──► driver formats sections ──► stdout, exit code
```

| Component | Module | Owner | Input | Output |
|---|---|---|---|---|
| Shared tokens | `minijava/tokens.py` | M1 | — | `TokenType`, `Token`, keyword/operator tables |
| Shared diagnostics | `minijava/diagnostics.py` | M1 | — | `Phase`, `ErrorCode`, `Diagnostic`, `make_diagnostic` |
| Shared AST | `minijava/ast_nodes.py` | M1 (M4 reviews) | — | AST dataclasses |
| Lexer (Phase 1) | `minijava/lexer.py` | M2 | `str` | `LexResult` |
| Symbol tables | `minijava/symbol_table.py` | M3 | calls from lexer/semantic | `IdentifierTable`, `ScopedSymbolTable` |
| Type system | `minijava/mj_types.py` | M3 | types/operators | `MJType`, type rules |
| Parser (Phase 2) | `minijava/parser.py` | M4 | `list[Token]` | `ParseResult` |
| AST printer | `minijava/ast_printer.py` | M2 | `Program` | `str` |
| Semantic analyzer (Phase 3) | `minijava/semantic.py` | M5 | `Program` | `SemanticResult` |
| Driver / CLI | `minijava/driver.py`, `minijava/__main__.py` | M1 | argv | stdout + exit code |

Errors are **never** raised as exceptions across component boundaries for user-program problems: every
stage returns a list of `Diagnostic` objects. Exceptions are reserved for programming errors (contract
violations, e.g. a token list without `EOF`) and I/O errors in the driver.

---

## 3. Approved directory structure

```
CompilerProject/
├── Projects/
│   └── ProjectGuidelinesAndDetails.pdf  (read-only, official spec)
├── README.md                            M1  build/run/test instructions
├── .gitignore                           M1
├── planning/                            M1  (planning docs; contract changes via §15 only)
│   ├── SHARED_PROJECT_CONTRACT.md
│   ├── CONTRACT_CHANGES.md              M1  change-request log (created at Milestone 2)
│   └── ...
├── minijava/
│   ├── __init__.py                      M1  contains only: __version__ = "1.0.0"
│   ├── __main__.py                      M1  `python3 -m minijava` entry → driver.main()
│   ├── tokens.py                        M1  verbatim from Appendix A
│   ├── diagnostics.py                   M1  verbatim from Appendix B
│   ├── ast_nodes.py                     M1  verbatim from Appendix C (M4 reviewer)
│   ├── driver.py                        M1
│   ├── lexer.py                         M2
│   ├── ast_printer.py                   M2
│   ├── symbol_table.py                  M3
│   ├── mj_types.py                      M3  (NOT named types.py: avoids stdlib clash)
│   ├── parser.py                        M4
│   └── semantic.py                      M5
├── tests/
│   ├── __init__.py                      M1  (empty)
│   ├── helpers.py                       M1  shared test utilities (§12.3)
│   ├── test_contract.py                 M1  signature/contract conformance tests
│   ├── test_driver.py                   M1
│   ├── test_golden.py                   M1  runs every golden case
│   ├── test_lexer.py                    M2
│   ├── test_ast_printer.py              M2
│   ├── test_symbol_table.py             M3
│   ├── test_mj_types.py                 M3
│   ├── test_pdf_examples.py             M4  PDF p3–p5 examples as acceptance tests
│   ├── test_parser.py                   M4
│   ├── test_semantic.py                 M5
│   └── golden/
│       ├── lex/        *.mj + *.out     M5  (independent oracle for the lexer)
│       ├── parse/      *.mj + *.out     M2  (independent oracle for the parser)
│       └── semantic/   *.mj + *.out     M5
├── samples/
│   ├── phase1_tests.txt                 M3  Phase 1 "input text file" deliverable
│   └── phase23_tests.txt                M3  Phase 2/3 input test file
└── docs/
    ├── phase1_report.md                 M3  editor (content from M2, M5, M1)
    └── phase23_report.md                M3  editor (content from M4, M5, M2)
```

Rules: every file has exactly one owner (column on the right). Nobody else edits it (§14).
No other top-level directories or files may be created without a change request.

---

## 4. Global conventions

| Topic | Rule |
|---|---|
| Language | Python ≥ 3.10 (uses `dataclass(kw_only=True)`, `X \| Y` in annotations with `from __future__ import annotations`) |
| Dependencies | **Standard library only.** No `pip install`. |
| Prohibited modules | `re`, `tokenize`, `ast` (stdlib), `shlex`, `ply`, `lark`, `antlr4`, `sly`, `pyparsing`, any parser/lexer generator, any third-party package. `str.split`/`str.find` may not be used to tokenize source code. (R-05, PDF p1) |
| Character classes | Use explicit ASCII comparisons (`"a" <= c <= "z"`), **not** `str.isalpha()/isdigit()/isalnum()` (they accept Unicode). |
| Imports | Absolute only: `from minijava.tokens import Token, TokenType`. Allowed import graph in §8. |
| Every module | starts with a module docstring, then `from __future__ import annotations`. |
| Naming | modules `snake_case.py`; classes `PascalCase`; functions/methods/variables `snake_case`; constants `UPPER_SNAKE_CASE`; private helpers prefixed `_`. |
| Formatting | PEP 8, 4-space indentation, no tabs, max line length 100, UTF-8 files, `\n` newlines, file ends with a newline. |
| Typing | All public functions/methods have full type hints. |
| Docstrings | Every public class/function has a docstring stating purpose, parameters, return value. |
| Positions | `line` and `column` are **1-based**. A tab counts as **one** column. `\n` ends a line. `\r` is ordinary whitespace (CRLF files are normalised by the driver anyway). |
| Encoding | Driver reads source with `encoding="utf-8-sig"` (strips BOM) in text mode (universal newlines). Non-ASCII characters outside string literals and comments are invalid characters. |
| Output | All compiler output (including diagnostics) to **stdout**. Only CLI/IO failures go to **stderr**. No logging module, no debug prints in committed code. |
| Exit codes | `0` = no diagnostics; `1` = at least one diagnostic; `2` = usage error, unreadable file, or phase not available. |
| Mutation | Tokens are immutable (frozen). The AST is **never mutated after `Parser.parse()` returns** (the semantic analyzer keeps its own data). |
| Placeholders | No `pass`/`TODO`/`NotImplementedError` stubs in merged code for contracted behavior. |

---

## 5. Build, run and test commands (identical for everyone)

Run from the project root `/home/yousefmahmoudali/Desktop/CompilerProject`:

| Purpose | Command |
|---|---|
| Python version check | `python3 --version` (must be ≥ 3.10) |
| "Build" (byte-compile, catches syntax errors) | `python3 -m compileall -q minijava tests` |
| Full test suite | `python3 -m unittest discover -s tests -t . -v` |
| One test module | `python3 -m unittest tests.test_lexer -v` |
| Phase 1 run | `python3 -m minijava samples/phase1_tests.txt --phase lex` |
| Phase 2 run | `python3 -m minijava <file> --phase parse` |
| Full run (Phase 3) | `python3 -m minijava <file>` (default `--phase semantic`) |

---

## 6. Exact shared interfaces

### 6.1 Tokens (`minijava/tokens.py`, owner M1, content = Appendix A verbatim)

Token type names and lexeme rules:

| TokenType | Lexemes |
|---|---|
| `CLASS PUBLIC STATIC VOID INT FLOAT CHAR IF ELSE WHILE FOR RETURN` | the lower-case keyword (`class`, `public`, …) |
| `BOOLEAN_TYPE` | `boolean` (P-03) |
| `STRING_TYPE` | `String` |
| `MAIN` | `main` (P-02) |
| `IDENTIFIER` | `[A-Za-z_][A-Za-z0-9_]*` that is not a reserved word |
| `NUMBER` | `[0-9]+` or `[0-9]+\.[0-9]+` |
| `STRING` | `"…"` or `'…'`, lexeme **includes the quotes** exactly as written |
| `BOOLEAN` | `true`, `false` |
| `MATHOP` | `+ - * / %` |
| `LOGOP` | `== != < > <= >= && \|\| !` |
| `ASSIGN` | `=` |
| `LPAR RPAR LCUR RCUR LSQU RSQU SEMICOLON COMMA DOT` | `( ) { } [ ] ; , .` |
| `EOF` | `""` — exactly one, always last |

### 6.2 Lexer (`minijava/lexer.py`, owner M2)

```python
@dataclass
class LexResult:
    tokens: list[Token]            # valid tokens in source order, ALWAYS ends with exactly one EOF token
    diagnostics: list[Diagnostic]  # lexical errors in source order (phase LEXICAL)
    identifier_table: IdentifierTable

class Lexer:
    def __init__(self, source: str) -> None: ...
    def tokenize(self) -> LexResult: ...   # may be called once per instance

def tokenize(source: str) -> LexResult:   # convenience: return Lexer(source).tokenize()
```

Lexer rules (all binding):
1. Scan character by character with an index; no regex (R-05, R-07).
2. Whitespace = space, `\t`, `\n`, `\r`, `\f`, `\v` — skipped.
3. `//` skips to (not including) the next `\n` or EOF. `/*` skips to the first `*/` (no nesting). If EOF is reached first: `LEX_UNCLOSED_COMMENT` at the position of `/`; scanning ends.
4. Identifier/keyword: start `[A-Za-z_]`, continue `[A-Za-z0-9_]`; look up `KEYWORDS` (case-sensitive) else `IDENTIFIER`. Every `IDENTIFIER` token is recorded with `identifier_table.add_occurrence(name, line, column)`.
5. Number: `[0-9]+`; if next is `.` **and** the char after it is a digit, consume `.` and `[0-9]+`. Then, if the next char is `[A-Za-z_]`, consume all following `[A-Za-z0-9_]` and report `LEX_INVALID_IDENTIFIER` with the whole lexeme (e.g. `1name`, `3abc`, `2.5x`) at the start position; no token is produced. `25.` → `NUMBER 25`, `DOT`. `.5` → `DOT`, `NUMBER 5`. A leading `-` is always a separate `MATHOP`.
6. String: opening quote `"` or `'`; ends at the next unescaped **same** quote. `\` escapes the next character (both kept in the lexeme). If `\n` or EOF occurs before the closing quote: `LEX_UNCLOSED_STRING` at the opening quote position, no token, scanning resumes **at** that `\n` (or ends at EOF).
7. Operators, maximal munch: try two-char `== != <= >= && ||` first, then one-char `= < > ! + - * / %`. A lone `&` or `|` is an invalid character.
8. Delimiters per `DELIMITERS`.
9. Anything else (including non-ASCII, `@`, `#`, `$`, `` ` ``, `~`, `^`, `?`, `:`, `\`, lone `&`, lone `|`): `LEX_INVALID_CHARACTER` with that single character, skip it, continue.
10. Lexing never stops at the first error (except an unclosed comment, which consumes to EOF). Erroneous lexemes produce **no token**.
11. `EOF` token position: the line/column just after the last character (for empty input: line 1, column 1).

### 6.3 Grammar (binding, P-06) — terminals are `TokenType` names; `LOGOP("&&")` means LOGOP with that lexeme

```
program        ::= classDecl { classDecl } EOF
classDecl      ::= CLASS IDENTIFIER LCUR { member } RCUR
member         ::= modifiers ( mainMethod
                             | VOID IDENTIFIER methodRest
                             | type IDENTIFIER ( methodRest | fieldRest ) )
modifiers      ::= [ PUBLIC ] [ STATIC ]
mainMethod     ::= VOID MAIN LPAR [ STRING_TYPE LSQU RSQU IDENTIFIER ] RPAR block
methodRest     ::= LPAR [ param { COMMA param } ] RPAR block
fieldRest      ::= [ ASSIGN initializer ] SEMICOLON
param          ::= type IDENTIFIER
type           ::= ( INT | FLOAT | BOOLEAN_TYPE | CHAR | STRING_TYPE ) [ LSQU RSQU ]
initializer    ::= expression | arrayInit
arrayInit      ::= LCUR [ expression { COMMA expression } ] RCUR
block          ::= LCUR { statement } RCUR
statement      ::= block
                 | varDecl SEMICOLON
                 | IF LPAR expression RPAR statement [ ELSE statement ]      (else binds to nearest if)
                 | WHILE LPAR expression RPAR statement
                 | FOR LPAR [ varDecl | simpleStmt ] SEMICOLON [ expression ] SEMICOLON
                       [ simpleStmt ] RPAR statement
                 | RETURN [ expression ] SEMICOLON
                 | simpleStmt SEMICOLON
varDecl        ::= type IDENTIFIER [ ASSIGN initializer ]
simpleStmt     ::= postfixExpr [ ASSIGN expression ]
expression     ::= orExpr
orExpr         ::= andExpr { LOGOP("||") andExpr }
andExpr        ::= eqExpr  { LOGOP("&&") eqExpr }
eqExpr         ::= relExpr { (LOGOP("==") | LOGOP("!=")) relExpr }
relExpr        ::= addExpr { (LOGOP("<") | LOGOP(">") | LOGOP("<=") | LOGOP(">=")) addExpr }
addExpr        ::= mulExpr { (MATHOP("+") | MATHOP("-")) mulExpr }
mulExpr        ::= unaryExpr { (MATHOP("*") | MATHOP("/") | MATHOP("%")) unaryExpr }
unaryExpr      ::= LOGOP("!") unaryExpr | MATHOP("-") unaryExpr | postfixExpr
postfixExpr    ::= primary { LSQU expression RSQU | DOT IDENTIFIER | LPAR [ expression { COMMA expression } ] RPAR }
primary        ::= NUMBER | STRING | BOOLEAN | IDENTIFIER | LPAR expression RPAR
```

Grammar notes (binding):
- All binary operators are **left-associative**.
- `LPAR` in `postfixExpr` is a call **only if** the expression built so far is an `Identifier` or `FieldAccess`; otherwise the postfix loop ends.
- `simpleStmt`: if followed by `ASSIGN`, the left side must be `Identifier`, `IndexExpr` or `FieldAccess`, else `SYN_INVALID_ASSIGNMENT_TARGET` (reported at the `=` token) → produces `Assign`. If not followed by `ASSIGN`, it must be a `CallExpr`, else `SYN_NOT_A_STATEMENT` (reported at the expression's position) → produces `ExprStmt`.
- A statement starting with a type keyword is a `varDecl`; starting with `LCUR` is a block.
- Not supported (syntax errors): class-typed variables, `new`, `++/--`, compound assignment, empty statement `;`, `static public` order, multi-dimensional arrays, `else if` is fine (it is `else` + `if` statement).

### 6.4 AST (`minijava/ast_nodes.py`, owner M1, content = Appendix C verbatim)

Position rule for every node (`line`, `column`):

| Node | Position of |
|---|---|
| `Program` | always line 1, column 1 |
| `ClassDecl`, `FieldDecl`, `MethodDecl`, `Param`, `VarDecl` | the **declared name** token (IDENTIFIER, or MAIN) |
| `TypeNode` | the type keyword token |
| `Block`, `ArrayInit` | the `{` token |
| `If`, `While`, `For`, `Return` | the keyword token |
| `Assign`, `Binary` | the **operator** token (`=`, `+`, …) |
| `Unary` | the operator token |
| `Literal`, `Identifier` | the token itself |
| `FieldAccess` | the member-name IDENTIFIER after the `.` |
| `CallExpr`, `IndexExpr`, `ExprStmt` | same position as their `callee` / `array` / `expression` child |

### 6.5 Parser (`minijava/parser.py`, owner M4)

```python
@dataclass
class ParseResult:
    program: Program | None        # None if and only if diagnostics is non-empty
    diagnostics: list[Diagnostic]  # 0 or 1 entries (P-09), phase SYNTAX

class Parser:
    def __init__(self, tokens: list[Token]) -> None: ...
        # raises ValueError if tokens is empty or tokens[-1].type is not TokenType.EOF
    def parse(self) -> ParseResult: ...

def parse(tokens: list[Token]) -> ParseResult:   # convenience: return Parser(tokens).parse()
```

Hand-written recursive descent (R-05). Syntax error arguments for `SYN_UNEXPECTED_TOKEN`:
- `found` = `token.describe()` (`'}'` or `end of input`).
- `expected` = one of: a quoted canonical lexeme (`';'`, `'('`, `')'`, `'{'`, `'}'`, `']'`, `'='`, `'class'`) **or** exactly one of the phrases `an identifier`, `a type`, `an expression`, `a statement`, `a class member`, `a method body`.

### 6.6 Semantic rules (binding, P-07) and analyzer (`minijava/semantic.py`, owner M5)

```python
@dataclass
class SemanticResult:
    diagnostics: list[Diagnostic]      # phase SEMANTIC, in deterministic order (below)
    symbol_table: ScopedSymbolTable

class SemanticAnalyzer:
    def __init__(self) -> None: ...
    def analyze(self, program: Program) -> SemanticResult: ...   # never mutates program

def analyze(program: Program) -> SemanticResult:   # convenience
```

**Passes (deterministic order):**
1. Pass 1a — for each `ClassDecl` in order: define a `CLASS` symbol in the global scope (duplicate → `SEM_DUPLICATE_DECLARATION`). Create its class scope (`enter_scope(name, ScopeKind.CLASS)`), store it in `symbol.scope`, exit.
2. Pass 1b — for each class in order, re-enter its class scope; for each member in order define `FIELD` or `METHOD` symbol (duplicates in one class scope → `SEM_DUPLICATE_DECLARATION`; fields and methods share one namespace). Method symbols get `param_types`.
3. Pass 2 — for each class in order, re-enter its scope; for each member in order: check field initializer; or for a method: `enter_scope(method_name, ScopeKind.METHOD)`, define `PARAMETER`s, analyze body **statements directly in the method scope** (the body `Block` does not open another scope), exit.
- Duplicate classes/methods are still analyzed (in their own new scopes) so errors inside them are reported.

**Scopes:** nested `Block` statement → `enter_scope(f"block@{line}:{column}", ScopeKind.BLOCK)`; `For` → `enter_scope(f"for@{line}:{column}", ScopeKind.BLOCK)` containing its init var, and its body if it is a `Block` statement is analyzed inside a further `block@…` scope like any block.

**Declarations:** `VarDecl`: (1) analyze initializer, (2) if `table.lookup_in_method(name)` finds a symbol → `SEM_DUPLICATE_DECLARATION` (no redefinition) else define `LOCAL`, (3) check initializer type. Parameters: duplicate param names → `SEM_DUPLICATE_DECLARATION`. Locals may shadow fields.

**Type rules** (implemented in `mj_types.py` by M3, applied by M5):
- `NUMBER` with `.` → `float`, else `int`. `BOOLEAN` → `boolean`. `STRING` single-quoted with exactly one character between the quotes (an escape `\x` counts as one) → `char`; any other `STRING` → `String`.
- Assignment / initializer / argument / return compatibility `is_assignable(target, source)`: identical types, or `float ← int`. `<error>` on either side is always compatible (no cascading errors).
- `+`: int,int→int; numeric with float→float; if either side is `String` and the other is a non-void, non-array, non-class primitive → `String`. `- * / %`: int,int→int; numeric mix→float. `< > <= >=`: numeric,numeric → boolean. `== !=`: both numeric, or identical non-void types → boolean. `&& ||`: boolean,boolean → boolean. Unary `-`: numeric → same; unary `!`: boolean → boolean. `char` is **not** numeric.
- Conditions of `if`/`while`/`for` must be `boolean` (a missing `for` condition is fine).
- `a[i]`: `a` must be an array type (else `SEM_NOT_AN_ARRAY`), `i` must be `int` (else `SEM_INDEX_NOT_INT`); result is the element type.
- `ArrayInit` allowed only for array-typed targets (else `SEM_INVALID_ARRAY_INITIALIZER`); each element must be assignable to the element type (else `SEM_TYPE_MISMATCH` at that element).
- `Return` with value in a `void` method → `SEM_RETURN_VALUE_IN_VOID`; without value in non-void → `SEM_MISSING_RETURN_VALUE`; incompatible → `SEM_RETURN_TYPE_MISMATCH`. No "missing return path" analysis.
- `static`/instance distinction is **not** checked.

**Name resolution in expressions:** `Identifier` → `table.resolve(name)`; none → `SEM_UNDECLARED_IDENTIFIER`; a `METHOD` or `CLASS` symbol used as a value → `SEM_NOT_A_VARIABLE`. Calls: callee `Identifier` must resolve to a `METHOD` (none → `SEM_UNDECLARED_IDENTIFIER`, otherwise non-method → `SEM_NOT_A_METHOD`); arity (`SEM_ARGUMENT_COUNT` at the call) then each argument (`SEM_ARGUMENT_TYPE` at the argument). A `void` call used as a value (anywhere except an `ExprStmt`) → `SEM_VOID_VALUE`.
`FieldAccess`: (a) `System.out.println(...)` (0 or 1 arg) and `System.out.print(...)` (exactly 1 arg), any non-void argument type, return `void`, when `System` is not a declared name (method name in messages: `System.out.println` / `System.out.print`, `expected` arg = `"0 or 1"` / `"1"`); (b) `x.length` where `x` is an array → `int` (not assignable → `SEM_NOT_A_VARIABLE` with name `length`); (c) `C.m` where `C` resolves to a `CLASS` symbol → look up `m` in `C`'s scope with `resolve_local` (none → `SEM_UNKNOWN_MEMBER`); (d) anything else → `SEM_INVALID_MEMBER_ACCESS`.
Assignment target must be a `LOCAL`/`PARAMETER`/`FIELD`, an `IndexExpr`, or a `FieldAccess` to a `FIELD`; otherwise `SEM_NOT_A_VARIABLE`.

**Evaluation order inside statements:** children before parents, left to right. `Assign`: target, then value, then compatibility (`SEM_TYPE_MISMATCH` at the `Assign` node). `VarDecl`/`FieldDecl` mismatch → at the decl node. Binary operand errors at the `Binary` node. Condition error at the condition expression node. Once a subexpression has type `<error>`, no further diagnostics are reported for expressions that depend on it.

**Error positions (summary):** undeclared/not-variable/not-method → the `Identifier` (or `FieldAccess`) node; duplicates → the new declaration node; `SEM_ARGUMENT_COUNT`, `SEM_VOID_VALUE` → the `CallExpr`; `SEM_NOT_AN_ARRAY` → the `IndexExpr`; `SEM_INDEX_NOT_INT` → the index expression; return errors → the `Return` node; `SEM_INVALID_ARRAY_INITIALIZER` → the declaration node.

### 6.7 Type system (`minijava/mj_types.py`, owner M3)

```python
@dataclass(frozen=True)
class MJType:
    name: str            # "int","float","boolean","char","String","void","<error>", or a class name
    is_array: bool = False
    def __str__(self) -> str: ...          # name + "[]" if is_array, e.g. "int[]"

INT: MJType; FLOAT: MJType; BOOLEAN: MJType; CHAR: MJType; STRING: MJType; VOID: MJType; ERROR: MJType
PRIMITIVE_NAMES: frozenset[str]      # {"int","float","boolean","char","String"}

def class_type(name: str) -> MJType
def is_class_type(t: MJType) -> bool         # non-array, name not primitive/"void"/"<error>"
def array_of(t: MJType) -> MJType            # ValueError if t is array/void/error/class
def element_type(t: MJType) -> MJType        # ValueError if not array
def from_type_node(node: TypeNode) -> MJType
def literal_type(kind: str, lexeme: str) -> MJType     # kind in {"NUMBER","STRING","BOOLEAN"}
def is_numeric(t: MJType) -> bool            # int or float, non-array
def is_assignable(target: MJType, source: MJType) -> bool
def binary_result_type(op: str, left: MJType, right: MJType) -> MJType | None
        # None = invalid operands; returns ERROR if either operand is ERROR
def unary_result_type(op: str, operand: MJType) -> MJType | None
def format_method_type(return_type: MJType, param_types: list[MJType]) -> str   # "int(int, float[])"
```

### 6.8 Symbol tables (`minijava/symbol_table.py`, owner M3)

```python
# ---- Phase 1: identifier table (built by the lexer) ----
@dataclass
class IdentifierEntry:
    index: int                      # 1-based, order of first appearance
    name: str
    first_line: int
    first_column: int
    occurrences: list[tuple[int, int]] = field(default_factory=list)   # (line, column), includes first

class IdentifierTable:
    def __init__(self) -> None: ...
    def add_occurrence(self, name: str, line: int, column: int) -> IdentifierEntry: ...
    def lookup(self, name: str) -> IdentifierEntry | None: ...
    def entries(self) -> list[IdentifierEntry]: ...     # copy, in index order
    def __len__(self) -> int: ...
    def __contains__(self, name: object) -> bool: ...

def format_identifier_table(table: IdentifierTable) -> str: ...

# ---- Phase 3: scoped symbol table (built by the semantic analyzer) ----
class SymbolKind(Enum):  CLASS = "CLASS"; FIELD = "FIELD"; METHOD = "METHOD"; PARAMETER = "PARAMETER"; LOCAL = "LOCAL"
class ScopeKind(Enum):   GLOBAL = "GLOBAL"; CLASS = "CLASS"; METHOD = "METHOD"; BLOCK = "BLOCK"

@dataclass
class Symbol:
    name: str
    kind: SymbolKind
    mj_type: MJType                  # variables: their type; METHOD: return type; CLASS: class_type(name)
    line: int                        # position of the declared name
    column: int
    param_types: list[MJType] = field(default_factory=list)   # METHOD only
    scope: Scope | None = None       # CLASS only: the class scope

class Scope:
    def __init__(self, name: str, kind: ScopeKind, parent: Scope | None) -> None: ...
    name: str; kind: ScopeKind; parent: Scope | None
    symbols: dict[str, Symbol]       # insertion ordered
    children: list[Scope]            # creation order
    def define(self, symbol: Symbol) -> bool: ...          # False (and no change) if name already in THIS scope
    def resolve_local(self, name: str) -> Symbol | None: ...
    def resolve(self, name: str) -> Symbol | None: ...     # this scope, then parents

class ScopedSymbolTable:
    def __init__(self) -> None: ...                         # creates global scope named "global", kind GLOBAL
    global_scope: Scope
    current: Scope
    def enter_scope(self, name: str, kind: ScopeKind) -> Scope: ...   # new child of current; becomes current
    def reenter_scope(self, scope: Scope) -> None: ...     # ValueError unless scope.parent is current
    def exit_scope(self) -> Scope: ...                     # returns the exited scope; RuntimeError at global
    def define(self, symbol: Symbol) -> bool: ...          # current.define
    def resolve(self, name: str) -> Symbol | None: ...     # current.resolve
    def resolve_local(self, name: str) -> Symbol | None: ...
    def lookup_in_method(self, name: str) -> Symbol | None: ...
        # search current scope and parents while kind is BLOCK or METHOD (inclusive); stop before CLASS/GLOBAL
    def all_scopes(self) -> list[Scope]: ...               # pre-order from global

def format_scoped_table(table: ScopedSymbolTable) -> str: ...
```

### 6.9 AST printer (`minijava/ast_printer.py`, owner M2)

```python
def format_ast(program: Program) -> str: ...   # lines joined with "\n", no trailing newline
```
Format: one node per line, two spaces of indentation per depth, children at depth+1:

| Node | Line text | Children (in order) |
|---|---|---|
| Program | `Program` | classes |
| ClassDecl | `ClassDecl <name>` | members |
| FieldDecl | `FieldDecl <modifiers…> <type> <name>` | initializer if present |
| MethodDecl | `MethodDecl <modifiers…> <return type> <name>(<type> <name>, …)` | body |
| Block | `Block` | statements |
| VarDecl | `VarDecl <type> <name>` | initializer if present |
| Assign | `Assign` | target, value |
| If | `If` | label `Condition:` → condition; `Then:` → then_branch; `Else:` → else_branch (only if present) |
| While | `While` | `Condition:`, `Body:` |
| For | `For` | `Init:`, `Condition:`, `Update:`, `Body:` — an absent slot prints `<Label>: <none>` with no child |
| Return | `Return` | value if present |
| ExprStmt | `ExprStmt` | expression |
| Binary | `Binary <op>` | left, right |
| Unary | `Unary <op>` | operand |
| Literal | `Literal <kind> <lexeme>` | — |
| Identifier | `Identifier <name>` | — |
| IndexExpr | `Index` | array, index |
| FieldAccess | `FieldAccess <name>` | target |
| CallExpr | `Call` | callee, then label `Arguments:` with args (label printed even if no args) |
| ArrayInit | `ArrayInit` | elements |

A label line is at depth+1 and its child at depth+2. Modifiers are omitted when empty (single spaces only). Types print as `int`, `int[]`, `void`.

### 6.10 Driver (`minijava/driver.py`, owner M1)

```python
@dataclass
class CompilationResult:
    phase: Phase                         # highest phase requested
    lex: LexResult
    parse: ParseResult | None            # None if not run
    semantic: SemanticResult | None      # None if not run
    diagnostics: list[Diagnostic]        # all diagnostics, in pipeline order

def read_source(path: str) -> str: ...   # utf-8-sig, universal newlines; raises OSError/UnicodeDecodeError
def compile_source(source: str, phase: Phase = Phase.SEMANTIC) -> CompilationResult: ...
def format_report(result: CompilationResult, show_tokens: bool = False,
                  show_symbols: bool = False, show_ast: bool = False) -> str: ...
def main(argv: list[str] | None = None) -> int: ...
```
`compile_source` stops after the first phase with diagnostics (P-08). Parser and semantic modules are imported lazily inside `compile_source`; if unavailable (`ImportError`), `main` prints `error: phase '<name>' is not available in this build` to stderr and returns 2.

---

## 7. Input/output conventions

### 7.1 CLI
```
python3 -m minijava <source-file> [--phase {lex,parse,semantic}] [--tokens] [--symbols] [--ast]
```
`--phase` maps `lex`→`Phase.LEXICAL`, `parse`→`Phase.SYNTAX`, `semantic`→`Phase.SEMANTIC` (default).
File read errors: stderr `error: cannot read file '<path>': <reason>`, exit 2. The source path is **never** printed on stdout.

### 7.2 Report layout (exact) — sections in this order; each header is its own line
`format_report` returns the printed sections joined by **one blank line** (`"\n\n"`); each section is its header
line followed by its body lines (`"\n"`-joined). The returned string has **no** trailing newline; `main` prints it
with `print()` (so stdout ends with exactly one `\n`).
| Section header | Printed when | Body |
|---|---|---|
| `=== Tokens ===` | phase is lex, or `--tokens` | `Token.format()` per token, EOF excluded; `(none)` if no tokens |
| `=== Symbol Table ===` | phase is lex, or (`--symbols` and phase is parse/semantic) | `format_identifier_table` |
| `=== AST ===` | parse succeeded and (phase is parse, or `--ast`) | `format_ast` |
| `=== Scoped Symbol Table ===` | semantic analysis ran | `format_scoped_table` |
| `=== Errors ===` | always | `Diagnostic.format()` per diagnostic, or `(none)` |
| `=== Result ===` | always | exactly one line from §7.3 |

### 7.3 Result lines (exact; `N` = number of diagnostics)
- `Lexical analysis completed successfully with no errors.` / `Lexical analysis failed with N error(s).` (phase lex)
- `Compilation stopped: N lexical error(s).` (phase parse/semantic with lexical errors)
- `Syntax analysis completed successfully with no errors.` / `Syntax analysis failed with N error(s).` (phase parse)
- `Compilation stopped: N syntax error(s).` (phase semantic with syntax error)
- `Semantic analysis completed successfully with no errors.` / `Semantic analysis failed with N error(s).`

### 7.4 Identifier table text
One line per entry: `Index: 1, Name: Student, First seen: line 1, column 7, Occurrences: 1`. Empty table: `(empty)`.

### 7.5 Scoped symbol table text
Pre-order; scope header `Scope <name> (<KIND>)`; each symbol at +2 spaces: `<KIND> <name> : <type> at line L, column C`
(method type via `format_method_type`, e.g. `METHOD add : int(int, int) at line 3, column 16`); child scopes at +2.

### 7.6 Example (Phase 1, PDF p4 sample)
`python3 -m minijava sample.mj --phase lex` prints the 41 `Lexeme: …, Token: …` lines exactly as on PDF p4–p5,
then the symbol table (`Student`, `age`, `grade`, `name`), `=== Errors ===`, `(none)`, `=== Result ===`,
`Lexical analysis completed successfully with no errors.` — exit code 0.

---

## 8. Dependency rules and permitted imports

| Module | May import (project) | Extra stdlib allowed |
|---|---|---|
| `tokens` | — | `dataclasses`, `enum` |
| `diagnostics` | — | `dataclasses`, `enum` |
| `ast_nodes` | — | `dataclasses`, `typing` |
| `mj_types` | `ast_nodes` | `dataclasses` |
| `symbol_table` | `mj_types` | `dataclasses`, `enum` |
| `lexer` | `tokens`, `diagnostics`, `symbol_table` | `dataclasses` |
| `parser` | `tokens`, `diagnostics`, `ast_nodes` | `dataclasses` |
| `ast_printer` | `ast_nodes` | — |
| `semantic` | `ast_nodes`, `diagnostics`, `mj_types`, `symbol_table` | `dataclasses` |
| `driver` | all of the above | `argparse`, `sys`, `dataclasses` |
| `__main__` | `driver` | `sys` |
| tests | anything in `minijava`, `tests.helpers` | `unittest`, `io`, `contextlib`, `pathlib`, `subprocess`, `sys`, `inspect`, `os`, `tempfile` |

No module may import `driver` except `__main__` and tests. No cycles.

---

## 9. Naming/formatting conventions
See §4. Additionally: test classes `Test<Thing>`, test methods `test_<behavior>`; golden files `NN_short_name.mj` / `.out` (e.g. `03_unclosed_string.mj`).

---

## 10. Error-handling and diagnostic conventions
- One shared `Diagnostic` type (Appendix B). Message text comes **only** from `MESSAGE_TEMPLATES`; components pass arguments, never free text.
- Lexical messages reproduce the PDF p3–p4 wording exactly, e.g.:
  `Error: invalid character '@' at line 1, column 12.` · `Error: invalid identifier '1name' at line 1, column 1.` ·
  `Error: unclosed string literal starting at line 1, column 15.` · `Error: unclosed comment starting at line 1, column 1.`
- Create diagnostics only via `make_diagnostic(code, line, column, **args)`; all arg values are `str`.
- Exceptions: only for contract violations (`ValueError`, `RuntimeError`) and I/O (driver). A component must never crash on any user input; unexpected internal exceptions are bugs.

---

## 11. Build and execution instructions
See §5. No build step beyond byte-compilation. No virtual environment needed (no dependencies).

---

## 12. Testing conventions and required coverage

### 12.1 Framework
`unittest` only. All tests run with `python3 -m unittest discover -s tests -t . -v` and must pass offline in < 30 s.

### 12.2 Golden tests
- Location: `tests/golden/<lex|parse|semantic>/NN_name.mj` with sibling `NN_name.out`.
- `.out` = exact stdout of `python3 -m minijava NN_name.mj --phase <dir>` (no extra flags).
- Comparison: both normalised with `\r\n`→`\n` and trailing newlines stripped.
- Expected exit code: `1` if any line of `.out` starts with `Error: `, else `0`.
- Each golden `.mj` begins with a comment stating the case's purpose.
- `.out` files are **hand-derived from this contract and the PDF**, never produced by running the code under test
  and copying its output. The author of a golden case must not be the owner of the component it primarily tests.
- `test_golden.py` and `test_contract.py` **skip** (via `unittest.SkipTest`) — not fail — cases whose required module
  (`minijava.parser` for parse, `minijava.semantic` for semantic) cannot be imported. Once the module is merged, the
  skip disappears automatically.

### 12.3 `tests/helpers.py` (M1) provides
```python
PROJECT_ROOT: pathlib.Path
GOLDEN_DIR: pathlib.Path
def run_cli(args: list[str]) -> tuple[int, str, str]: ...    # in-process driver.main, returns (exit_code, stdout, stderr)
def lex(source: str) -> LexResult: ...
def tokens_of(source: str) -> list[tuple[str, str]]: ...      # [(TokenType name, lexeme)], EOF excluded
def parse_source(source: str) -> ParseResult: ...              # lex then parse; AssertionError if lexical errors
def analyze_source(source: str) -> SemanticResult: ...         # lex, parse, analyze; AssertionError if earlier errors
def diagnostic_lines(diags: list[Diagnostic]) -> list[str]: ...
```
Until M1 delivers `helpers.py`, members may use local private helpers inside their own test files.

### 12.4 Minimum coverage per component
| Component | Minimum |
|---|---|
| lexer | every TokenType; every error code; all 4 PDF error examples; PDF sample; positions; comments; escapes; edge cases (§6.2) — ≥ 40 test methods |
| symbol_table / mj_types | every public method; every operator × type class in §6.6 — ≥ 40 test methods total |
| parser | every grammar production; precedence/associativity; dangling else; every SYN code — ≥ 40 test methods |
| ast_printer | every node type and label rule — ≥ 20 test methods; ≥ 10 parse golden cases |
| semantic | every SEM code at least twice; scope rules; built-ins — ≥ 45 test methods; ≥ 15 semantic golden cases |
| driver | every section/result line, exit codes, file errors — ≥ 15 test methods |
| golden lex | ≥ 12 cases including PDF p3–p5 examples |

---

## 13. Integration rules
1. Integration happens on the `integration` branch, owned by M1, in the order of `INTEGRATION_AND_TESTING_PLAN.md`.
2. A merge is accepted only if `compileall` and the **entire** test suite pass on the integration branch after the merge.
3. `tests/test_contract.py` (M1) must pass; it checks names/signatures of every contracted symbol present.
4. Nobody merges their own branch into `integration`/`main`; M1 merges, M1's own branches are merged by M4.

## 14. Restrictions on modifying shared files
- `tokens.py`, `diagnostics.py`, `ast_nodes.py` are created by M1 **verbatim** from the appendices and are then **frozen**. Only an approved change request can alter them.
- Each member edits only the files they own (§3). Needing a change in someone else's file → open an issue / message to the owner; never edit it.
- AI assistants must never regenerate, reformat or "clean up" files they do not own.

## 15. Interface change procedure
1. Proposer writes a change request in `planning/CONTRACT_CHANGES.md`: `CR-NNN`, date, proposer, affected sections/files, problem, proposed exact text, impact per member.
2. Approval required from M1 **and** every owner of an affected file (recorded by name in the CR).
3. M1 updates this contract, bumps the version (patch = wording, minor = additive interface, major = breaking), regenerates the five member prompts, and notifies all members.
4. Until approved, everyone keeps implementing the current version. No silent workarounds.

## 16. Definition of done (every component)
A component is done only when: (a) it implements all contracted behavior with no stubs; (b) public names/signatures match this contract exactly and `test_contract.py` passes; (c) its owner's tests meet §12.4 and pass; (d) the full suite and `compileall` pass on the integration branch after merge; (e) it touches no files outside its ownership; (f) it imports only what §8 allows; (g) a completion report (format in each member prompt) was delivered; (h) the owner can explain the code in the team discussion (R-04).

## 17. Examples of how components connect

```python
from minijava.lexer import tokenize
from minijava.parser import parse
from minijava.semantic import analyze
from minijava.ast_printer import format_ast

lex_result = tokenize("int x = 10 @ 5;")
[d.format() for d in lex_result.diagnostics]
# ["Error: invalid character '@' at line 1, column 12."]
[(t.type.name, t.lexeme) for t in lex_result.tokens]
# [('INT','int'), ('IDENTIFIER','x'), ('ASSIGN','='), ('NUMBER','10'), ('NUMBER','5'), ('SEMICOLON',';'), ('EOF','')]

src = "class A { int f(int a) { return a + 1; } }"
lr = tokenize(src); pr = parse(lr.tokens)
print(format_ast(pr.program))
# Program
#   ClassDecl A
#     MethodDecl int f(int a)
#       Block
#         Return
#           Binary +
#             Identifier a
#             Literal NUMBER 1

sr = analyze(pr.program)
sr.diagnostics   # []

pr2 = parse(tokenize("class A { void m() { int x = true; y = 1; } }").tokens)
[d.format() for d in analyze(pr2.program).diagnostics]
# ["Error: semantic error at line 1, column 26: type mismatch: cannot assign a value of type 'boolean' to a target of type 'int'.",
#  "Error: semantic error at line 1, column 36: 'y' is not declared."]

pr3 = parse(tokenize("class A { void m() { int x = 1 } }").tokens)
pr3.program, [d.format() for d in pr3.diagnostics]
# (None, ["Error: syntax error at line 1, column 32: expected ';' but found '}'."])
```

---

## Appendix A — `minijava/tokens.py` (verbatim)

```python
"""Shared token definitions for the Mini Java compiler (contract v1.0.0).

Do not modify without an approved change request (SHARED_PROJECT_CONTRACT.md section 15).
"""
from __future__ import annotations

from dataclasses import dataclass
from enum import Enum


class TokenType(Enum):
    """Every token category produced by the lexer. The value is the printed token name."""

    # Reserved words
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
    # Identifiers and literals
    IDENTIFIER = "IDENTIFIER"
    NUMBER = "NUMBER"
    STRING = "STRING"
    BOOLEAN = "BOOLEAN"
    # Operators
    MATHOP = "MATHOP"
    LOGOP = "LOGOP"
    ASSIGN = "ASSIGN"
    # Delimiters / punctuation
    LPAR = "LPAR"
    RPAR = "RPAR"
    LCUR = "LCUR"
    RCUR = "RCUR"
    LSQU = "LSQU"
    RSQU = "RSQU"
    SEMICOLON = "SEMICOLON"
    COMMA = "COMMA"
    DOT = "DOT"
    # End of input
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
    """One lexical token. line/column are 1-based and point at the first character."""

    type: TokenType
    lexeme: str
    line: int
    column: int

    def format(self) -> str:
        """Return the listing line, e.g. 'Lexeme: class, Token: CLASS'."""
        return f"Lexeme: {self.lexeme}, Token: {self.type.value}"

    def describe(self) -> str:
        """Return how the token is named in syntax errors: quoted lexeme or 'end of input'."""
        if self.type is TokenType.EOF:
            return "end of input"
        return f"'{self.lexeme}'"
```

## Appendix B — `minijava/diagnostics.py` (verbatim)

```python
"""Shared diagnostic (error) definitions for the Mini Java compiler (contract v1.0.0).

Do not modify without an approved change request (SHARED_PROJECT_CONTRACT.md section 15).
"""
from __future__ import annotations

from dataclasses import dataclass, field
from enum import Enum


class Phase(Enum):
    """Compiler phase that produced a diagnostic."""

    LEXICAL = "lexical"
    SYNTAX = "syntax"
    SEMANTIC = "semantic"


class ErrorCode(Enum):
    """Every error the compiler can report. The name prefix (LEX_/SYN_/SEM_) gives the phase."""

    LEX_INVALID_CHARACTER = "LEX_INVALID_CHARACTER"
    LEX_INVALID_IDENTIFIER = "LEX_INVALID_IDENTIFIER"
    LEX_UNCLOSED_STRING = "LEX_UNCLOSED_STRING"
    LEX_UNCLOSED_COMMENT = "LEX_UNCLOSED_COMMENT"
    SYN_UNEXPECTED_TOKEN = "SYN_UNEXPECTED_TOKEN"
    SYN_INVALID_ASSIGNMENT_TARGET = "SYN_INVALID_ASSIGNMENT_TARGET"
    SYN_NOT_A_STATEMENT = "SYN_NOT_A_STATEMENT"
    SEM_UNDECLARED_IDENTIFIER = "SEM_UNDECLARED_IDENTIFIER"
    SEM_DUPLICATE_DECLARATION = "SEM_DUPLICATE_DECLARATION"
    SEM_NOT_A_VARIABLE = "SEM_NOT_A_VARIABLE"
    SEM_NOT_A_METHOD = "SEM_NOT_A_METHOD"
    SEM_TYPE_MISMATCH = "SEM_TYPE_MISMATCH"
    SEM_INVALID_ARRAY_INITIALIZER = "SEM_INVALID_ARRAY_INITIALIZER"
    SEM_INVALID_BINARY_OPERANDS = "SEM_INVALID_BINARY_OPERANDS"
    SEM_INVALID_UNARY_OPERAND = "SEM_INVALID_UNARY_OPERAND"
    SEM_CONDITION_NOT_BOOLEAN = "SEM_CONDITION_NOT_BOOLEAN"
    SEM_RETURN_TYPE_MISMATCH = "SEM_RETURN_TYPE_MISMATCH"
    SEM_RETURN_VALUE_IN_VOID = "SEM_RETURN_VALUE_IN_VOID"
    SEM_MISSING_RETURN_VALUE = "SEM_MISSING_RETURN_VALUE"
    SEM_ARGUMENT_COUNT = "SEM_ARGUMENT_COUNT"
    SEM_ARGUMENT_TYPE = "SEM_ARGUMENT_TYPE"
    SEM_NOT_AN_ARRAY = "SEM_NOT_AN_ARRAY"
    SEM_INDEX_NOT_INT = "SEM_INDEX_NOT_INT"
    SEM_INVALID_MEMBER_ACCESS = "SEM_INVALID_MEMBER_ACCESS"
    SEM_UNKNOWN_MEMBER = "SEM_UNKNOWN_MEMBER"
    SEM_VOID_VALUE = "SEM_VOID_VALUE"


_SYN = "syntax error at line {line}, column {column}: "
_SEM = "semantic error at line {line}, column {column}: "

MESSAGE_TEMPLATES: dict[ErrorCode, str] = {
    ErrorCode.LEX_INVALID_CHARACTER: "invalid character '{lexeme}' at line {line}, column {column}.",
    ErrorCode.LEX_INVALID_IDENTIFIER: "invalid identifier '{lexeme}' at line {line}, column {column}.",
    ErrorCode.LEX_UNCLOSED_STRING: "unclosed string literal starting at line {line}, column {column}.",
    ErrorCode.LEX_UNCLOSED_COMMENT: "unclosed comment starting at line {line}, column {column}.",
    ErrorCode.SYN_UNEXPECTED_TOKEN: _SYN + "expected {expected} but found {found}.",
    ErrorCode.SYN_INVALID_ASSIGNMENT_TARGET: _SYN + "invalid assignment target.",
    ErrorCode.SYN_NOT_A_STATEMENT: _SYN + "expression is not a valid statement.",
    ErrorCode.SEM_UNDECLARED_IDENTIFIER: _SEM + "'{name}' is not declared.",
    ErrorCode.SEM_DUPLICATE_DECLARATION: _SEM + "'{name}' is already declared "
                                                "(previous declaration at line {prev_line}, "
                                                "column {prev_column}).",
    ErrorCode.SEM_NOT_A_VARIABLE: _SEM + "'{name}' is not a variable.",
    ErrorCode.SEM_NOT_A_METHOD: _SEM + "'{name}' is not a method.",
    ErrorCode.SEM_TYPE_MISMATCH: _SEM + "type mismatch: cannot assign a value of type '{actual}' "
                                        "to a target of type '{expected}'.",
    ErrorCode.SEM_INVALID_ARRAY_INITIALIZER: _SEM + "array initializer cannot be assigned to "
                                                    "non-array type '{expected}'.",
    ErrorCode.SEM_INVALID_BINARY_OPERANDS: _SEM + "operator '{op}' cannot be applied to types "
                                                  "'{left}' and '{right}'.",
    ErrorCode.SEM_INVALID_UNARY_OPERAND: _SEM + "operator '{op}' cannot be applied to type "
                                                "'{operand}'.",
    ErrorCode.SEM_CONDITION_NOT_BOOLEAN: _SEM + "condition of '{statement}' must be of type "
                                                "'boolean' but found '{actual}'.",
    ErrorCode.SEM_RETURN_TYPE_MISMATCH: _SEM + "method '{method}' must return a value of type "
                                               "'{expected}' but found '{actual}'.",
    ErrorCode.SEM_RETURN_VALUE_IN_VOID: _SEM + "method '{method}' is declared 'void' and cannot "
                                               "return a value.",
    ErrorCode.SEM_MISSING_RETURN_VALUE: _SEM + "method '{method}' must return a value of type "
                                               "'{expected}'.",
    ErrorCode.SEM_ARGUMENT_COUNT: _SEM + "method '{method}' expects {expected} argument(s) but "
                                         "{actual} were given.",
    ErrorCode.SEM_ARGUMENT_TYPE: _SEM + "argument {position} of method '{method}' expects type "
                                        "'{expected}' but found '{actual}'.",
    ErrorCode.SEM_NOT_AN_ARRAY: _SEM + "type '{actual}' is not an array and cannot be indexed.",
    ErrorCode.SEM_INDEX_NOT_INT: _SEM + "array index must be of type 'int' but found '{actual}'.",
    ErrorCode.SEM_INVALID_MEMBER_ACCESS: _SEM + "cannot access member '{name}' on a value of type "
                                                "'{actual}'.",
    ErrorCode.SEM_UNKNOWN_MEMBER: _SEM + "class '{class_name}' has no member named '{name}'.",
    ErrorCode.SEM_VOID_VALUE: _SEM + "method '{method}' returns 'void' and cannot be used as a "
                                     "value.",
}

REQUIRED_ARGS: dict[ErrorCode, tuple[str, ...]] = {
    ErrorCode.LEX_INVALID_CHARACTER: ("lexeme",),
    ErrorCode.LEX_INVALID_IDENTIFIER: ("lexeme",),
    ErrorCode.LEX_UNCLOSED_STRING: (),
    ErrorCode.LEX_UNCLOSED_COMMENT: (),
    ErrorCode.SYN_UNEXPECTED_TOKEN: ("expected", "found"),
    ErrorCode.SYN_INVALID_ASSIGNMENT_TARGET: (),
    ErrorCode.SYN_NOT_A_STATEMENT: (),
    ErrorCode.SEM_UNDECLARED_IDENTIFIER: ("name",),
    ErrorCode.SEM_DUPLICATE_DECLARATION: ("name", "prev_line", "prev_column"),
    ErrorCode.SEM_NOT_A_VARIABLE: ("name",),
    ErrorCode.SEM_NOT_A_METHOD: ("name",),
    ErrorCode.SEM_TYPE_MISMATCH: ("actual", "expected"),
    ErrorCode.SEM_INVALID_ARRAY_INITIALIZER: ("expected",),
    ErrorCode.SEM_INVALID_BINARY_OPERANDS: ("op", "left", "right"),
    ErrorCode.SEM_INVALID_UNARY_OPERAND: ("op", "operand"),
    ErrorCode.SEM_CONDITION_NOT_BOOLEAN: ("statement", "actual"),
    ErrorCode.SEM_RETURN_TYPE_MISMATCH: ("method", "expected", "actual"),
    ErrorCode.SEM_RETURN_VALUE_IN_VOID: ("method",),
    ErrorCode.SEM_MISSING_RETURN_VALUE: ("method", "expected"),
    ErrorCode.SEM_ARGUMENT_COUNT: ("method", "expected", "actual"),
    ErrorCode.SEM_ARGUMENT_TYPE: ("position", "method", "expected", "actual"),
    ErrorCode.SEM_NOT_AN_ARRAY: ("actual",),
    ErrorCode.SEM_INDEX_NOT_INT: ("actual",),
    ErrorCode.SEM_INVALID_MEMBER_ACCESS: ("name", "actual"),
    ErrorCode.SEM_UNKNOWN_MEMBER: ("class_name", "name"),
    ErrorCode.SEM_VOID_VALUE: ("method",),
}

_PHASE_BY_PREFIX: dict[str, Phase] = {
    "LEX": Phase.LEXICAL,
    "SYN": Phase.SYNTAX,
    "SEM": Phase.SEMANTIC,
}


@dataclass
class Diagnostic:
    """One compiler error. Build instances with make_diagnostic()."""

    code: ErrorCode
    line: int
    column: int
    args: dict[str, str] = field(default_factory=dict)

    @property
    def phase(self) -> Phase:
        """Phase derived from the error code prefix."""
        return _PHASE_BY_PREFIX[self.code.name[:3]]

    def message(self) -> str:
        """Message text without the 'Error: ' prefix."""
        return MESSAGE_TEMPLATES[self.code].format(line=self.line, column=self.column, **self.args)

    def format(self) -> str:
        """Full printed line, e.g. "Error: invalid character '@' at line 1, column 12."."""
        return "Error: " + self.message()


def make_diagnostic(code: ErrorCode, line: int, column: int, **args: str) -> Diagnostic:
    """Create a Diagnostic, validating that exactly the required string arguments are given."""
    required = REQUIRED_ARGS[code]
    if set(args) != set(required):
        raise ValueError(f"{code.name} requires args {required}, got {tuple(sorted(args))}")
    for key, value in args.items():
        if not isinstance(value, str):
            raise ValueError(f"argument '{key}' of {code.name} must be str")
    if line < 1 or column < 1:
        raise ValueError("line and column are 1-based")
    return Diagnostic(code=code, line=line, column=column, args=dict(args))
```

## Appendix C — `minijava/ast_nodes.py` (verbatim)

```python
"""Shared abstract syntax tree node definitions for Mini Java (contract v1.0.0).

Produced by the parser, consumed (read-only) by the AST printer and semantic analyzer.
Do not modify without an approved change request (SHARED_PROJECT_CONTRACT.md section 15).
Always construct nodes with keyword arguments. See contract section 6.4 for positions.
"""
from __future__ import annotations

from dataclasses import dataclass, field
from typing import Optional, Union


@dataclass(kw_only=True)
class Node:
    """Base class. line/column are 1-based (see contract section 6.4)."""

    line: int
    column: int


@dataclass(kw_only=True)
class TypeNode(Node):
    """A written type. name is 'int','float','boolean','char','String' or 'void'."""

    name: str
    is_array: bool = False


# ----------------------------------------------------------------- expressions
@dataclass(kw_only=True)
class Expr(Node):
    """Base class for expressions."""


@dataclass(kw_only=True)
class Literal(Expr):
    """kind is 'NUMBER', 'STRING' or 'BOOLEAN'; lexeme is the exact token lexeme (quotes kept)."""

    kind: str
    lexeme: str


@dataclass(kw_only=True)
class Identifier(Expr):
    name: str


@dataclass(kw_only=True)
class Binary(Expr):
    """op is one of + - * / % == != < > <= >= && ||."""

    op: str
    left: Expr
    right: Expr


@dataclass(kw_only=True)
class Unary(Expr):
    """op is '-' or '!'."""

    op: str
    operand: Expr


@dataclass(kw_only=True)
class IndexExpr(Expr):
    array: Expr
    index: Expr


@dataclass(kw_only=True)
class FieldAccess(Expr):
    target: Expr
    name: str


@dataclass(kw_only=True)
class CallExpr(Expr):
    """callee is an Identifier or a FieldAccess."""

    callee: Expr
    args: list[Expr] = field(default_factory=list)


@dataclass(kw_only=True)
class ArrayInit(Node):
    """Array initializer '{e1, e2}'. Only valid as a declaration initializer (not an Expr)."""

    elements: list[Expr] = field(default_factory=list)


# ------------------------------------------------------------------ statements
@dataclass(kw_only=True)
class Stmt(Node):
    """Base class for statements."""


@dataclass(kw_only=True)
class Block(Stmt):
    statements: list[Stmt] = field(default_factory=list)


@dataclass(kw_only=True)
class VarDecl(Stmt):
    var_type: TypeNode
    name: str
    initializer: Optional[Union[Expr, ArrayInit]] = None


@dataclass(kw_only=True)
class Assign(Stmt):
    """target is an Identifier, IndexExpr or FieldAccess."""

    target: Expr
    value: Expr


@dataclass(kw_only=True)
class ExprStmt(Stmt):
    expression: CallExpr


@dataclass(kw_only=True)
class If(Stmt):
    condition: Expr
    then_branch: Stmt
    else_branch: Optional[Stmt] = None


@dataclass(kw_only=True)
class While(Stmt):
    condition: Expr
    body: Stmt


@dataclass(kw_only=True)
class For(Stmt):
    init: Optional[Union[VarDecl, Assign, ExprStmt]]
    condition: Optional[Expr]
    update: Optional[Union[Assign, ExprStmt]]
    body: Stmt


@dataclass(kw_only=True)
class Return(Stmt):
    value: Optional[Expr] = None


# ---------------------------------------------------------------- declarations
@dataclass(kw_only=True)
class Param(Node):
    param_type: TypeNode
    name: str


@dataclass(kw_only=True)
class FieldDecl(Node):
    """modifiers is a sublist of ['public', 'static'] in source order."""

    modifiers: list[str]
    var_type: TypeNode
    name: str
    initializer: Optional[Union[Expr, ArrayInit]] = None


@dataclass(kw_only=True)
class MethodDecl(Node):
    """is_main is True only for 'void main(...)'; then name == 'main'."""

    modifiers: list[str]
    return_type: TypeNode
    name: str
    params: list[Param]
    body: Block
    is_main: bool = False


@dataclass(kw_only=True)
class ClassDecl(Node):
    name: str
    members: list[Union[FieldDecl, MethodDecl]] = field(default_factory=list)


@dataclass(kw_only=True)
class Program(Node):
    classes: list[ClassDecl] = field(default_factory=list)
```

*End of SHARED_PROJECT_CONTRACT.md v1.0.0*

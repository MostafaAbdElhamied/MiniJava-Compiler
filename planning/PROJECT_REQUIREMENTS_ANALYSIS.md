# Mini Java Compiler: Comprehensive Requirements Analysis

**Course:** CSCI415 – Compiler Design (Fall 2026)  
**Project Title:** Mini Compiler for a Java-like Language (Mini Java)  
**Authoritative Specification:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)  
**Project Workspace:** `/home/yousefmahmoudali/Desktop/CompilerProject/`  
**Document Classification:** Technical Specification & Requirements Analysis  

---

## 1. Project Overview

### 1.1 Objective and Scope
The objective of this project is to design, implement, test, and integrate a complete compiler front-end for **Mini Java**, a statically typed, case-sensitive subset of the Java programming language. 

As explicitly stated in the official guidelines (*PDF p. 1*):
> *"The final submission MUST perform lexical, syntax, and semantic analysis on Mini Java source programs."*

The compiler is structured into three progressive, cumulative phases:
1. **Phase 01: Lexical Analyzer (Scanner):** Reads Mini Java source text character by character, skips whitespace and comments, identifies and classifies tokens, tracks 1-based source positions (line and column), constructs an identifier symbol table, and detects/reports lexical errors.
2. **Phase 02: Syntax Analyzer (Parser):** Consumes the token stream produced by Phase 01, verifies grammatical structure according to the Mini Java grammar using a hand-crafted parser (recursive descent), constructs an Abstract Syntax Tree (AST), and reports syntax errors.
3. **Phase 03: Semantic Analyzer (Type & Scope Checker):** Traverses the AST produced by Phase 02, manages multi-level nested scopes (global, class, method, block), validates type consistency across expressions, assignments, method calls, and control-flow statements, enforces declaration-before-use rules, and detects semantic errors.

**Explicit Scope Boundary:**
The official project specification specifies compiler front-end stages only (lexical, syntax, and semantic analysis). No intermediate code generation (IR), bytecode generation, target assembly/machine code generation, or optimization stages are required. None will be built.

---

### 1.2 Timeline and Deadlines
The project timeline is defined on *PDF p. 1*:

| Milestone / Phase | Start Date | Deadline | Scope & Deliverables |
|---|---|---|---|
| **Teams Registration** | Oct 04, 2026 | Oct 10, 2026 | Team of 5 members registered |
| **Phase 01: Lexical Analysis** | Oct 12, 2026 | **Oct 30, 2026** | Scanner implementation, Identifier symbol table, input test programs, Phase 1 technical report |
| **Phase 02: Syntax Analysis** | Nov 21, 2026 | **Dec 20, 2026** | Hand-crafted parser, AST generation, AST printer, syntax error diagnostics |
| **Phase 03: Semantic Analysis** | Nov 21, 2026 | **Dec 20, 2026** | Scoped symbol table, type checker, semantic error diagnostics, final integrated submission |

> [!IMPORTANT]
> *PDF p. 1 Note:* *"Take into consideration that all phases must be able to build on each other to avoid issues."*  
> Phase 02 directly consumes Phase 01's token stream; Phase 03 directly consumes Phase 02's AST and leverages the symbol table abstractions. Architectural decisions made in Phase 01 must accommodate Phases 02 and 03 seamlessly.

---

### 1.3 Permitted Tools, Languages, and Technical Constraints
- **Implementation Language:** *PDF p. 1* permits any programming language (*"You may use any coding language you prefer to complete the project"*). The team has selected **Python 3 (≥ 3.10)** due to its expressive standard library (`dataclasses`, `enum`, `typing`), cross-platform determinism, readability for university code inspection, and absence of complex build toolchains.
- **Strict Prohibition on Compiler Generators & Direct Helper Functions:**
  *PDF p. 1* states:
  > *"DO NOT use ready-made lexical analysis/parser libraries or ready functions that directly solve the required compiler tasks; build your own methods."*
  
  This is a critical grading constraint:
  - **Prohibited:** Lex/Flex, Yacc/Bison, ANTLR, PLY, Lark, SLY, PyParsing, JavaCC.
  - **Prohibited:** Standard library shortcuts that bypass compiler fundamentals (e.g., Python's `re` regular expressions for lexing, `tokenize`, `ast`, `shlex`, or naive `str.split()` / `str.find()` token splitting).
  - **Mandatory:** The lexical analyzer must be hand-coded, processing source characters sequentially using an index/cursor. The syntax analyzer must be a hand-coded recursive-descent parser.
- **Academic Integrity & Oral Examination:**
  *PDF p. 1–2* explicitly states:
  - *"A discussion will be held with ALL team members; all members must participate in implementation."*
  - *"ANY PLAGIARISM DETECTED WILL LEAD TO A ZERO AND NO DISCUSSION WILL BE CONDUCTED FOR THE TEAM."*
  Every team member must be able to walk through and explain their code during the oral grading interview.

---

### 1.4 Source Language Specification (Mini Java)

#### Case Sensitivity (*PDF p. 2*)
Mini Java is strictly case-sensitive. Keywords and identifiers differing only in case (e.g., `class` vs `Class`, `int` vs `Int`) are distinct lexemes.

#### Reserved Keywords (*PDF p. 2, 4*)
The language reserves the following keywords:
`class`, `public`, `static`, `void`, `int`, `float`, `boolean`, `char`, `String`, `if`, `else`, `while`, `for`, `return`.
- **Special Keyword Rule:** `String` is classified as `STRING_TYPE` (a type keyword), whereas string literal values enclosed in quotes are classified as `STRING` (*PDF p. 2*).
- **Sample Output Precedent:** `main` appears in the sample output on *PDF p. 4* with the token name `MAIN` (`Lexeme: main, Token: MAIN`). It is reserved as a dedicated keyword token to match the required output format.
- **Boolean Literal Keywords:** `true` and `false` are literals classified as `BOOLEAN` (*PDF p. 2*).

#### Identifiers (*PDF p. 2*)
- Must begin with an ASCII letter (`[A-Za-z]`) or an underscore (`_`).
- Subsequent characters may include letters, digits (`[0-9]`), and underscores.
- **Invalid Identifier Detection:** If a sequence starts with a digit and is immediately followed by letters or underscores (e.g., `1name`, `3count`), it must be detected and reported as an invalid identifier error rather than scanned as a number followed by an identifier (*PDF p. 2, 3*).

#### Literals (*PDF p. 2*)
- `NUMBER`: Any integer or floating-point numeric value (e.g., `10`, `0`, `25.5`).
- `STRING`: Any value enclosed in single or double quotation marks (e.g., `"Hello"`, `'A'`). The token lexeme preserves the opening and closing quotation marks.
- `BOOLEAN`: The boolean values `true` and `false`.

#### Delimiters / Punctuation (*PDF p. 3*)
The language specifies 9 standard delimiters:
- `(` -> `LPAR`
- `)` -> `RPAR`
- `{` -> `LCUR`
- `}` -> `RCUR`
- `[` -> `LSQU`
- `]` -> `RSQU`
- `;` -> `SEMICOLON`
- `,` -> `COMMA`
- `.` -> `DOT`

#### Operators (*PDF p. 3, 4, 5*)
The PDF specifies *"Arithmetic and comparison operators"*, with concrete assignments evidenced in the sample output:
- Arithmetic (`MATHOP`): `+`, `-`, `*`, `/`, `%`
- Relational / Logical (`LOGOP`): `==`, `!=`, `<`, `>`, `<=`, `>=`, `&&`, `||`, `!`
- Assignment (`ASSIGN`): `=` (sample output *PDF p. 4–5* maps `=` to `ASSIGN`, while `>=` maps to `LOGOP` and `+` maps to `MATHOP`).

#### Comments (*PDF p. 3*)
- Single-line comments: begin with `//` and extend to the end of the line.
- Multi-line comments: begin with `/*` and terminate at `*/`.
- Comments are completely skipped by the lexer and do not produce tokens.

---

### 1.5 Diagnostic and Error Reporting Requirements
The compiler must detect, format, and report errors across all phases (*PDF p. 3–4*). Every error diagnostic must include:
1. Line number (1-based)
2. Column number (1-based)
3. A descriptive, precise error message

**Required Lexical Error Forms (*PDF p. 3–4*):**
1. **Unknown or invalid symbols:**
   *Example:* `int x = 10 @ 5;`  
   *Output:* `Error: invalid character '@' at line 1, column 12.`
2. **Invalid identifiers:**
   *Example:* `1name = 5;`  
   *Output:* `Error: invalid identifier '1name' at line 1, column 1.`
3. **Unclosed quoted string literals:**
   *Example:* `String name = "Ali;`  
   *Output:* `Error: unclosed string literal starting at line 1, column 15.`
4. **Unterminated multi-line comments:**
   *Example:* `/* This comment does not end`  
   *Output:* `Error: unclosed comment starting at line 1, column 1.`

---

## 2. Requirements Traceability Matrix

| ID | Requirement Statement | Classification | Source Page | Planned Implementation | Verification Method |
|---|---|---|---|---|---|
| **REQ-01** | Compiler front-end performing lexical, syntax, and semantic analysis on Mini Java source programs | Mandatory | p. 1 | Modular architecture with `minijava/lexer.py`, `minijava/parser.py`, `minijava/semantic.py` executed via `minijava/driver.py` | Full compiler pipeline CLI execution tests (`test_driver.py`, end-to-end golden tests) |
| **REQ-02** | Three sequential phases building upon each other | Mandatory | p. 1 | Lexer outputs `list[Token]` to Parser; Parser outputs `Program` AST to Semantic Analyzer | Integration contract tests (`test_contract.py`) verifying pipeline data flow |
| **REQ-03** | Use any coding language preferred | Mandatory | p. 1 | Python 3 (≥ 3.10) with standard library only | Byte-compilation with `python3 -m compileall`, version checks |
| **REQ-04** | All 5 team members must participate in implementation; discussion with all members | Mandatory | p. 1 | Distinct 5-way task allocation, file ownership rules, clear per-member deliverables | Git commit logs, oral examination readiness walkthrough |
| **REQ-05** | No ready-made lexical analysis/parser libraries or shortcut helper functions | Mandatory | p. 1 | Hand-written character scanner loop (`lexer.py`), hand-written recursive-descent parser (`parser.py`); no `re`, `ply`, `antlr` | Source code inspection, zero third-party dependencies, banned module checks |
| **REQ-06** | Zero plagiarism tolerance | Mandatory | p. 2 | Clean-room original implementation adhering strictly to shared project contract | Automated code audits, peer reviews |
| **REQ-07** | Lexer reads source character by character, groups into tokens, classifies them | Mandatory | p. 2 | `Lexer.tokenize()` iterating character index with line/column tracking | Unit tests covering every token category (`test_lexer.py`) |
| **REQ-08** | Ignore whitespace and comments (`//`, `/* ... */`) | Mandatory | p. 2, 3 | Lexer whitespace skipper; comment scanner skipping to newline or `*/` | Unit tests for single-line, multi-line, consecutive, and trailing comments |
| **REQ-09** | Detect illegal characters and report exact line and column positions | Mandatory | p. 2, 3 | Error detection for characters outside grammar (e.g. `@`, `#`, `$`, lone `&`, lone `\|`); records `Diagnostic` | `test_lexer.py` and PDF p. 3 example regression test |
| **REQ-10** | Case-sensitive language (`class` != `Class`) | Mandatory | p. 2 | Exact case comparisons against `KEYWORDS` dictionary | Case sensitivity unit tests (`class` keyword vs `Class` identifier) |
| **REQ-11** | Generate token stream consumed by Phase 2 parser | Mandatory | p. 2 | `LexResult.tokens` producing frozen `Token` instances terminating in a single `EOF` token | Parser unit tests consuming `LexResult.tokens` |
| **REQ-12** | Construct and maintain a symbol table for identifiers | Mandatory | p. 2 | `IdentifierTable` tracking index, name, first location, and all occurrence locations | `test_symbol_table.py` checking entry creation and lookup |
| **REQ-13** | Reserved keywords: `class`, `public`, `static`, `void`, `int`, `float`, `boolean`, `char`, `String`, `if`, `else`, `while`, `for`, `return`; `String` -> `STRING_TYPE` | Mandatory | p. 2 | `KEYWORDS` mapping in `tokens.py` mapping keyword strings to `TokenType` enums | Parameterized tests verifying each keyword emits the exact corresponding `TokenType` |
| **REQ-14** | Identifiers: letter/underscore start, followed by letters/digits/underscores; digit start followed by letters -> invalid identifier | Mandatory | p. 2 | Scanning identifiers vs checking digit-prefixed invalid identifier rule (`1name`) | Tests verifying valid identifiers and error emission for `1name`, `2_count`, etc. |
| **REQ-15** | Literals: `NUMBER` (int/float), `STRING` (single/double quoted), `BOOLEAN` (`true`/`false`) | Mandatory | p. 2 | Scanner rules for numbers (`10`, `85.5`), quoted strings (`"Ali"`, `'A'`), and booleans (`true`, `false`) | Literal token recognition tests with preserved quotation marks |
| **REQ-16** | Operators: Arithmetic and comparison | Mandatory | p. 3 | Maximal-munch operator scanner (`MATHOP`, `LOGOP`, `ASSIGN`) | Unit tests for binary and unary operators, two-char operators (`==`, `!=`, `<=`, `>=`) |
| **REQ-17** | Delimiters / Punctuation: `(`, `)`, `{`, `}`, `[`, `]`, `;`, `,`, `.` | Mandatory | p. 3 | `DELIMITERS` lookup table in `tokens.py` matching `LPAR`, `RPAR`, `LCUR`, `RCUR`, `LSQU`, `RSQU`, `SEMICOLON`, `COMMA`, `DOT` | Tests for all 9 delimiters individually and in sequence |
| **REQ-18** | Lexical error detection: invalid symbols, invalid identifiers, unclosed strings, unclosed comments | Mandatory | p. 3, 4 | Dedicated error codes `LEX_INVALID_CHARACTER`, `LEX_INVALID_IDENTIFIER`, `LEX_UNCLOSED_STRING`, `LEX_UNCLOSED_COMMENT` | Direct tests asserting exact message format and positions for each error |
| **REQ-19** | Error format: Line number, Column number, Descriptive error message | Mandatory | p. 3, 4 | `Diagnostic` class with `format()` producing `Error: <msg> at line L, column C.` or `Error: <msg> starting at line L, column C.` | Verbatim string assertion against PDF p. 3–4 examples |
| **REQ-20** | Output format matching `Lexeme: <lexeme>, Token: <TOKEN>` | Mandatory | p. 4, 5 | `Token.format()` and CLI `--phase lex` printing token list in PDF format | Golden test comparing CLI output on PDF sample program |
| **REQ-21** | Phase 1 submission deliverables: Source code, input text file, brief 1–2 page report | Mandatory | p. 5 | Repository structure: `minijava/`, `samples/phase1_tests.txt`, `docs/phase1_report.md` | Deliverables presence and completeness audit |
| **REQ-22** | Phase 2 Syntax Analyzer (Parser) hand-written implementation | Mandatory | p. 1 | `minijava/parser.py` implementing recursive-descent parsing for classes, methods, statements, expressions | `test_parser.py` parsing comprehensive language constructs |
| **REQ-23** | Phase 3 Semantic Analyzer implementation | Mandatory | p. 1 | `minijava/semantic.py` implementing symbol scoping, type checking, return/call validation | `test_semantic.py` verifying valid and invalid semantic scenarios |

---

## 3. Evaluation and Deliverables Breakdown

### 3.1 Course Deliverables Matrix

```
CSCI415 Deliverables
├── Phase 01 (Oct 30, 2026)
│   ├── Source Code: Lexer, Symbol Table, Shared Modules, CLI Driver, Tests
│   ├── Test Programs: samples/phase1_tests.txt (valid and invalid Mini Java programs)
│   └── Report: docs/phase1_report.md (1–2 pages: tokens, error handling, challenges)
├── Phase 02 & Phase 03 (Dec 20, 2026)
│   ├── Source Code: Parser, AST nodes, AST Printer, Semantic Analyzer, Scoped Symbol Table
│   ├── Test Programs: samples/phase23_tests.txt (syntax and semantic test programs)
│   └── Report: docs/phase23_report.md (grammar design, semantic rules, challenges)
└── Oral Examination (Post-Submission)
    └── Individual interview with every team member testing code comprehension
```

### 3.2 Evaluation Criteria Mapping
1. **Correctness of Lexical Analysis (Phase 1):**
   - Correct token categorization and lexeme extraction across all keywords, identifiers, literals, operators, and delimiters.
   - Exact compliance with PDF sample output format.
   - Accurate line and column positioning for all tokens and diagnostics.
   - Comprehensive error recovery (scanning continues after errors to report multiple defects).
2. **Grammar Conformance & AST Construction (Phase 2):**
   - Correct handling of operator precedence and left-associativity.
   - Elimination of grammar ambiguities (e.g., dangling-else disambiguation).
   - Clean, immutable AST generation with faithful source coordinate tracking.
   - Informative syntax error diagnostics identifying expected vs found tokens.
3. **Semantic Integrity & Type Safety (Phase 3):**
   - Correct scope hierarchy management (global classes, class members, methods, local blocks).
   - Enforcement of variable shadowing and duplicate declaration constraints.
   - Statically sound type checking for arithmetic, relational, boolean, and string operations.
   - Proper validation of method invocations (arity, parameter types) and return statements.
4. **Code Quality and Architecture:**
   - Strict separation of compiler phases with unidirectional dependencies.
   - Zero third-party dependencies; 100% standard library compliance.
   - Absence of code duplication; shared abstractions for tokens, diagnostics, and AST nodes.
   - Robust automated unit test suite and golden test harness.
5. **Team Participation:**
   - Balanced contribution across all five members.
   - Clean modular ownership boundaries.
   - Every member prepared to defend their modules during oral examination.

---

## 4. Ambiguities, Risks, and Provisional Decisions

Where the 5-page PDF is concise or silent on specific technical details, the team has established **safest specification-compliant provisional decisions** (labeled `PROVISIONAL`). These decisions provide unambiguous contracts for the AI coding assistants while remaining completely faithful to standard compiler design practices and the Mini Java specification.

| Item | Topic / Issue | PDF Source | Impact / Risk | Provisional Technical Decision | Requires Instructor Clarification? |
|---|---|---|---|---|---|
| **P-01** | Operator Token Grouping | p. 3 mentions *"Arithmetic and comparison operators"* without listing token names; p. 4–5 sample shows `+` as `MATHOP`, `>=` as `LOGOP`, `=` as `ASSIGN` | Lexer assistant might invent fine-grained tokens (e.g. `PLUS`, `MINUS`, `GE`) breaking sample output compatibility | Group operators into three broad token types: `MATHOP` (`+`, `-`, `*`, `/`, `%`), `LOGOP` (`==`, `!=`, `<`, `>`, `<=`, `>=`, `&&`, `||`, `!`), and `ASSIGN` (`=`). | Recommended for confirmation, but strictly compliant with PDF sample output. |
| **P-02** | Keyword status of `main` | p. 2 keyword list omits `main`; p. 4 sample output lists `Lexeme: main, Token: MAIN` | Scanner might classify `main` as `IDENTIFIER` instead of `MAIN` | Treat `main` as a reserved keyword with token type `MAIN`. | Unambiguously resolved by PDF p. 4 sample output. |
| **P-03** | Token name for keyword `boolean` | p. 2 lists `boolean` in keywords, but `BOOLEAN` is designated for literal `true`/`false` | Token name collision between type keyword and boolean literal | Assign token `BOOLEAN_TYPE` to keyword `boolean`, matching the pattern established by `String` -> `STRING_TYPE`. | Safest non-conflicting mapping. |
| **P-04** | Concrete Syntax Grammar | PDF outlines Mini Java constructs in samples and descriptions but does not provide an EBNF grammar | Parser assistant might choose an incompatible grammar dialect | Formulate an authoritative EBNF grammar (contract §6.3) supporting classes, fields, methods, if/else, while, for, blocks, return, expressions, and 1D arrays. | Team contract frozen; grammar covers all PDF features. |
| **P-05** | Error Recovery Strategy | PDF specifies error reporting format but does not require multi-error syntax error recovery | Recursive-descent error recovery (panic mode) can cause cascading false errors and architectural bloat | Lexer reports all errors across the file. Parser halts at the first syntax error. Semantic analyzer reports all semantic errors found across validly parsed ASTs. | Safest university compiler approach. |
| **P-06** | System Printing Built-in | Typical Java programs use `System.out.println()` / `System.out.print()` | Parser and Semantic Analyzer might fail on unresolved `System` symbol | Support `System.out.println` and `System.out.print` as built-in member accesses in semantic analysis when `System` is not explicitly declared as a user class. | Recommended convenience for testing standard Java snippets. |
| **P-07** | Array Dimensions | p. 3 lists delimiters `[` and `]` (`LSQU`, `RSQU`) | Multi-dimensional arrays add complexity not specified in PDF | Support single-dimensional arrays (`type[] identifier`, array indexing `a[i]`, and array initializers `{e1, e2}`). | Aligns with standard Mini Java pedagogical benchmarks. |
| **P-08** | Exit Codes and CLI Output | PDF does not mandate UNIX exit codes | Automated test harnesses require deterministic exit status | `0` = successful run with 0 errors; `1` = compilation errors reported; `2` = CLI usage or unreadable file error. Compiler report goes to stdout; fatal CLI/IO errors go to stderr. | Standard UNIX CLI convention. |

---

## 5. Verification and Acceptance Strategy
Every requirement in the matrix is linked to a concrete verification mechanism:
1. **Contract Conformance Suite (`tests/test_contract.py`):** Uses reflection (`inspect`) to assert that all module interfaces, function signatures, dataclass fields, and enum values match the contract exactly.
2. **Unit Test Suites (`tests/test_*.py`):** Focused tests for each component targeting nominal cases, edge cases, and invalid inputs.
3. **Golden Test Suites (`tests/golden/`):** Pairings of Mini Java source files (`.mj`) and expected stdout (`.out`), authored independently of the component owners to prevent confirmation bias.
4. **PDF Acceptance Suite (`tests/test_pdf_examples.py`):** Exact replicas of the code snippets from PDF pages 3–5 tested against the compiler.

# Mini Java Compiler: Phase 1 (Lexical Analysis) Requirements Analysis

**Course:** CSCI415 – Compiler Design (Fall 2026)  
**Project Title:** Mini Compiler for a Java-like Language (Mini Java) — Phase 1: Lexical Analyzer  
**Authoritative Specification:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)  
**Project Workspace:** `/home/yousefmahmoudali/Desktop/CompilerProject/`  
**Phase 1 Working Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/Phase 1/`  
**Target Deadline:** **October 30, 2026**  

---

## 1. Phase 1 Project Overview

### 1.1 Objective and Scope
In Phase 1, the team must design, implement, test, and document a complete **Lexical Analyzer (Scanner)** for the Mini Java programming language (*PDF p. 1–2*).

The lexical analyzer reads Mini Java source code character by character, skips whitespace and comments, classifies characters into tokens, tracks 1-based source positions (`line` and `column`), constructs an identifier symbol table, and detects/reports lexical errors.

> [!IMPORTANT]
> **Forward Compatibility Requirement (*PDF p. 1*):**  
> *"Take into consideration that all phases must be able to build on each other to avoid issues."*  
> Phase 1 must produce a clean, immutable token stream (`list[Token]`) ending in an `EOF` token that can be consumed directly by the Phase 2 Parser (Nov 21 – Dec 20) without requiring any changes to the Phase 1 scanner.

---

### 1.2 Phase 1 Timeline and Milestones
- **Phase 01 Start:** October 12, 2026 (*PDF p. 1*)
- **Phase 01 Submission Deadline:** **October 30, 2026** (*PDF p. 1*)
- **Subsequent Phases:** Phase 02 (Syntax Analysis) and Phase 03 (Semantic Analysis) start November 21, 2026, and conclude December 20, 2026.

---

### 1.3 Technical Constraints and Permitted Tools
1. **Implementation Language:** Python ≥ 3.10 using **standard library only** (*PDF p. 1*).
2. **Prohibited Tools & Direct Shortcuts (*PDF p. 1*):**
   > *"DO NOT use ready-made lexical analysis/parser libraries or ready functions that directly solve the required compiler tasks; build your own methods."*
   - **Prohibited:** Lex/Flex, ANTLR, PLY, Lark, SLY, PyParsing.
   - **Prohibited:** Python's standard library shortcut modules: `re` (regular expressions), `tokenize`, `ast`, `shlex`. Tokenization must NOT be implemented using `re.finditer()` or naive `str.split()`.
   - **Mandatory:** Pure hand-crafted sequential character scanning using index cursors, lookahead helpers, and character classification.
3. **Oral Examination & Anti-Plagiarism (*PDF p. 1–2*):**
   - *"A discussion will be held with ALL team members; all members must participate in implementation."*
   - *"ANY PLAGIARISM DETECTED WILL LEAD TO A ZERO AND NO DISCUSSION WILL BE CONDUCTED FOR THE TEAM."*
   - Every one of the 5 team members must write and own code in Phase 1 so they can confidently explain it during the oral grading interview.

---

### 1.4 Mini Java Lexical Specifications (*PDF p. 2–5*)

#### Case Sensitivity
Mini Java is strictly case-sensitive (*PDF p. 2*). Keywords like `class` must not match identifiers like `Class`.

#### Reserved Keywords
The language specifies 15 reserved keywords (*PDF p. 2, 4*):
`class`, `public`, `static`, `void`, `int`, `float`, `boolean`, `char`, `String`, `if`, `else`, `while`, `for`, `return`, `main`.
- **Keyword `String`:** Classified as `STRING_TYPE` (a type keyword), while quoted literal values are classified as `STRING` (*PDF p. 2*).
- **Keyword `boolean`:** Classified as `BOOLEAN_TYPE` (to avoid collision with the `BOOLEAN` literal token for `true`/`false`).
- **Keyword `main`:** Appears in the sample output (*PDF p. 4*) as `Lexeme: main, Token: MAIN`.
- **Booleans:** `true` and `false` are literals with token `BOOLEAN`.

#### Identifiers
- Must start with an ASCII letter (`[A-Za-z]`) or underscore (`_`).
- Can be followed by letters, digits, and underscores.
- **Invalid Identifier Rule (*PDF p. 2, 3*):** If a sequence starts with a digit and is immediately followed by letters or underscores (e.g. `1name`, `2_count`), it must be detected and reported as an invalid identifier error rather than split into a number followed by an identifier.

#### Literals
- `NUMBER`: Any integer or floating-point numeric value (e.g. `10`, `0`, `25.5`).
- `STRING`: Any value enclosed in single or double quotes (e.g. `"Hello"`, `'A'`). Preserves enclosing quotes in lexeme.
- `BOOLEAN`: Literals `true` and `false`.

#### Operators
- Arithmetic (`MATHOP`): `+`, `-`, `*`, `/`, `%`
- Comparison / Relational / Logical (`LOGOP`): `==`, `!=`, `<`, `>`, `<=`, `>=`, `&&`, `||`, `!`
- Assignment (`ASSIGN`): `=`
- *Maximal Munch:* Two-character operators (`==`, `!=`, `<=`, `>=`, `&&`, `||`) must be matched before single-character operators.

#### Delimiters / Punctuation
- `(` -> `LPAR`
- `)` -> `RPAR`
- `{` -> `LCUR`
- `}` -> `RCUR`
- `[` -> `LSQU`
- `]` -> `RSQU`
- `;` -> `SEMICOLON`
- `,` -> `COMMA`
- `.` -> `DOT`

#### Comments
- Single-line: `//` skips to end of line.
- Multi-line: `/*` skips to `*/`.
- Comments do not produce tokens.

---

### 1.5 Required Error Handling and Reporting Format (*PDF p. 3–4*)
Every lexical error must include:
1. 1-based Line number
2. 1-based Column number
3. A descriptive error message

**The Four Required Lexical Error Formats:**
1. **Invalid Character:**
   - *Example:* `int x = 10 @ 5;`
   - *Output:* `Error: invalid character '@' at line 1, column 12.`
2. **Invalid Identifier:**
   - *Example:* `1name = 5;`
   - *Output:* `Error: invalid identifier '1name' at line 1, column 1.`
3. **Unclosed Quoted String:**
   - *Example:* `String name = "Ali;`
   - *Output:* `Error: unclosed string literal starting at line 1, column 15.`
4. **Unclosed Comment:**
   - *Example:* `/* This comment does not end`
   - *Output:* `Error: unclosed comment starting at line 1, column 1.`

---

## 2. Phase 1 Requirements Traceability Matrix

| ID | Requirement Statement | Classification | Source Page | Phase 1 Implementation Module | Verification Method |
|---|---|---|---|---|---|
| **REQ-01** | Lexical analysis on Mini Java source programs | Mandatory | p. 1 | `minijava/lexer.py` | CLI execution with `--phase lex` |
| **REQ-02** | Phase 1 must build foundation for Phases 2 & 3 | Mandatory | p. 1 | `minijava/tokens.py`, `minijava/symbol_table.py` | Contract reflection tests (`test_contract.py`) |
| **REQ-03** | Use any language preferred | Mandatory | p. 1 | Python ≥ 3.10 (stdlib only) | `python3 -m compileall` byte-compilation |
| **REQ-04** | All 5 members must participate in implementation | Mandatory | p. 1 | 5 distinct module ownerships (M1 to M5) | Git commit history and oral walkthrough rehearsal |
| **REQ-05** | No ready-made scanner libraries or shortcut helpers | Mandatory | p. 1 | Pure character-by-character scanner; zero regex (`re`) | Automated AST/import inspection script |
| **REQ-06** | Zero plagiarism tolerance | Mandatory | p. 2 | Clean-room modular implementation | Code audit against public repos |
| **REQ-07** | Read character by character, group into tokens, classify | Mandatory | p. 2 | `Lexer.tokenize()` sequential cursor | Unit tests covering every token type |
| **REQ-08** | Ignore whitespace and comments (`//`, `/* ... */`) | Mandatory | p. 2, 3 | `minijava/lexer.py` comment/whitespace skipper | Unit tests for single and multi-line comments |
| **REQ-09** | Detect illegal characters and report exact line & col | Mandatory | p. 2, 3 | `minijava/lexer.py` invalid symbol detector | Regression test on `int x = 10 @ 5;` |
| **REQ-10** | Case-sensitive language (`class` != `Class`) | Mandatory | p. 2 | `minijava/scanner_ident_num.py` exact lookup | Unit test comparing `class` and `Class` |
| **REQ-11** | Generate token stream ending with `EOF` | Mandatory | p. 2 | `LexResult.tokens` | Assertion that last token is `TokenType.EOF` |
| **REQ-12** | Construct and maintain symbol table for identifiers | Mandatory | p. 2 | `minijava/symbol_table.py` (`IdentifierTable`) | `test_symbol_table.py` checking occurrence tracking |
| **REQ-13** | 15 Keywords; `String` -> `STRING_TYPE`, `main` -> `MAIN` | Mandatory | p. 2, 4 | `minijava/tokens.py` (`KEYWORDS`) | Verification against PDF p. 4 sample output |
| **REQ-14** | Identifiers: letter/`_` start; digit-prefix -> invalid id | Mandatory | p. 2, 3 | `minijava/scanner_ident_num.py` | Regression test on `1name = 5;` |
| **REQ-15** | Literals: `NUMBER` (int/float), `STRING`, `BOOLEAN` | Mandatory | p. 2 | `scanner_ident_num.py`, `scanner_str_op.py` | Literal scanning unit tests |
| **REQ-16** | Operators: `MATHOP`, `LOGOP`, `ASSIGN` (Maximal Munch) | Mandatory | p. 3, 4, 5 | `minijava/scanner_str_op.py` | Two-char vs one-char operator tests |
| **REQ-17** | Delimiters: `(`, `)`, `{`, `}`, `[`, `]`, `;`, `,`, `.` | Mandatory | p. 3 | `minijava/lexer.py` delimiter table | All 9 delimiter unit tests |
| **REQ-18** | Detect 4 lexical errors with line, column, description | Mandatory | p. 3, 4 | `minijava/diagnostics.py` templates | Exact string comparison against PDF p. 3–4 |
| **REQ-19** | Output format matching `Lexeme: <lex>, Token: <TOK>` | Mandatory | p. 4, 5 | `Token.format()` and `driver.py` | Golden test matching PDF sample output verbatim |
| **REQ-20** | Phase 1 Deliverable 1: Source code | Mandatory | p. 5 | `minijava/` package | Clean build and test execution |
| **REQ-21** | Phase 1 Deliverable 2: Input test programs file | Mandatory | p. 5 | `samples/phase1_tests.txt` | File presence and test case diversity check |
| **REQ-22** | Phase 1 Deliverable 3: 1–2 page technical report | Mandatory | p. 5 | `docs/phase1_report.md` | Report covers tokens, error handling, challenges |

---

## 3. Phase 1 Submission Deliverables Breakdown (*PDF p. 5*)

1. **Source Code:**
   Complete Python implementation in `minijava/` and test suites in `tests/`.
2. **Input Text File (`samples/phase1_tests.txt`):**
   Contains multiple diverse Mini Java test programs demonstrating:
   - Valid classes, methods, variable declarations, loops, and expressions.
   - All token categories (keywords, literals, operators, delimiters).
   - Comments (single-line, multi-line, nested delimiters within comments).
   - Lexical error cases (invalid characters, invalid identifiers, unclosed strings, unclosed comments).
3. **Brief Report (`docs/phase1_report.md`, 1–2 pages):**
   - **Section 1: Tokens Implemented:** Overview of keywords, literals, operators (`MATHOP`, `LOGOP`, `ASSIGN`), delimiters, and `EOF`.
   - **Section 2: Error Handling:** How 1-based coordinates are tracked, error message structure, and how scanning continues after errors.
   - **Section 3: Challenges Faced & Solutions:** Maximal munch for two-character operators, distinguishing floating-point numbers from dot delimiters, digit-prefixed invalid identifier detection, and unclosed comment tracking.

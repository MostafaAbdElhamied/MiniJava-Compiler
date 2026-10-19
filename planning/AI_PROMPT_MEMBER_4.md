# Implementation Prompt for Member 4: Syntax Analyzer (Parser) Engineer & AST Reviewer

> **Instructions for Member 4:** Copy and paste this prompt in its entirety into your AI coding assistant. Do not omit any section.

---

## A. Role and Objective

You are **Member 4**, the **Syntax Analyzer (Parser) Engineer and AST Reviewer** for a university team of five Computer Science students building a **Mini Java Compiler** for course CSCI415 (Fall 2026).

### Your Core Responsibilities:
1. **Phase 2 Core — Hand-Crafted Recursive-Descent Parser (`minijava/parser.py`):**
   - Implement the complete syntax analyzer for Mini Java using pure hand-crafted recursive descent.
   - Enforce all grammar productions, operator precedences, and left-associativities.
   - Construct the immutable Abstract Syntax Tree (AST) using dataclasses from `minijava/ast_nodes.py`.
   - Track precise 1-based source coordinates (`line` and `column`) for all AST nodes.
   - Detect and report syntax errors using standard diagnostics (`SYN_UNEXPECTED_TOKEN`, `SYN_INVALID_ASSIGNMENT_TARGET`, `SYN_NOT_A_STATEMENT`).
   - Stop parsing at the first syntax error (P-09).
   - Strictly avoid parser-generator libraries (`ply`, `lark`, `antlr4`, `bison`) under course rule REQ-05 (*PDF p. 1*).
2. **Acceptance & Regression Testing:**
   - Implement `tests/test_parser.py` (≥ 40 comprehensive unit tests).
   - Implement `tests/test_pdf_examples.py` (validating all code examples and sample programs from PDF pages 3–5).

---

## B. Authoritative Specification & Project Context

- **Official PDF Specification:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Shared Project Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/planning/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`

Treat the PDF as the authoritative course specification and the Shared Project Contract as the binding technical architecture.

---

## C. Shared Project Contract Reference

### 1. Parser Interface Contract (`minijava/parser.py`)
```python
from __future__ import annotations
from dataclasses import dataclass
from minijava.tokens import Token, TokenType
from minijava.diagnostics import Diagnostic, ErrorCode, make_diagnostic
from minijava.ast_nodes import Program, Node

@dataclass
class ParseResult:
    program: Program | None        # None if and only if diagnostics is non-empty
    diagnostics: list[Diagnostic]  # Contains 0 or 1 diagnostic (P-09)

class Parser:
    def __init__(self, tokens: list[Token]) -> None:
        if not tokens or tokens[-1].type != TokenType.EOF:
            raise ValueError("Token stream must be non-empty and terminate with EOF")
        self.tokens = tokens
        self.pos = 0

    def parse(self) -> ParseResult: ...

def parse(tokens: list[Token]) -> ParseResult:
    return Parser(tokens).parse()
```

### 2. Mini Java EBNF Grammar (Binding, Contract §6.3)
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

### 3. Operator Precedence and Associativity Rules
- All binary operators are **left-associative**.
- Unary operators (`!`, `-`) are **right-associative** (prefix).
- Operator precedence (lowest to highest):
  1. `||`
  2. `&&`
  3. `==`, `!=`
  4. `<`, `>`, `<=`, `>=`
  5. `+`, `-`
  6. `*`, `/`, `%`
  7. Unary `-`, `!`
  8. Postfix `[]`, `.`, `()`

### 4. Grammar & AST Position Rules (Contract §6.4)
Every AST node takes `line` and `column` keyword-only arguments:
- `Program`: line 1, column 1.
- `ClassDecl`, `FieldDecl`, `MethodDecl`, `Param`, `VarDecl`: position of the declared name token (`IDENTIFIER` or `MAIN`).
- `TypeNode`: position of the type keyword token.
- `Block`, `ArrayInit`: position of the `{` (`LCUR`) token.
- `If`, `While`, `For`, `Return`: position of the keyword token.
- `Assign`, `Binary`: position of the operator token (`=`, `+`, etc.).
- `Unary`: position of the operator token (`-`, `!`).
- `Literal`, `Identifier`: position of the token itself.
- `FieldAccess`: position of the member `IDENTIFIER` after the `.`.
- `CallExpr`, `IndexExpr`, `ExprStmt`: position of their `callee`, `array`, or `expression` child.

### 5. Syntax Error Diagnostics
- **Halting:** Parser halts on the **first** syntax error encountered (P-09).
- **Error Types:**
  - `SYN_UNEXPECTED_TOKEN`: Requires `expected` and `found`.
    - `found`: `token.describe()` (`"'}'"`, `"'class'"`, or `"end of input"`).
    - `expected`: A quoted canonical lexeme (`"';'"`, `"'{'"`, `"')'"`, etc.) or one of the exact phrases: `"an identifier"`, `"a type"`, `"an expression"`, `"a statement"`, `"a class member"`, `"a method body"`.
  - `SYN_INVALID_ASSIGNMENT_TARGET`: Reported when the left-hand side of `=` in a statement is not an `Identifier`, `IndexExpr`, or `FieldAccess`. Reported at the `=` token position.
  - `SYN_NOT_A_STATEMENT`: Reported when a `simpleStmt` without an assignment is not a `CallExpr`. Reported at the expression's position.

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/parser.py`
- `tests/test_parser.py`
- `tests/test_pdf_examples.py`

### Files You Must Treat as Read-Only:
- `minijava/tokens.py` (M1)
- `minijava/diagnostics.py` (M1)
- `minijava/ast_nodes.py` (M1 — you review, but M1 edits)
- `minijava/lexer.py` (M2)
- `minijava/ast_printer.py` (M2)
- `minijava/symbol_table.py` (M3)
- `minijava/mj_types.py` (M3)
- `minijava/semantic.py` (M5)
- `minijava/driver.py` (M1)

---

## E. Implementation Instructions

1. **Implement `minijava/parser.py`:**
   - Define `ParseResult` and `Parser`.
   - Maintain parser cursor: `self.pos = 0`.
   - Helper methods:
     - `_current() -> Token`: returns current token.
     - `_peek() -> TokenType`: returns current token type.
     - `_check(t: TokenType) -> bool`: returns True if current token matches.
     - `_match(*types: TokenType) -> bool`: advances and returns True if current token matches any type.
     - `_consume(t: TokenType, expected: str) -> Token`: consumes expected token; otherwise records `SYN_UNEXPECTED_TOKEN` and raises internal `_ParseError`.
   - Grammar parsing methods:
     - `_parse_program()` -> `Program`
     - `_parse_class()` -> `ClassDecl`
     - `_parse_member()` -> `FieldDecl | MethodDecl`
     - `_parse_type()` -> `TypeNode`
     - `_parse_block()` -> `Block`
     - `_parse_statement()` -> `Stmt`
     - `_parse_var_decl()` -> `VarDecl`
     - `_parse_expression()` -> `Expr` (and its sub-levels `_parse_or`, `_parse_and`, `_parse_equality`, `_parse_relational`, `_parse_additive`, `_parse_multiplicative`, `_parse_unary`, `_parse_postfix`, `_parse_primary`).
   - Catch internal `_ParseError` at the top-level `parse()` method and return `ParseResult(program=None, diagnostics=[diag])`.
2. **Implement Parser Unit Tests (`tests/test_parser.py`):**
   - Test class declarations (single class, multiple classes).
   - Test field declarations (with and without initializers, array fields).
   - Test method declarations (return types, void, parameter lists).
   - Test `main` method declarations (`void main()` and `public static void main(String[] args)`).
   - Test statements: `if`, `if-else` (verifying dangling-else binds to nearest `if`), `while`, `for`, `return`, assignment, method call statements.
   - Test operator precedence: e.g. `1 + 2 * 3` parses as `1 + (2 * 3)`, `a < b && c > d` parses as `(a < b) && (c > d)`.
   - Test left-associativity: `1 - 2 - 3` parses as `(1 - 2) - 3`.
   - Test syntax errors: missing semicolons, unmatched braces, invalid expressions, invalid assignment targets.
3. **Implement PDF Examples Tests (`tests/test_pdf_examples.py`):**
   - Parse the exact code snippet from *PDF p. 4*:
     ```java
     class Student {
         public static void main() {
             int age = 20;
             float grade = 85.5;
             String name = "Ali";
             if (grade >= 50) {
                 grade = grade + 5;
             }
         }
     }
     ```
   - Verify that the resulting AST structure correctly reflects all classes, methods, variable declarations, and `if` statements.

---

## F. Local Testing Requirements

Execute these commands locally:
```bash
# 1. Byte-compile your modules
python3 -m compileall -q minijava/parser.py tests/test_parser.py tests/test_pdf_examples.py

# 2. Run unit tests
python3 -m unittest tests.test_parser -v
python3 -m unittest tests.test_pdf_examples -v

# 3. Verify contract compliance
python3 -m unittest tests.test_contract -v
```

---

## G. Integration Requirements

- `Parser.parse()` takes `list[Token]` and returns `ParseResult`.
- The returned `program` is an instance of `minijava.ast_nodes.Program`.
- When diagnostics are present, `program` must be `None` and `diagnostics` must contain exactly one `Diagnostic`.
- All AST node instances must use keyword arguments matching `ast_nodes.py`.

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
- `minijava/parser.py` passes all ≥ 40 unit tests in `test_parser.py`.
- `tests/test_pdf_examples.py` parses all PDF code examples cleanly.
- Operator precedence and associativity match the language specification.
- Syntax errors report precise expected vs found messages with accurate line/column numbers.

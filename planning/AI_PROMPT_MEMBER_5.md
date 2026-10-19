# Implementation Prompt for Member 5: Semantic Analyzer Engineer & Test Oracle Lead

> **Instructions for Member 5:** Copy and paste this prompt in its entirety into your AI coding assistant. Do not omit any section.

---

## A. Role and Objective

You are **Member 5**, the **Semantic Analyzer (Type & Scope Checker) Engineer and Independent Test Oracle Lead** for a university team of five Computer Science students building a **Mini Java Compiler** for course CSCI415 (Fall 2026).

### Your Core Responsibilities:
1. **Phase 3 Core — Static Semantic Analyzer (`minijava/semantic.py`):**
   - Implement the multi-pass semantic analyzer and type checker for Mini Java.
   - Enforce scope hierarchy rules (global, class, method, block scopes) using `ScopedSymbolTable`.
   - Prevent duplicate declarations of classes, members, parameters, and local variables.
   - Perform static name resolution for identifiers, method calls, and member accesses.
   - Enforce type consistency across assignments, binary/unary expressions, method arguments, and returns using `minijava/mj_types.py`.
   - Support built-in Mini Java features (`System.out.println`, `System.out.print`, and array `x.length`).
   - Report comprehensive semantic diagnostics with exact coordinates and suppress cascading errors.
   - Never mutate the input AST (`Program`).
2. **Independent Test Oracle Lead:**
   - Author independent golden test cases for the Phase 1 Lexer (`tests/golden/lex/*.mj` and `*.out`).
   - Author independent golden test cases for the Phase 3 Semantic Analyzer (`tests/golden/semantic/*.mj` and `*.out`).
3. **Automated Testing:**
   - Implement `tests/test_semantic.py` (≥ 45 comprehensive unit tests).

---

## B. Authoritative Specification & Project Context

- **Official PDF Specification:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Shared Project Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/planning/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`

Treat the PDF as the authoritative course specification and the Shared Project Contract as the binding technical architecture.

---

## C. Shared Project Contract Reference

### 1. Semantic Analyzer Interface (`minijava/semantic.py`)
```python
from __future__ import annotations
from dataclasses import dataclass
from minijava.ast_nodes import Program, Node
from minijava.diagnostics import Diagnostic, ErrorCode, make_diagnostic
from minijava.symbol_table import ScopedSymbolTable, Scope, Symbol, SymbolKind, ScopeKind
from minijava.mj_types import (
    MJType, INT, FLOAT, BOOLEAN, CHAR, STRING, VOID, ERROR,
    is_numeric, is_assignable, binary_result_type, unary_result_type,
    from_type_node, literal_type, class_type, is_class_type, element_type
)

@dataclass
class SemanticResult:
    diagnostics: list[Diagnostic]      # Semantic errors in deterministic order
    symbol_table: ScopedSymbolTable

class SemanticAnalyzer:
    def __init__(self) -> None:
        self.table = ScopedSymbolTable()
        self.diagnostics: list[Diagnostic] = []
        self.current_method: Symbol | None = None

    def analyze(self, program: Program) -> SemanticResult: ...

def analyze(program: Program) -> SemanticResult:
    return SemanticAnalyzer().analyze(program)
```

### 2. Multi-Pass Analysis Architecture (Deterministic Order)
1. **Pass 1a — Global Class Registration:**
   - Iterate through `program.classes` in order.
   - For each `ClassDecl`, attempt to define a `Symbol(name, SymbolKind.CLASS, class_type(name), line, column)` in `global_scope`.
   - If already defined, emit `SEM_DUPLICATE_DECLARATION` with `prev_line` and `prev_column`.
   - Create a class scope (`self.table.enter_scope(class_decl.name, ScopeKind.CLASS)`), store it in `symbol.scope`, and exit back to global.
2. **Pass 1b — Member Declarations:**
   - For each class in order, re-enter its class scope via `self.table.reenter_scope(sym.scope)`.
   - Register fields: `Symbol(name, SymbolKind.FIELD, from_type_node(field.var_type), line, column)`. Duplicate -> `SEM_DUPLICATE_DECLARATION`.
   - Register methods: `Symbol(name, SymbolKind.METHOD, from_type_node(method.return_type), line, column, param_types=[...])`. Duplicate -> `SEM_DUPLICATE_DECLARATION`. (Fields and methods share one class namespace).
   - Exit class scope.
3. **Pass 2 — Method Body & Statement Type Checking:**
   - For each class, re-enter its class scope.
   - Check field initializers if present.
   - For each method:
     - Record `self.current_method = method_symbol`.
     - Enter method scope: `self.table.enter_scope(method.name, ScopeKind.METHOD)`.
     - Register parameters as `SymbolKind.PARAMETER`. Duplicate param names -> `SEM_DUPLICATE_DECLARATION`.
     - Analyze body statements directly in the method scope (the top-level `Block` of a method body does not open a secondary scope).
     - Exit method scope.

### 3. Scoping Rules
- **Blocks:** Any nested `Block` statement enters a scope named `f"block@{line}:{column}"` of kind `ScopeKind.BLOCK`.
- **For Loops:** A `For` loop enters a scope named `f"for@{line}:{column}"` of kind `ScopeKind.BLOCK` containing its initialization variable.
- **Local Variables:** `VarDecl` defines `SymbolKind.LOCAL`. Before defining, call `self.table.lookup_in_method(name)`. If a symbol is found, emit `SEM_DUPLICATE_DECLARATION` (redeclaring a variable or parameter inside the same method is prohibited). Note: local variables may shadow fields in enclosing class scopes.

### 4. Expression & Statement Typing Rules
- **Error Suppression:** If a subexpression evaluates to `ERROR`, do not emit further cascading errors for parent expressions.
- **Assignments:** In `Assign(target, value)` or `VarDecl(initializer)`:
  - Check `is_assignable(target_type, value_type)`. If incompatible, emit `SEM_TYPE_MISMATCH` at the assignment node.
  - Assignment targets must be a variable (`LOCAL`, `PARAMETER`, `FIELD`), `IndexExpr`, or `FieldAccess` to a `FIELD`. Otherwise emit `SEM_NOT_A_VARIABLE`.
- **Method Invocations (`CallExpr`):**
  - Callee must resolve to a `METHOD` symbol. If undeclared -> `SEM_UNDECLARED_IDENTIFIER`. If not a method -> `SEM_NOT_A_METHOD`.
  - Validate argument count: if mismatch, emit `SEM_ARGUMENT_COUNT` at the `CallExpr` node.
  - Validate argument types: for each argument, check `is_assignable(param_type, arg_type)`. If mismatch, emit `SEM_ARGUMENT_TYPE` at the argument node.
  - If a method returns `void` and is used as an expression (outside an `ExprStmt`), emit `SEM_VOID_VALUE`.
- **Member Access (`FieldAccess`):**
  - **Built-in `System.out.println(arg)`:** Valid with 0 or 1 non-void argument. Returns `VOID`.
  - **Built-in `System.out.print(arg)`:** Valid with exactly 1 non-void argument. Returns `VOID`.
  - **Array length `x.length`:** If target is array type, returns `INT`.
  - **Class member `C.m`:** If `C` resolves to a `CLASS`, resolve `m` in `C`'s class scope.
- **Array Indexing (`IndexExpr`):**
  - Target must be array type -> `SEM_NOT_AN_ARRAY`.
  - Index must be `INT` -> `SEM_INDEX_NOT_INT`.
- **Return Statements:**
  - In a `void` method, `return <expr>;` -> `SEM_RETURN_VALUE_IN_VOID`.
  - In a non-void method, `return;` without value -> `SEM_MISSING_RETURN_VALUE`.
  - If value type is incompatible with return type -> `SEM_RETURN_TYPE_MISMATCH`.
- **Conditions:**
  - In `If`, `While`, `For`: condition expression must evaluate to `BOOLEAN`. If not, emit `SEM_CONDITION_NOT_BOOLEAN`.

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/semantic.py`
- `tests/test_semantic.py`
- `tests/golden/lex/*.mj` and `tests/golden/lex/*.out`
- `tests/golden/semantic/*.mj` and `tests/golden/semantic/*.out`

### Files You Must Treat as Read-Only:
- `minijava/tokens.py` (M1)
- `minijava/diagnostics.py` (M1)
- `minijava/ast_nodes.py` (M1)
- `minijava/lexer.py` (M2)
- `minijava/ast_printer.py` (M2)
- `minijava/symbol_table.py` (M3)
- `minijava/mj_types.py` (M3)
- `minijava/parser.py` (M4)
- `minijava/driver.py` (M1)

---

## E. Implementation Instructions

1. **Implement `minijava/semantic.py`:**
   - Define `SemanticResult` and `SemanticAnalyzer`.
   - Implement `analyze(program: Program) -> SemanticResult`.
   - Structure passes cleanly: `_pass1a_classes()`, `_pass1b_members()`, `_pass2_bodies()`.
   - Implement expression type checkers:
     - `_check_expr(expr: Expr) -> MJType`
     - `_check_literal(lit: Literal) -> MJType`
     - `_check_identifier(ident: Identifier) -> MJType`
     - `_check_binary(bin: Binary) -> MJType`
     - `_check_unary(un: Unary) -> MJType`
     - `_check_call(call: CallExpr) -> MJType`
     - `_check_index(idx: IndexExpr) -> MJType`
     - `_check_field_access(fa: FieldAccess) -> MJType`
   - Implement statement checkers:
     - `_check_stmt(stmt: Stmt)`
     - `_check_var_decl(decl: VarDecl)`
     - `_check_assign(assign: Assign)`
     - `_check_if(if_stmt: If)`
     - `_check_while(while_stmt: While)`
     - `_check_for(for_stmt: For)`
     - `_check_return(ret: Return)`
2. **Author Independent Golden Tests:**
   - In `tests/golden/lex/`: Create ≥ 12 `.mj` files and hand-derived `.out` files testing comments, escapes, delimiters, invalid characters, invalid identifiers, unclosed strings, and unclosed comments.
   - In `tests/golden/semantic/`: Create ≥ 15 `.mj` files and hand-derived `.out` files testing scope resolution, type mismatch, argument counts, void usage, and duplicate declarations.
3. **Implement Semantic Unit Tests (`tests/test_semantic.py`):**
   - Write tests verifying all `SEM_` error codes.
   - Test class and method duplicate declarations.
   - Test variable shadowing and scope isolation.
   - Test arithmetic and boolean expression typing.
   - Test method call argument type checking and arity checking.
   - Test return statements in void and non-void methods.
   - Test `System.out.println` and `System.out.print` calls.

---

## F. Local Testing Requirements

Execute these commands locally:
```bash
# 1. Byte-compile your modules
python3 -m compileall -q minijava/semantic.py tests/test_semantic.py

# 2. Run unit tests
python3 -m unittest tests.test_semantic -v

# 3. Verify contract compliance
python3 -m unittest tests.test_contract -v
```

---

## G. Integration Requirements

- `SemanticAnalyzer.analyze(program)` accepts `Program` and returns `SemanticResult`.
- All diagnostics must use `make_diagnostic` with exact `ErrorCode.SEM_*` codes.
- Symbol table operations must strictly use `ScopedSymbolTable` from `minijava.symbol_table`.
- Type checking must strictly use `minijava.mj_types`.

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
- `minijava/semantic.py` passes all ≥ 45 unit tests in `test_semantic.py`.
- All semantic error codes (`SEM_*`) are properly tested and verified.
- Independent golden test cases are stored in `tests/golden/lex/` (≥ 12) and `tests/golden/semantic/` (≥ 15).
- Cascading errors are properly suppressed when subexpressions fail.

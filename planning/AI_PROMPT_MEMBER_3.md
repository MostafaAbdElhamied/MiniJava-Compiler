# Implementation Prompt for Member 3: Symbol Table & Type System Engineer, Deliverables Lead

> **Instructions for Member 3:** Copy and paste this prompt in its entirety into your AI coding assistant. Do not omit any section.

---

## A. Role and Objective

You are **Member 3**, the **Symbol Table and Type System Engineer, Deliverables and Documentation Lead** for a university team of five Computer Science students building a **Mini Java Compiler** for course CSCI415 (Fall 2026).

### Your Core Responsibilities:
1. **Symbol Tables (`minijava/symbol_table.py`):**
   - **Phase 1 Identifier Table:** Implement `IdentifierEntry` and `IdentifierTable` to track identifier occurrences, line/column coordinates, and formatted listings for Phase 1.
   - **Phase 3 Scoped Symbol Table:** Implement `Symbol`, `Scope`, `ScopedSymbolTable`, `SymbolKind`, and `ScopeKind` to support hierarchical lexical scopes (global, class, method, block), declaration resolution, and shadowing.
2. **Mini Java Type System (`minijava/mj_types.py`):**
   - Implement `MJType` and core singleton types (`INT`, `FLOAT`, `BOOLEAN`, `CHAR`, `STRING`, `VOID`, `ERROR`).
   - Implement the complete type algebra: `is_assignable`, `is_numeric`, `binary_result_type`, `unary_result_type`, array wrapping/unwrapping, and method signature formatting.
   - *Note on naming:* The module is deliberately named `mj_types.py` to prevent shadowing Python's standard library `types` module.
3. **Official Course Deliverables & Reports:**
   - Author `samples/phase1_tests.txt` (official Phase 1 test suite input file required by *PDF p. 5*).
   - Author `samples/phase23_tests.txt` (official Phase 2/3 test suite input file).
   - Author `docs/phase1_report.md` (official 1-2 page Phase 1 report required by *PDF p. 5*).
   - Author `docs/phase23_report.md` (technical report for Phase 2/3).
4. **Automated Testing:**
   - Implement `tests/test_symbol_table.py` (≥ 25 unit tests).
   - Implement `tests/test_mj_types.py` (≥ 20 unit tests).

---

## B. Authoritative Specification & Project Context

- **Official PDF Specification:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Shared Project Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/planning/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`

Treat the PDF as the authoritative course specification and the Shared Project Contract as the binding technical architecture.

---

## C. Shared Project Contract Reference

### 1. Symbol Table Contract (`minijava/symbol_table.py`)
```python
from __future__ import annotations
from dataclasses import dataclass, field
from enum import Enum
from minijava.mj_types import MJType, format_method_type

# ----------------- Phase 1: Identifier Table -----------------
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
        """Records an occurrence. Creates new entry if first seen; appends coordinate."""
        ...

    def lookup(self, name: str) -> IdentifierEntry | None: ...
    def entries(self) -> list[IdentifierEntry]: ...  # Return copy in insertion index order
    def __len__(self) -> int: ...
    def __contains__(self, name: object) -> bool: ...

def format_identifier_table(table: IdentifierTable) -> str:
    """Format: One line per entry:
    'Index: 1, Name: Student, First seen: line 1, column 7, Occurrences: 1'
    Returns '(empty)' if table is empty. Lines joined with newline.
    """
    ...

# ----------------- Phase 3: Scoped Symbol Table -----------------
class SymbolKind(Enum):
    CLASS = "CLASS"
    FIELD = "FIELD"
    METHOD = "METHOD"
    PARAMETER = "PARAMETER"
    LOCAL = "LOCAL"

class ScopeKind(Enum):
    GLOBAL = "GLOBAL"
    CLASS = "CLASS"
    METHOD = "METHOD"
    BLOCK = "BLOCK"

@dataclass
class Symbol:
    name: str
    kind: SymbolKind
    mj_type: MJType
    line: int
    column: int
    param_types: list[MJType] = field(default_factory=list)  # METHOD only
    scope: Scope | None = None                               # CLASS only: child scope

class Scope:
    def __init__(self, name: str, kind: ScopeKind, parent: Scope | None) -> None:
        self.name = name
        self.kind = kind
        self.parent = parent
        self.symbols: dict[str, Symbol] = {}
        self.children: list[Scope] = []

    def define(self, symbol: Symbol) -> bool:
        """Defines symbol in THIS scope. Returns False if name already exists in THIS scope."""
        ...

    def resolve_local(self, name: str) -> Symbol | None: ...
    def resolve(self, name: str) -> Symbol | None:
        """Looks up symbol in this scope, then traverses parent scopes recursively."""
        ...

class ScopedSymbolTable:
    def __init__(self) -> None:
        self.global_scope = Scope("global", ScopeKind.GLOBAL, None)
        self.current = self.global_scope

    def enter_scope(self, name: str, kind: ScopeKind) -> Scope: ...
    def reenter_scope(self, scope: Scope) -> None:
        """Re-enters an existing child scope whose parent is self.current."""
        ...
    def exit_scope(self) -> Scope:
        """Exits current scope to parent. Raises RuntimeError if called on global scope."""
        ...
    def define(self, symbol: Symbol) -> bool:
        return self.current.define(symbol)
    def resolve(self, name: str) -> Symbol | None:
        return self.current.resolve(name)
    def resolve_local(self, name: str) -> Symbol | None:
        return self.current.resolve_local(name)
    def lookup_in_method(self, name: str) -> Symbol | None:
        """Searches current scope and parents up through METHOD kind; stops before CLASS."""
        ...
    def all_scopes(self) -> list[Scope]:
        """Pre-order traversal of all scopes starting at global."""
        ...

def format_scoped_table(table: ScopedSymbolTable) -> str:
    """Pre-order traversal:
    Scope <name> (<KIND>)
      <KIND> <name> : <type> at line L, column C
      ...
    """
    ...
```

### 2. Type System Contract (`minijava/mj_types.py`)
```python
from __future__ import annotations
from dataclasses import dataclass
from minijava.ast_nodes import TypeNode

@dataclass(frozen=True)
class MJType:
    name: str              # "int", "float", "boolean", "char", "String", "void", "<error>", or class name
    is_array: bool = False

    def __str__(self) -> str:
        return f"{self.name}[]" if self.is_array else self.name

INT = MJType("int")
FLOAT = MJType("float")
BOOLEAN = MJType("boolean")
CHAR = MJType("char")
STRING = MJType("String")
VOID = MJType("void")
ERROR = MJType("<error>")

PRIMITIVE_NAMES: frozenset[str] = frozenset({"int", "float", "boolean", "char", "String"})

def class_type(name: str) -> MJType: return MJType(name=name, is_array=False)
def is_class_type(t: MJType) -> bool:
    return not t.is_array and t.name not in PRIMITIVE_NAMES and t.name not in {"void", "<error>"}

def array_of(t: MJType) -> MJType:
    if t.is_array or t in (VOID, ERROR) or is_class_type(t):
        raise ValueError(f"Cannot form array of {t}")
    return MJType(name=t.name, is_array=True)

def element_type(t: MJType) -> MJType:
    if not t.is_array:
        raise ValueError(f"{t} is not an array")
    return MJType(name=t.name, is_array=False)

def from_type_node(node: TypeNode) -> MJType:
    base = MJType(node.name)
    return array_of(base) if node.is_array else base

def literal_type(kind: str, lexeme: str) -> MJType:
    if kind == "NUMBER":
        return FLOAT if "." in lexeme else INT
    elif kind == "BOOLEAN":
        return BOOLEAN
    elif kind == "STRING":
        # Single-quoted string with exactly 1 character (or 1 escape sequence) is char
        if lexeme.startswith("'") and lexeme.endswith("'"):
            content = lexeme[1:-1]
            if len(content) == 1 or (len(content) == 2 and content[0] == "\\"):
                return CHAR
        return STRING
    return ERROR

def is_numeric(t: MJType) -> bool:
    return not t.is_array and t in (INT, FLOAT)

def is_assignable(target: MJType, source: MJType) -> bool:
    if target == ERROR or source == ERROR:
        return True
    if target == source:
        return True
    if target == FLOAT and source == INT:  # Numeric promotion: int -> float
        return True
    return False

def binary_result_type(op: str, left: MJType, right: MJType) -> MJType | None:
    if left == ERROR or right == ERROR:
        return ERROR
    # String concatenation
    if op == "+":
        if left == STRING and not right.is_array and right not in (VOID, ERROR) and not is_class_type(right):
            return STRING
        if right == STRING and not left.is_array and left not in (VOID, ERROR) and not is_class_type(left):
            return STRING
    # Arithmetic
    if op in ("+", "-", "*", "/", "%"):
        if left == INT and right == INT:
            return INT
        if is_numeric(left) and is_numeric(right):
            return FLOAT
        return None
    # Relational
    if op in ("<", ">", "<=", ">="):
        if is_numeric(left) and is_numeric(right):
            return BOOLEAN
        return None
    # Equality
    if op in ("==", "!="):
        if is_numeric(left) and is_numeric(right):
            return BOOLEAN
        if left == right and left != VOID:
            return BOOLEAN
        return None
    # Logical
    if op in ("&&", "||"):
        if left == BOOLEAN and right == BOOLEAN:
            return BOOLEAN
        return None
    return None

def unary_result_type(op: str, operand: MJType) -> MJType | None:
    if operand == ERROR:
        return ERROR
    if op == "-" and is_numeric(operand):
        return operand
    if op == "!" and operand == BOOLEAN:
        return BOOLEAN
    return None

def format_method_type(return_type: MJType, param_types: list[MJType]) -> str:
    params_str = ", ".join(str(p) for p in param_types)
    return f"{return_type}({params_str})"
```

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/symbol_table.py`
- `minijava/mj_types.py`
- `tests/test_symbol_table.py`
- `tests/test_mj_types.py`
- `samples/phase1_tests.txt`
- `samples/phase23_tests.txt`
- `docs/phase1_report.md`
- `docs/phase23_report.md`

### Files You Must Treat as Read-Only:
- `minijava/tokens.py` (M1)
- `minijava/diagnostics.py` (M1)
- `minijava/ast_nodes.py` (M1)
- `minijava/lexer.py` (M2)
- `minijava/ast_printer.py` (M2)
- `minijava/parser.py` (M4)
- `minijava/semantic.py` (M5)
- `minijava/driver.py` (M1)

---

## E. Implementation Instructions

1. **Implement `minijava/mj_types.py`:**
   - Define `MJType` with `frozen=True`.
   - Implement singletons, type check helpers, assignability rules, and binary/unary typing logic.
2. **Implement `minijava/symbol_table.py`:**
   - Implement `IdentifierEntry` and `IdentifierTable` for Phase 1.
   - Implement `format_identifier_table` strictly matching §7.4 of the contract.
   - Implement `Symbol`, `Scope`, and `ScopedSymbolTable` for Phase 3.
   - Implement `lookup_in_method`: traverse parents while `kind in (ScopeKind.BLOCK, ScopeKind.METHOD)`.
   - Implement `format_scoped_table` strictly matching §7.5 of the contract.
3. **Implement Unit Tests:**
   - In `tests/test_symbol_table.py`: test identifier tracking, duplicate identifier positions, scoped symbol table nesting, shadowing, and formatting.
   - In `tests/test_mj_types.py`: test type assignability, `int` to `float` promotion, string concatenation rules, operator validity, and array restrictions.
4. **Author Test Programs Deliverables:**
   - In `samples/phase1_tests.txt`: Write a series of diverse Mini Java test programs illustrating:
     - All keywords, literals, and operators.
     - Single-line and multi-line comments.
     - Lexical errors (invalid characters, invalid identifiers, unclosed strings, unclosed comments).
   - In `samples/phase23_tests.txt`: Mini Java test programs with class declarations, loops, conditionals, method calls, type mismatches, and scope errors.
5. **Author Phase 1 & 2/3 Reports:**
   - In `docs/phase1_report.md`: Author a 1-2 page technical report complying with *PDF p. 5*:
     - Section 1: Tokens implemented (reserved keywords, literals, operators, delimiters).
     - Section 2: How errors were handled (line/column tracking, descriptive error messages, non-halting scan).
     - Section 3: Challenges faced and how they were solved (maximal munch for operators, digit-prefixed identifiers, unclosed comments).
   - In `docs/phase23_report.md`: Author the technical report for Phase 2/3 covering grammar design and semantic rules.

---

## F. Local Testing Requirements

Execute these commands locally:
```bash
# 1. Byte-compile your modules
python3 -m compileall -q minijava/symbol_table.py minijava/mj_types.py tests/test_symbol_table.py tests/test_mj_types.py

# 2. Run unit tests
python3 -m unittest tests.test_symbol_table -v
python3 -m unittest tests.test_mj_types -v

# 3. Verify contract compliance
python3 -m unittest tests.test_contract -v
```

---

## G. Integration Requirements

- `IdentifierTable.add_occurrence(name, line, column)` is called by Member 2 (`lexer.py`).
- `format_identifier_table` and `format_scoped_table` are called by Member 1 (`driver.py`).
- `ScopedSymbolTable`, `Symbol`, and `MJType` are consumed by Member 5 (`semantic.py`).
- Ensure all method signatures match the contract exactly.

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
- `minijava/symbol_table.py` passes all ≥ 25 unit tests in `test_symbol_table.py`.
- `minijava/mj_types.py` passes all ≥ 20 unit tests in `test_mj_types.py`.
- `samples/phase1_tests.txt` and `samples/phase23_tests.txt` are created with comprehensive Mini Java test cases.
- `docs/phase1_report.md` (1-2 pages) and `docs/phase23_report.md` are fully drafted and compliant with *PDF p. 5*.

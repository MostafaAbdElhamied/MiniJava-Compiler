# Phase 1 Implementation Prompt for Member 3: Symbol Table & Submission Deliverables Lead

> **Instructions for Member 3:** Copy and paste this prompt in its entirety into your AI coding assistant. This prompt is specifically configured for **Phase 1 (Lexical Analysis)** of the Mini Java Compiler project.

---

## A. Role and Objective

You are **Member 3**, the **Symbol Table Engineer and Official Course Deliverables Lead** for a university team of five Computer Science students building **Phase 1 (Lexical Analyzer)** of a Mini Java Compiler for course CSCI415 (Fall 2026).

### Your Core Phase 1 Responsibilities:
1. **Identifier Symbol Table (`minijava/symbol_table.py`):**
   - Implement `IdentifierEntry` and `IdentifierTable` (*PDF p. 2*).
   - Track 1-based sequential index (order of appearance), identifier name, first seen coordinates (`first_line`, `first_column`), and list of all occurrence coordinates (`occurrences`).
   - Implement `format_identifier_table(table: IdentifierTable) -> str` matching §7.4 of the contract:
     `Index: 1, Name: Student, First seen: line 1, column 7, Occurrences: 1`
2. **Official Phase 1 Test Programs File (`samples/phase1_tests.txt`):**
   - Author the official input text file required for Phase 1 submission (*PDF p. 5*).
   - Must contain diverse, comprehensive Mini Java test programs demonstrating valid constructs, all token types, comments, and the 4 required lexical error categories.
3. **Official Phase 1 Technical Report (`docs/phase1_report.md`):**
   - Author the official 1-2 page technical report required for Phase 1 submission (*PDF p. 5*).
   - Must clearly answer the 3 mandatory questions:
     - (1) Tokens implemented.
     - (2) How errors were handled.
     - (3) Challenges faced and how they were solved.
4. **Automated Testing:**
   - Implement `tests/test_symbol_table.py` (≥ 20 unit tests).

---

## B. Authoritative Specification & Project Context

- **Official Course PDF:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/ProjectGuidelinesAndDetails.pdf` (5 pages)
- **Phase 1 Shared Contract:** `/home/yousefmahmoudali/Desktop/CompilerProject/Projects/Phase 1/SHARED_PROJECT_CONTRACT.md`
- **Project Root Directory:** `/home/yousefmahmoudali/Desktop/CompilerProject/`
- **Deadline:** **October 30, 2026**

---

## C. Shared Project Contract Reference (Verbatim)

### Symbol Table Specification (`minijava/symbol_table.py`)
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

---

## D. Strict Task Boundaries

### Files You Own (Create and Maintain):
- `minijava/symbol_table.py`
- `tests/test_symbol_table.py`
- `samples/phase1_tests.txt`
- `docs/phase1_report.md`

### Files You Must Treat as Read-Only:
- `minijava/tokens.py` (M1)
- `minijava/diagnostics.py` (M1)
- `minijava/driver.py` (M1)
- `minijava/lexer.py` (M2)
- `minijava/scanner_ident_num.py` (M4)
- `minijava/scanner_str_op.py` (M5)

---

## E. Implementation Instructions

1. **Implement `minijava/symbol_table.py`:**
   - Define `IdentifierEntry` and `IdentifierTable`.
   - Implement `add_occurrence(name, line, column)`: if first seen, create entry with `index = len(self._order) + 1`; append coordinate tuple to `occurrences`.
   - Implement `lookup(name)`, `entries()`, `__len__`, `__contains__`.
   - Implement `format_identifier_table(table)`: outputs formatted lines or `"(empty)"`.
2. **Implement Unit Tests (`tests/test_symbol_table.py`):**
   - Test adding first occurrence creates entry with index 1.
   - Test subsequent occurrences update coordinate list without changing index.
   - Test multiple distinct identifiers maintain appearance order.
   - Test `lookup`, `__contains__`, and `format_identifier_table`.
3. **Author Phase 1 Test Input Programs (`samples/phase1_tests.txt`):**
   - Create a well-structured text file containing multiple test programs:
     - Program 1: The official sample program from *PDF p. 4*.
     - Program 2: Complex arithmetic and boolean expressions.
     - Program 3: All keywords and delimiter combinations.
     - Program 4: Single-line and multi-line comments.
     - Program 5: Invalid character errors (`@`, `#`).
     - Program 6: Invalid identifier errors (`1name`, `3_count`).
     - Program 7: Unclosed string literal errors.
     - Program 8: Unclosed multi-line comment errors.
4. **Author Phase 1 Technical Report (`docs/phase1_report.md`):**
   - Draft a professional 1-2 page markdown report complying with *PDF p. 5*:
     - **Title & Metadata:** Course CSCI415, Project Title, Team Members, Date.
     - **1. Tokens Implemented:** Detailed breakdown of all 15 keywords, literal rules (`NUMBER`, `STRING`, `BOOLEAN`), operators (`MATHOP`, `LOGOP`, `ASSIGN`), delimiters, and `EOF`.
     - **2. How Errors Were Handled:** Explanation of 1-based coordinate tracking, descriptive error messages, non-halting scanning to capture all errors, and exact error reproduction.
     - **3. Challenges Faced & Solutions:** Maximal munch for two-character operators, distinguishing float dots from punctuation dots, detecting digit-prefixed invalid identifiers, and unclosed comment tracking.

---

## F. Local Testing Requirements

```bash
# 1. Byte-compile your modules
python3 -m compileall -q minijava/symbol_table.py tests/test_symbol_table.py

# 2. Run unit tests
python3 -m unittest tests.test_symbol_table -v

# 3. Verify contract compliance
python3 -m unittest tests.test_contract -v
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

- `minijava/symbol_table.py` passes all unit tests in `test_symbol_table.py`.
- `samples/phase1_tests.txt` contains comprehensive, diverse Mini Java test programs.
- `docs/phase1_report.md` is complete, professional, and directly addresses the 3 required questions (*PDF p. 5*).

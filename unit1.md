# UNIT 1: INTRODUCTION AND LEXICAL ANALYSIS

---

## PART 1: FOUNDATIONS OF COMPILATION

### 1.1 What is a Compiler? Analysis of the Source Program
Before we discuss phases, buffering, or tokens, we must establish absolute clarity on what a compiler actually is, why it exists, and how it fundamentally operates. If you are reading this with zero prior exposure to systems programming, imagine you have written a letter in English (your native language) but the recipient only understands Japanese. You need a translator. A **compiler** is precisely that: a specialized software system that translates a program written in a **high-level source language** (like C, C++, Java, or Python) into a semantically equivalent **low-level target language** (typically assembly language or machine code that a specific CPU can execute directly).

Let us define every term in that sentence explicitly:
- **Source Program:** The human-readable code you write. It uses English-like keywords (`if`, `while`, `return`), mathematical notation (`+`, `*`, `=`), and structural symbols (`{`, `}`, `;`). Computers cannot execute this directly.
- **Target Language:** The machine-specific instruction set. It consists of binary opcodes, register names, and memory addresses. It is unreadable to humans but directly executable by the CPU.
- **Semantically Equivalent:** This is critical. The translated program must do exactly what the source program intended. If the source calculates `area = 3.14 * r * r`, the target must produce the identical numerical result, even if the internal steps differ.
- **Translation vs. Interpretation:** A compiler translates the entire program upfront, producing a standalone executable file. An interpreter reads and executes the source code line-by-line at runtime without producing a separate executable. Compilation favors execution speed; interpretation favors development speed and portability.

#### The Analysis-Synthesis Model
Compilation is not a single magical step. It is a disciplined pipeline divided into two macroscopic halves: **Analysis** and **Synthesis**.

**Analysis (Understanding the Source):** The compiler must first comprehend what you wrote. It cannot translate what it does not understand. Analysis breaks the source program into constituent pieces and builds a structured representation. It occurs in three hierarchical layers:
1. **Linear Analysis (Lexical Analysis):** The raw stream of characters is scanned left-to-right. Characters are grouped into meaningful words called **tokens**. Whitespace, tabs, newlines, and comments are discarded because they carry no computational meaning. Think of this as identifying individual words in a sentence.
2. **Hierarchical Analysis (Syntax Analysis):** Tokens are nested into grammatical structures according to the language's rules. Expressions become statements, statements become blocks, blocks become functions. This builds a tree-like structure called a **Parse Tree** or **Abstract Syntax Tree (AST)**. Think of this as identifying subject, verb, object, and clause boundaries in a sentence.
3. **Semantic Analysis:** Syntax checks structure; semantics checks meaning. The compiler verifies that the syntactically correct structures actually make logical sense. It checks: Are variables declared before use? Are types compatible? Can you add a string to a floating-point number? Does a function return the promised type? Think of this as verifying that "The colorless green ideas sleep furiously" is grammatically correct but semantically nonsensical.

**Synthesis (Constructing the Target):** Once the source is fully understood, the compiler constructs the equivalent target program.
1. **Intermediate Code Generation:** The compiler produces a machine-independent representation that bridges high-level semantics and low-level instructions.
2. **Code Optimization:** The intermediate code is refined to run faster, use less memory, or consume less power, without changing its meaning.
3. **Code Generation:** The optimized representation is mapped to actual CPU instructions, registers, and memory layouts.

This analysis-synthesis pipeline is universal. Every production compiler (GCC, Clang, javac, rustc) follows this architectural blueprint, though the number of internal steps and optimization aggressiveness varies.

---

### 1.2 The Phases of a Compiler `[PYQ 1.1.1, 1.1.3, 1.1.6]`
A compiler is architecturally divided into **six primary phases**. Each phase consumes the output of the previous phase, transforms it, and passes it forward. Two cross-cutting components interact with every phase: the **Symbol Table Manager** and the **Error Handler**.

Let us define and dissect each phase with absolute precision.

#### Phase 1: Lexical Analysis (The Scanner)
- **Input:** Raw character stream from the source file.
- **Action:** Reads characters sequentially, groups them into lexemes, matches them against patterns, and emits tokens. Strips whitespace, comments, and preprocessor directives. Tracks line/column numbers for error reporting.
- **Output:** A stream of tokens in the format `<token_type, attribute_value>`.
- **Data Structure:** Interfaces with the Symbol Table to store identifier names.
- **Example:** Input `count = count + 1;` becomes `<ID, count> <ASSIGN, => <ID, count> <PLUS, +> <NUM, 1> <SEMI, ;>`

#### Phase 2: Syntax Analysis (The Parser)
- **Input:** Token stream from the lexical analyzer.
- **Action:** Validates the token sequence against the language's **Context-Free Grammar (CFG)**. A CFG is a formal set of production rules defining valid syntactic structures. The parser builds a hierarchical tree representation. If the token sequence violates grammar rules (e.g., missing semicolon, mismatched parentheses), a syntax error is raised.
- **Output:** Parse Tree or Abstract Syntax Tree (AST). The AST is a condensed version of the parse tree, omitting syntactic sugar like parentheses and semicolons, retaining only computational structure.
- **Example:** For `a = b + c * 70`, the AST enforces operator precedence: `*` binds tighter than `+`, so `c * 70` is evaluated first, then added to `b`, then assigned to `a`.

#### Phase 3: Semantic Analysis
- **Input:** AST from the syntax analyzer.
- **Action:** Performs meaning validation. Key tasks:
  - **Type Checking:** Ensures operands match operator expectations. `int + float` may require implicit conversion. `string + int` may be illegal.
  - **Scope Resolution:** Verifies that identifier references resolve to valid declarations in the correct lexical scope.
  - **Declaration Checking:** Ensures variables/functions are declared before use (in languages that require it).
  - **Control Flow Validation:** Checks `break`/`continue` are inside loops, `return` types match function signatures, etc.
- **Output:** Type-annotated AST. Nodes are decorated with type information, memory offsets, and scope metadata.
- **Example:** If `b` is `int` and `c` is `float`, the semantic analyzer inserts an implicit type conversion node: `int_to_float(b)` before addition.

#### Phase 4: Intermediate Code Generation
- **Input:** Annotated AST.
- **Action:** Translates the tree into a linear, machine-independent **Intermediate Representation (IR)**. The most common form is **Three-Address Code (TAC)**, where each instruction has at most three operands: `result = operand1 operator operand2`. This simplifies optimization and retargeting.
- **Output:** Sequence of IR instructions (TAC, Quadruples, Triples, or SSA form).
- **Example:** `a = b + c * 70` becomes:
  ```
  t1 = c * 70
  t2 = b + t1
  a = t2
  ```

#### Phase 5: Code Optimization
- **Input:** Intermediate code.
- **Action:** Improves the IR for performance (speed, space, power) without altering semantics. Optimizations are mathematically proven transformations. Common techniques:
  - **Constant Folding:** Evaluate constant expressions at compile time. `3 * 5` becomes `15`.
  - **Dead Code Elimination:** Remove code that never executes or whose results are never used.
  - **Common Subexpression Elimination:** Compute repeated expressions once and reuse the result.
  - **Loop Invariant Code Motion:** Move calculations outside loops if they don't change per iteration.
  - **Strength Reduction:** Replace expensive operations with cheaper equivalents. `x * 2` becomes `x << 1` (bit shift).
- **Output:** Optimized IR.
- **Example:** If `70` is constant and `c` is known at compile time, `c * 70` may be precomputed. Temporaries may be reused to reduce register pressure.

#### Phase 6: Code Generation
- **Input:** Optimized IR + Symbol Table.
- **Action:** Maps IR to target machine instructions. Critical subtasks:
  - **Instruction Selection:** Chooses optimal CPU instructions for each IR operation.
  - **Register Allocation:** Assigns frequently used variables to CPU registers (fast) vs. memory (slow). Uses graph coloring algorithms.
  - **Instruction Scheduling:** Orders instructions to avoid CPU pipeline stalls.
  - **Memory Layout:** Assigns stack offsets, heap pointers, and static data addresses.
- **Output:** Target assembly or relocatable machine code.
- **Example:** TAC `t1 = c * 70` may become x86 assembly: `MOV EAX, [c]`, `IMUL EAX, 70`, `MOV [t1], EAX`.

#### Supporting Components
- **Symbol Table:** A centralized database storing every identifier's name, type, scope, memory location, size, and attributes. Implemented as hash tables, trees, or chained scopes. Accessed by lexical, semantic, and code generation phases.
- **Error Handler:** Detects, reports, and attempts recovery from errors. Lexical errors: illegal characters. Syntax errors: grammar violations. Semantic errors: type mismatches. Warnings: suspicious but legal code. Recovery strategies include panic mode (skip tokens until synchronizing token), phrase-level (local correction), and global correction (theoretical, rarely used).

---

#### 🔍 Step-by-Step Phase Walkthrough: PYQ Strings `[PYQ 1.1.1 & 1.1.3]`
Let us trace exactly how the compiler processes each PYQ string. I will show the transformation at every phase so you can replicate this logic for any expression.

**String 1: `A = B * C + D / E`**
Assume all variables are `float`.

1. **Lexical Analysis:** Scans left-to-right. Emits:
   `<ID, A> <ASSIGN, => <ID, B> <MUL, *> <ID, C> <PLUS, +> <ID, D> <DIV, /> <ID, E> <SEMI, ;>`
   Whitespace is discarded. Each identifier is entered into the symbol table with type `float`.

2. **Syntax Analysis:** Validates against expression grammar. Builds AST respecting precedence (`*` and `/` higher than `+`) and left-associativity:
   ```
         (=)
        /   \
       A    (+)
           /   \
         (*)   (/)
        /  \   /  \
       B    C D    E
   ```

3. **Semantic Analysis:** All operands are `float`. No type conversion needed. Type of entire expression is `float`. AST is annotated with type tags.

4. **Intermediate Code Generation (TAC):**
   ```
   t1 = B * C
   t2 = D / E
   t3 = t1 + t2
   A = t3
   ```

5. **Code Optimization:** No constants, no dead code, no common subexpressions. IR remains unchanged.

6. **Code Generation (Generic Assembly):**
   ```
   LOAD  R1, B
   MUL   R1, C
   LOAD  R2, D
   DIV   R2, E
   ADD   R1, R2
   STORE A, R1
   ```

**String 2: `X = Y + Z * 8.0`**
Assume `X, Y, Z` are `float`. `8.0` is a float literal.

1. **Lexical:** `<ID, X> <ASSIGN, => <ID, Y> <PLUS, +> <ID, Z> <MUL, *> <NUM, 8.0> <SEMI, ;>`
2. **Syntax:** AST enforces `Z * 8.0` first, then `Y + result`.
3. **Semantic:** All types match `float`. No coercion.
4. **IR:**
   ```
   t1 = Z * 8.0
   t2 = Y + t1
   X = t2
   ```
5. **Optimization:** If `Z` is loop-invariant or constant, compiler may fold. Otherwise, unchanged.
6. **Code Gen:** Similar to above, but `8.0` may be loaded from a constant pool or embedded as an immediate floating-point value.

**String 3: `Area = 3.14 * radius * radius`**
Assume `Area, radius` are `float`.

1. **Lexical:** `<ID, Area> <ASSIGN, => <NUM, 3.14> <MUL, *> <ID, radius> <MUL, *> <ID, radius> <SEMI, ;>`
2. **Syntax:** Multiplication is left-associative. AST:
   ```
         (=)
        /   \
     Area   (*)
           /   \
         (*)  radius
        /   \
     3.14  radius
   ```
3. **Semantic:** All `float`. Valid.
4. **IR:**
   ```
   t1 = 3.14 * radius
   t2 = t1 * radius
   Area = t2
   ```
5. **Optimization:** Recognizes `radius * radius`. May apply strength reduction or register reuse. If `radius` is constant, folds entirely.
6. **Code Gen:** Loads `3.14` from constant pool, multiplies twice, stores to `Area`.

**String 4: `position := initial + rate * 60`**
Assume `position, initial, rate` are `real` (float). `60` is integer.

1. **Lexical:** `<ID, position> <ASSIGN, :=> <ID, initial> <PLUS, +> <ID, rate> <MUL, *> <NUM, 60> <SEMI, ;>`
2. **Syntax:** Standard precedence tree. `rate * 60` evaluated first.
3. **Semantic:** Type mismatch detected: `rate` is `real`, `60` is `int`. Compiler inserts implicit conversion: `inttoreal(60)`. Annotated AST reflects this.
4. **IR:**
   ```
   t1 = inttoreal(60)
   t2 = rate * t1
   t3 = initial + t2
   position = t3
   ```
5. **Optimization:** Constant folding applies: `inttoreal(60)` → `60.0`. IR becomes:
   ```
   t1 = rate * 60.0
   t2 = initial + t1
   position = t2
   ```
6. **Code Gen:** Floating-point multiplication and addition instructions. `60.0` embedded as immediate or constant pool reference.

**Key Takeaway:** Every string follows the identical pipeline. The differences emerge in semantic type handling and optimization opportunities. Master this trace pattern; it solves 90% of phase-walkthrough PYQs.

---

### 1.3 Front-End vs. Back-End & Modular Compiler Design `[PYQ 1.1.2]`
Compilers are logically partitioned into two halves. This is not arbitrary; it is an engineering necessity for **retargetability** (supporting multiple CPUs) and **multi-language support** (compiling C, C++, Rust, Fortran with shared infrastructure).

#### Front-End Responsibilities (Machine-Independent)
The front-end understands the **source language**. It knows nothing about the target CPU.
- **Tasks:** Lexical analysis, syntax analysis, semantic analysis, intermediate code generation, symbol table population, source-level error reporting.
- **Output:** A clean, machine-independent Intermediate Representation (IR).
- **Examples:** Parsing C syntax trees, validating Java type hierarchies, resolving Python dynamic scopes, generating LLVM IR from Swift.

#### Back-End Responsibilities (Machine-Dependent)
The back-end understands the **target architecture**. It knows nothing about the source language.
- **Tasks:** Machine-specific optimization, register allocation, instruction selection, instruction scheduling, stack frame layout, target code emission.
- **Input:** Common IR from any front-end.
- **Examples:** Mapping IR to x86-64 vs ARM64 vs RISC-V, applying peephole optimization for MIPS, vectorizing loops for AVX-512, handling calling conventions for Windows vs Linux.

#### Designing a Modular Compiler for Swappable Components
To enable easy swapping of front-ends and back-ends, the compiler must enforce a **strict interface contract** via a well-defined IR. Here is how you architect it:

1. **Unified Intermediate Representation (IR):** The IR is the bridge. It must be expressive enough to capture high-level semantics (types, control flow, function calls) but low enough to map efficiently to multiple architectures. Examples: LLVM IR, GCC GIMPLE, Java Bytecode, WebAssembly. The IR specification is frozen and versioned.

2. **Language-Specific Front-End Modules:** Each source language gets an isolated front-end plugin. It parses, validates, and emits the common IR. It links against a shared symbol table and error reporting library. Adding a new language requires only a new front-end; the back-end remains untouched.

3. **Target-Specific Back-End Modules:** Each CPU architecture gets an isolated back-end plugin. It consumes the common IR, applies target-aware optimizations, and emits assembly. Adding a new CPU requires only a new back-end; all existing front-ends automatically support it.

4. **Compiler Driver & Plugin Architecture:** A central driver program (`gcc`, `clang`) orchestrates the pipeline. It uses dynamic linking or configuration flags to load front-end and back-end modules at runtime. Example: `clang -target arm64 -std=c++17 source.cpp` loads the C++ front-end and ARM64 back-end.

5. **Serialization & Decoupling:** For extreme modularity, the IR can be serialized to disk or memory buffers between phases. This enables distributed compilation, incremental builds, and language servers (IDEs) that only run the front-end for autocomplete/type checking.

6. **Real-World Proof:** LLVM is the canonical example. Clang (C/C++), Swift, Rust, and Julia all emit LLVM IR. LLVM's back-ends generate code for x86, ARM, RISC-V, GPU, and WebAssembly. Swap the back-end flag, and the same front-end code compiles to a different architecture instantly.

---

### 1.4 Grouping of Phases: Single-Pass vs. Multi-Pass `[PYQ 1.1.5.ii]`
A **pass** is a complete traversal of the program representation (source, IR, or object code). Phases can be grouped into passes based on memory constraints, language semantics, and optimization requirements.

#### Single-Pass Compiler
- **Definition:** All phases are interleaved and executed in one sequential sweep over the source code. As tokens are recognized, they are immediately parsed, semantically checked, and translated to target code.
- **Constraints:** Requires strict declaration-before-use rules. Cannot handle forward references (e.g., calling a function defined later). Minimal or no optimization. Symbol table must be populated on-the-fly.
- **Example:** Early Pascal compilers, Turbo C (limited optimization), simple educational compilers.
- **Pros:** Extremely fast compilation. Low memory footprint. Simple implementation.
- **Cons:** Cannot compile modern languages with complex scoping, templates, or forward references. No global optimization. Poor code quality.

#### Multi-Pass Compiler
- **Definition:** Phases are separated into distinct passes. The compiler may read/write intermediate representations multiple times. Each pass focuses on a specific transformation or analysis.
- **Enables:** Forward reference resolution, whole-program optimization, separate compilation, complex type inference, template instantiation, link-time optimization (LTO).
- **Example:** Modern GCC, Clang, Java `javac` + JIT, Rust compiler.
- **Pros:** Handles complex language semantics. Enables aggressive optimization. Modular architecture. Better error recovery. Higher quality target code.
- **Cons:** Slower compilation. Higher memory/disk I/O. More complex implementation.

**Why Group Phases?** Historically, memory was measured in kilobytes. Compilers used multiple passes to fit within RAM limits. Today, grouping is driven by:
- **Optimization Scope:** Local optimization (within a function) vs. global optimization (across functions) vs. LTO (across translation units).
- **Language Semantics:** C++ templates require instantiation passes. Haskell type inference requires constraint-solving passes.
- **Incremental Compilation:** IDEs and build systems recompile only changed modules, requiring pass isolation.
- **JIT/AOT Hybridization:** Java compiles to bytecode (pass 1), then JIT compiles hot methods at runtime (pass 2+).

---

### 1.5 Cousins of the Compiler `[PYQ 1.1.4]`
Compilers do not operate in isolation. They exist in an ecosystem of translation, linking, and execution tools. Each "cousin" handles a specific stage of the software lifecycle.

| Tool | Definition & Role | Input | Output |
|------|-------------------|-------|--------|
| **Preprocessor** | A text-substitution tool that runs before compilation. Handles macro expansion, file inclusion, and conditional compilation. | Source code with directives (`#include`, `#define`, `#ifdef`) | Expanded source code (directives removed, macros replaced) |
| **Assembler** | Translates human-readable assembly language into relocatable machine code. Resolves symbolic labels to numeric offsets. | Assembly code (`.s` or `.asm`) | Relocatable object file (`.o` or `.obj`) |
| **Loader** | Loads an executable program into main memory for execution. Assigns absolute memory addresses, initializes stack/heap, and transfers control to `main`. | Executable file | Running process in RAM |
| **Linker** | Combines multiple relocatable object files and libraries into a single executable. Resolves external references (e.g., `printf` from libc), assigns final addresses, and patches jump targets. | Multiple `.o` files + libraries | Executable file (`.exe`, ELF, Mach-O) |
| **Interpreter** | Executes source code directly without producing a standalone executable. Performs lexing, parsing, and execution on-the-fly, line-by-line or AST-by-AST. | Source code or bytecode | Program output (side effects, return values) |
| **Cross-Compiler** | Runs on Host Machine A but generates code for Target Machine B. Essential for embedded systems, mobile development, and OS kernel building. | Source code | Target machine code for different architecture |
| **Source-to-Source Compiler** | Translates between high-level languages. Also called a transpiler. Preserves abstraction level but changes syntax/ecosystem. | High-level source A | High-level source B |
| **JIT Compiler** | Compiles bytecode to native machine code at runtime. Blends interpretation (fast startup) with compilation (fast execution). Profiles hot code paths and optimizes dynamically. | Bytecode/IR | Native machine code (cached in memory) |

**Pipeline Visualization:**
`Source.c` → Preprocessor → `Expanded.c` → Compiler → `Source.s` → Assembler → `Source.o` → Linker → `a.out` → Loader → RAM → CPU Execution.

---

### 1.6 Bootstrapping & Cross-Compilation `[PYQ 1.1.5.i]`
**Bootstrapping** is the process of writing a compiler for a language in the language itself. It solves the fundamental paradox: *How do you compile a language before its compiler exists?*

#### The Chicken-and-Egg Problem
If you want to write a C compiler, you need a compiler to compile it. But if C doesn't have a compiler yet, what do you use? You use a **subset** of the language, or another language, to build the first version. Then you use that first version to compile a more complete version. This is bootstrapping.

#### Bootstrap Arrangement for Machines A, B, and C
Assume we want a native C compiler for Machine C. We have:
- Machine A: Runs an existing compiler for a C subset (or another language like Assembly).
- Machine B: Can execute binaries compiled on A.
- Machine C: Target machine with no C compiler yet.

**Step 1: Write a Minimal Compiler in a Subset**
Write a simple C compiler using only a restricted subset of C (no complex libraries, no advanced features). Call this `C_subset_compiler.c`. It can compile basic C but is written in C_subset.

**Step 2: Compile on Machine A**
Use an existing tool on A (e.g., an assembler or a different language compiler) to compile `C_subset_compiler.c`. Now A has a working C compiler: `C_compiler_A`.

**Step 3: Cross-Compile for Machine B**
Modify `C_compiler_A`'s code generator to emit Machine B's assembly instead of A's. Compile this modified source on A using `C_compiler_A`. Output: `C_compiler_B` (runs on A, generates code for B). This is a **cross-compiler**.

**Step 4: Bootstrap on Machine B**
Run `C_compiler_B` on A to compile the full C compiler source code, targeting B. Transfer the resulting binary to B. Now B has a native C compiler: `C_compiler_B_native`.

**Step 5: Port to Machine C**
Repeat: Modify the compiler's back-end to target C. Cross-compile on B. Transfer to C. Compile the compiler's own source on C. Result: `C_compiler_C_native`. The compiler is now **self-hosting**.

**T-Diagram Notation (Conceptual):**
A compiler is denoted as `T^Source_Target_Host`.
- Initial: `T^Csubset_A_A` (compiled via external tool)
- Cross: `T^C_B_A` (runs on A, targets B)
- Native B: `T^C_B_B` (runs on B, targets B)
- Cross to C: `T^C_C_B` (runs on B, targets C)
- Native C: `T^C_C_C` (self-hosted)

Bootstrapping proves language maturity. GCC, Clang, Rust, Go, and Swift are all self-hosted.

---

### 1.7 Factors Deciding Compiler Phases `[PYQ 1.1.6]`
The number, grouping, and complexity of compiler phases are engineering decisions driven by:

1. **Source Language Complexity:** Languages with simple syntax (C, Pascal) need fewer passes. Languages with complex type systems (Haskell, Rust), template metaprogramming (C++), or dynamic features (Python, JavaScript) require additional semantic, instantiation, or runtime compilation passes.
2. **Target Architecture:** RISC architectures (ARM, RISC-V) have uniform instructions, simplifying code generation. CISC (x86) has complex, variable-length instructions, requiring sophisticated instruction selection and scheduling passes. Register count dictates allocation complexity.
3. **Memory & Storage Constraints:** Embedded compilers merge phases to fit in kilobytes of RAM. Historical compilers used multi-pass designs due to memory limits. Modern desktop compilers prioritize optimization over memory.
4. **Optimization Requirements:** Debug builds skip optimization passes for fast compilation. Release builds enable aggressive optimization (loop unrolling, vectorization, LTO), adding multiple IR transformation passes.
5. **Compilation Speed vs. Execution Speed Trade-off:** JIT compilers (V8, JVM) minimize initial passes for fast startup, then add optimization passes at runtime for hot code. AOT compilers (GCC, Clang) maximize upfront passes for peak runtime performance.
6. **Toolchain Ecosystem:** Availability of parser generators, IR frameworks, and linker capabilities influences phase boundaries. LLVM's modular IR encourages strict phase separation.
7. **Error Recovery & IDE Integration:** Compilers designed for IDEs (Roslyn, clang) include fault-tolerant parsing, incremental analysis, and real-time diagnostic passes that traditional batch compilers omit.

---

### 1.8 Tokens, Lexemes, and Patterns `[PYQ 1.1.5.iii]`
These three terms are foundational to lexical analysis. Confusing them is the most common student error. Let us define them with mathematical precision.

- **Lexeme:** The actual sequence of characters in the source code that matches a rule. It is a concrete string. Example: `count`, `3.14`, `>=`, `while`.
- **Pattern:** The formal rule (usually a regular expression) that describes the structure of all valid lexemes for a given token category. It is an abstract template. Example: `letter(letter|digit)*` describes all valid identifiers.
- **Token:** A logical category or classification of lexemes. It is an abstract symbol paired with an attribute. Format: `<token_name, attribute_value>`. Example: `<ID, pointer_to_symbol_table_entry_for_count>`.

**Analogy:** In English, "run", "running", "ran" are lexemes. The pattern is "verb forms of run". The token is `<VERB>`. The parser cares about the token; the lexer cares about the lexeme and pattern.

#### Token Types:
1. **Keywords:** Reserved words with fixed syntactic meaning. Cannot be used as identifiers. Examples: `if`, `else`, `while`, `return`, `void`, `int`.
2. **Identifiers:** User-defined names for variables, functions, classes, labels. Pattern: typically starts with letter/underscore, followed by alphanumeric/underscore.
3. **Literals/Constants:** Fixed values embedded in code. Subtypes: integer (`42`), float (`3.14`), character (`'A'`), string (`"hello"`), boolean (`true`).
4. **Operators:** Symbols denoting operations. Arithmetic (`+`, `-`, `*`, `/`, `%`), Relational (`<`, `<=`, `>`, `>=`, `==`, `!=`), Logical (`&&`, `||`, `!`), Assignment (`=`, `+=`, `:=`), Bitwise (`&`, `|`, `^`, `~`, `<<`, `>>`).
5. **Punctuators/Separators:** Structural delimiters. `;` (statement terminator), `,` (separator), `(` `)` (grouping/parameters), `{` `}` (blocks), `[` `]` (arrays).

**Example Trace:** `int x = 10;`
- `int` → Lexeme: "int", Pattern: keyword list, Token: `<KEYWORD, int>`
- `x` → Lexeme: "x", Pattern: `letter(letter|digit)*`, Token: `<ID, symtab_ptr>`
- `=` → Lexeme: "=", Pattern: assignment operator, Token: `<ASSIGN, =>`
- `10` → Lexeme: "10", Pattern: `digit+`, Token: `<NUM, 10>`
- `;` → Lexeme: ";", Pattern: punctuator, Token: `<SEMI, ;>`

---

## PART 2: LEXICAL ANALYSIS: THEORY & PRACTICE

### 2.1 The Role of the Lexical Analyzer `[PYQ 1.2.1]`
The lexical analyzer (scanner) is the compiler's first line of defense. It transforms raw text into structured tokens. Its responsibilities are precise and non-negotiable:

1. **Token Recognition:** Scans characters, groups them into lexemes, matches against patterns, and emits `<token, attribute>` pairs.
2. **Whitespace & Comment Stripping:** Discards spaces, tabs, newlines, and comments (`//`, `/* */`). The parser never sees them. This drastically simplifies grammar rules.
3. **Symbol Table Interaction:** When a new identifier is encountered, the lexer inserts it into the symbol table and returns a pointer/index as the token's attribute. Subsequent occurrences return the same pointer.
4. **Error Detection:** Reports illegal characters (e.g., `@` in C), malformed literals (e.g., `3.14.15`), unterminated strings/comments, and invalid escape sequences.
5. **Source Tracking:** Maintains line and column counters. When the parser reports an error, it references these coordinates for precise diagnostics.
6. **Macro Expansion (Optional):** Some scanners handle simple preprocessor directives or integrate with a separate preprocessor phase.

**Why Separate Lexical Analysis from Parsing?**
- **Simplicity:** Parser grammar remains clean. Without lexer separation, every grammar rule would need optional whitespace/comment tokens, exploding complexity.
- **Efficiency:** Specialized buffering and DFA-based pattern matching are orders of magnitude faster than CFG parsing. Lexing is O(n); parsing is O(n³) worst-case.
- **Portability:** Device-specific input handling (file encoding, line endings `\n` vs `\r\n`, BOM markers) is isolated in the scanner.
- **Modularity:** Token specifications can be updated without rewriting the parser. Language dialects share parsers but swap lexers.

---

### 2.2 Input Buffering: Schemes, Sentinels, and Algorithms `[PYQ 1.2.2, 1.2.3, 1.2.4]`
Reading characters one-by-one from disk or standard input is catastrophically slow. Disk I/O latency is measured in milliseconds; CPU cycles are nanoseconds. A naive `getchar()` loop would spend 99% of its time waiting for I/O. **Input buffering** solves this by loading chunks of source code into RAM, allowing the scanner to advance pointers at CPU speed.

#### Memory Layout & Pointers
The scanner maintains two critical pointers:
- `lexemeBegin`: Points to the first character of the current token being scanned.
- `forward`: Scans ahead character-by-character to find the token's end.
When a token is recognized, the lexeme is the substring from `lexemeBegin` to `forward-1`. `lexemeBegin` is then updated to `forward`.

#### One-Buffer Scheme
A single buffer of size N (e.g., 1024 bytes) is filled. The scanner reads until `forward` reaches the end. The buffer is then reloaded.
**Fatal Flaw:** If a token spans the buffer boundary (e.g., `lexemeBegin` is at position N-2, but the token ends at position N+5), it gets split. The scanner must detect this, save the partial lexeme, reload the buffer, prepend the saved fragment, and resume. This requires complex boundary checks, memory copies, and state preservation. Performance degrades significantly.

#### Two-Buffer Scheme (Double Buffering) `[PYQ 1.2.2]`
Two buffers of size N are used alternately: Buffer A and Buffer B.
- Initially, Buffer A is loaded. `lexemeBegin` and `forward` start at A[0].
- `forward` advances through A. When it reaches A's end, Buffer B is pre-loaded with the next N characters. `forward` wraps to B[0].
- When `forward` reaches B's end, Buffer A is reloaded. `forward` wraps to A[0].
- `lexemeBegin` follows `forward`. If `lexemeBegin` and `forward` are in different buffers, the lexeme spans the boundary, but the two-buffer scheme handles this seamlessly because both halves are in memory simultaneously.

**Advantages:**
- Eliminates boundary-split token reconstruction.
- I/O overlaps with scanning (while CPU scans Buffer A, disk can preload Buffer B).
- Only one boundary check per buffer instead of per character.
- Standard in production compilers (Flex, GCC, Clang).

#### Will a Three-Buffer Technique Improve Performance? `[PYQ 1.2.2]`
**No. The marginal gain is negligible, and the costs outweigh benefits.** Here is the rigorous justification:
1. **Diminishing Returns:** Two buffers already fully decouple I/O from scanning. The CPU processes tokens faster than disk/SSD can supply data. A third buffer adds memory overhead without reducing I/O wait time.
2. **Cache Locality:** Modern CPUs optimize for spatial and temporal locality. Three buffers increase the working set size, potentially evicting hot data from L1/L2 cache, causing cache thrashing and slowing down pointer arithmetic.
3. **Complexity vs. Gain:** Pointer management, synchronization, boundary logic, and state tracking become significantly more complex. The marginal throughput gain (<2% in synthetic benchmarks) is drowned out by branch mispredictions and memory management overhead.
4. **I/O Bottleneck Dominance:** Disk/SSD latency and OS page cache behavior dominate performance. Double buffering already hides this latency effectively. Triple buffering is useful in graphics/rendering pipelines (frame pacing to prevent tearing), not in sequential text scanning where data consumption is strictly linear.
5. **Empirical Evidence:** Production scanner generators (Flex, re2c, ANTLR) use double buffering. Decades of compiler engineering confirm it is the optimal trade-off.

#### Lookahead Code with Sentinels `[PYQ 1.2.3]`
Checking `if forward == end_of_buffer` on every character advance is expensive. It adds a conditional branch per character. In a 10,000-line program, this means millions of branch predictions, pipeline stalls, and comparison instructions. **Sentinels** optimize this by placing a special character (typically `EOF` or `#`) at the end of each buffer. The scanner only checks for the sentinel, eliminating boundary comparisons during normal scanning.

**Algorithm: Lookahead with Sentinels (Step-by-Step)**
```
Initialize:
  Load Buffer1 with N characters from source file
  Append sentinel character '#' at position N
  Load Buffer2 with next N characters
  Append sentinel '#' at position N
  lexemeBegin = forward = start of Buffer1
  currentBuffer = 1

Function Advance():
  c = buffer[currentBuffer][forward]
  forward = forward + 1
  
  if c == '#':
    // Sentinel hit. Determine if it's buffer boundary or true EOF.
    if currentBuffer == 1:
      if forward == N + 1:
        // True EOF reached (sentinel was placed after actual file end)
        return EOF
      else:
        // Buffer boundary. Switch to Buffer2.
        Reload Buffer2 with next N characters
        Append '#' at end
        currentBuffer = 2
        forward = 0
        c = buffer[2][forward]
        forward = forward + 1
    else: // currentBuffer == 2
      if forward == N + 1:
        return EOF
      else:
        Reload Buffer1 with next N characters
        Append '#' at end
        currentBuffer = 1
        forward = 0
        c = buffer[1][forward]
        forward = forward + 1
        
  return c
```
**Performance Improvement:** Reduces two comparisons per character (`forward >= N` AND `buffer[forward]`) to one (`c == '#'`). Eliminates branch misprediction on normal characters. In a 50,000-line codebase, this saves ~10-15 million CPU cycles, significantly accelerating the scanner. The sentinel acts as a "tripwire" that only triggers at boundaries, allowing the hot path (normal character scanning) to run without conditional checks.

#### Pseudocode Simulation: Two-Buffer Scheme `[PYQ 1.2.4]`
```
// Global configuration
BUFFER_SIZE = 1024
char Buffer1[BUFFER_SIZE + 1], Buffer2[BUFFER_SIZE + 1]
int lexemeBegin = 0, forward = 0
int currentBuffer = 1  // 1 or 2
FILE* srcFile
bool eofReached = false

// Load a buffer with source data and append sentinel
function loadBuffer(bufID):
    char* buf = (bufID == 1) ? Buffer1 : Buffer2
    count = fread(buf, 1, BUFFER_SIZE, srcFile)
    if count < BUFFER_SIZE:
        buf[count] = EOF_SENTINEL   // Marks actual end of file
        buf[count+1] = '\0'
        eofReached = true
    else:
        buf[BUFFER_SIZE] = BUFFER_SENTINEL  // Marks buffer boundary
    return count

// Get next character with automatic buffer switching
function getNextChar():
    char* buf = (currentBuffer == 1) ? Buffer1 : Buffer2
    c = buf[forward]
    forward++
    
    if c == BUFFER_SENTINEL:
        // Switch to alternate buffer
        if currentBuffer == 1:
            loadBuffer(2)
            currentBuffer = 2
        else:
            loadBuffer(1)
            currentBuffer = 1
        forward = 0
        buf = (currentBuffer == 1) ? Buffer1 : Buffer2
        c = buf[forward]
        forward++
        
    if c == EOF_SENTINEL:
        return EOF
    return c

// Form token by scanning until pattern breaks
function formToken():
    lexemeBegin = forward - 1
    state = START
    while true:
        c = getNextChar()
        switch(state):
            case START:
                if isLetter(c): state = IN_ID
                else if isDigit(c): state = IN_NUM
                else if isWhitespace(c): 
                    lexemeBegin = forward
                    continue
                else if c == '<': state = SAW_LT
                // ... handle other starts
            case IN_ID:
                if isAlnum(c) or c == '_': 
                    continue  // Keep consuming
                else:
                    retract() // forward--
                    return makeToken(ID, extractLexeme())
            case IN_NUM:
                if isDigit(c): continue
                else if c == '.': state = IN_FLOAT
                else:
                    retract()
                    return makeToken(NUM, extractLexeme())
            // ... other states
```
**Buffer Switching & Token Formation Explained:** 
- `forward` advances character-by-character. When it hits `BUFFER_SENTINEL`, the alternate buffer is loaded, and `forward` resets to 0. 
- `lexemeBegin` marks where the current token started. 
- When a character breaks the pattern (e.g., space after identifier), `forward` is retracted by one (`forward--`), the lexeme is extracted between `lexemeBegin` and `forward-1`, and a token is emitted. 
- The two buffers ensure that even if a token spans the boundary, both halves are in memory, so `extractLexeme()` can safely concatenate or copy without I/O blocking.

---

### 2.3 Specification of Tokens: Regular Expressions `[PYQ 1.2.1]`
Tokens are formally specified using **Regular Expressions (RE)**. An RE is a mathematical notation that describes a **regular language** (the simplest class in the Chomsky hierarchy). REs are built from alphabet symbols and operators. They are perfect for lexical analysis because tokens have flat, non-nested structure.

#### Fundamental Operations (Defined from Scratch)
Let `r` and `s` be regular expressions denoting languages `L(r)` and `L(s)`. A language is simply a set of strings.

1. **Union (`r | s`)**
   - Denotes `L(r) ∪ L(s)`. Matches strings in either language.
   - Example: `a | b` matches `a` or `b`. `digit | letter` matches any single digit or letter.
   - Properties: Commutative (`r|s = s|r`), Associative (`(r|s)|t = r|(s|t)`), Idempotent (`r|r = r`), Identity (`r|∅ = r`).

2. **Concatenation (`rs`)**
   - Denotes `{xy | x ∈ L(r), y ∈ L(s)}`. Matches strings formed by appending a string from `L(s)` to a string from `L(r)`.
   - Example: `ab` matches exactly `ab`. `digit letter` matches `0a`, `9Z`, etc.
   - Properties: Associative (`(rs)t = r(st)`), Distributive over union (`r(s|t) = rs|rt`), Identity (`rε = εr = r`), Annihilator (`r∅ = ∅r = ∅`).

3. **Kleene Closure (`r*`)**
   - Denotes zero or more repetitions: `∪_{i=0}^{∞} L(r)^i`. Includes the empty string `ε`.
   - Example: `a*` matches `ε, a, aa, aaa, ...`. `(0|1)*` matches all binary strings.
   - Properties: `r* = r r* | ε`, `(r*)* = r*`, `r* = (r|ε)*`, `∅* = ε`.

4. **Positive Closure (`r+`)**
   - Denotes one or more repetitions: `r r*`. Excludes `ε`.
   - Example: `a+` matches `a, aa, aaa, ...`. `digit+` matches `0, 42, 999`.
   - Properties: `r+ = r r*`, `r* = r+ | ε`.

5. **Optional (`r?`)**
   - Denotes zero or one occurrence: `r | ε`.
   - Example: `sign?` matches `+`, `-`, or nothing. `digit+ \.? digit+` matches integers or floats.

#### Precedence (Highest to Lowest)
1. `*`, `+`, `?` (Unary closure operators)
2. Concatenation (Implicit, left-to-right)
3. `|` (Union, lowest precedence)
Parentheses `()` override precedence.

#### Examples for Common Tokens
- **Identifier:** `letter (letter | digit)*` → Starts with letter, followed by zero or more alphanumeric.
- **Integer:** `digit+` → One or more digits.
- **Float:** `digit+ \. digit+ (E (+|-)? digit+)?` → Digits, decimal, digits, optional exponent.
- **Relational Op:** `< | <= | > | >= | == | !=` → Explicit enumeration.
- **Whitespace:** `( | \t | \n | \r)+` → One or more space/tab/newline/carriage-return.

**Limitation of REs:** They cannot count or match nested structures. You cannot write an RE for balanced parentheses `((()))` or `/* ... */` comments with arbitrary nesting. That requires Context-Free Grammars (parsers). Lexers handle flat patterns; parsers handle hierarchical patterns.

---

### 2.4 Recognition of Tokens: Transition Diagrams `[PYQ 1.2.5]`
A **Transition Diagram (TD)** is a graphical representation of a **Deterministic Finite Automaton (DFA)** used to recognize tokens. It is a flowchart for the scanner.

#### Components Defined:
- **States:** Circles representing scanning progress. Labeled 0, 1, 2, etc.
- **Transitions:** Directed arrows labeled with input characters or character classes. Indicate state changes.
- **Start State:** Initial state (usually 0). Marked with an incoming arrow.
- **Accepting States:** Double circles. Indicate a valid token has been recognized. May trigger `retract()` if lookahead consumed extra characters.
- **Actions:** Code executed on acceptance (return token, install in symbol table, update line counter).
- **Deterministic:** For any state and input character, there is exactly one next state. No ambiguity.

#### i. Identifier TD
Pattern: `letter (letter | digit)*`
```
State 0 (start) --[letter]--> State 1
State 1 --[letter|digit|_]--> State 1
State 1 --[other]--> State 2 (accept, retract)
Action at State 2: return <ID, install_lexeme()>
```
**Step-by-Step Trace for `count123;`:**
- Start at State 0. Read `c` (letter) → Move to State 1.
- Read `o` (letter) → Loop State 1.
- Read `u`, `n`, `t`, `1`, `2`, `3` → All match `letter|digit|_`. Loop State 1.
- Read `;` (other) → Move to State 2.
- State 2 is accepting. Retract `forward` by 1 (put `;` back). Lexeme = `count123`. Install in symbol table. Return `<ID, ptr>`.

#### ii. Relational Operator TD
Pattern: `< | <= | > | >= | == | !=`
```
State 0 --[<]--> State 1
State 1 --[=]--> State 2 (accept: <=)
State 1 --[other]--> State 3 (accept, retract: <)

State 0 --[>]--> State 4
State 4 --[=]--> State 5 (accept: >=)
State 4 --[other]--> State 6 (accept, retract: >)

State 0 --[=]--> State 7
State 7 --[=]--> State 8 (accept: ==)
State 7 --[other]--> Error (single = is assignment, not relational)

State 0 --[!]--> State 9
State 9 --[=]--> State 10 (accept: !=)
State 9 --[other]--> Error (! alone is logical NOT, not relational)
```
**Trace for `<=`:** State 0 → `<` → State 1 → `=` → State 2 (accept `<=`). No retract needed.
**Trace for `<`:** State 0 → `<` → State 1 → `a` (other) → State 3 (accept `<`, retract `a`).

#### iii. Unsigned Number TD
Pattern: `digit+ (\. digit+)? (E (+|-)? digit+)?`
```
State 0 --[digit]--> State 1
State 1 --[digit]--> State 1
State 1 --[.]--> State 2
State 1 --[E|e]--> State 4
State 1 --[other]--> State 3 (accept integer, retract)

State 2 --[digit]--> State 3
State 3 --[digit]--> State 3
State 3 --[E|e]--> State 4
State 3 --[other]--> State 3 (accept float, retract)

State 4 --[+|-]--> State 5
State 4 --[digit]--> State 6
State 5 --[digit]--> State 6
State 6 --[digit]--> State 6
State 6 --[other]--> State 7 (accept scientific, retract)
```
**Trace for `3.14E+2;`:**
- `3` → State 1
- `.` → State 2
- `1`, `4` → State 3 (loop)
- `E` → State 4
- `+` → State 5
- `2` → State 6
- `;` → State 7 (accept, retract `;`). Lexeme = `3.14E+2`. Return `<NUM, 314.0>`.

**Key Rule:** Always follow the **longest match** principle. If `3.14` and `3` both match, the scanner chooses `3.14`. TDs enforce this by consuming characters until the pattern breaks, then retracting one step.

---

### 2.5 Practical Tokenization Exercise `[PYQ 1.2.6]`
**Source Segment:**
```c
void sumnum(int i, int j) {
    int k = 6;
    return i+j;
}
```

**Line-by-Line Lexeme | Token | Pattern Analysis:**

| Lexeme | Token | Pattern / Processing Notes |
|--------|-------|---------------------------|
| `void` | `<KEYWORD, void>` | Matches keyword list. Reserved. |
| `sumnum` | `<ID, ptr_1>` | `letter(letter|digit|_)*`. Inserted into symbol table at index 1. |
| `(` | `<LPAREN, (>` | Punctuator. Delimits parameter list. |
| `int` | `<KEYWORD, int>` | Reserved type keyword. |
| `i` | `<ID, ptr_2>` | Identifier. Inserted at index 2. Type pending semantic analysis. |
| `,` | `<COMMA, ,>` | Separator. Delimits parameters. |
| `int` | `<KEYWORD, int>` | Reserved type keyword. |
| `j` | `<ID, ptr_3>` | Identifier. Inserted at index 3. |
| `)` | `<RPAREN, )>` | Punctuator. Closes parameter list. |
| `{` | `<LBRACE, {>` | Block opener. Enters new scope. |
| `int` | `<KEYWORD, int>` | Reserved type keyword. |
| `k` | `<ID, ptr_4>` | Identifier. Inserted at index 4. Local scope. |
| `=` | `<ASSIGN, =>` | Assignment operator. |
| `6` | `<NUM, 6>` | `digit+`. Integer literal. Attribute = numeric value 6. |
| `;` | `<SEMI, ;>` | Statement terminator. |
| `return` | `<KEYWORD, return>` | Control flow keyword. |
| `i` | `<ID, ptr_2>` | Reuses existing symbol table pointer. No new insertion. |
| `+` | `<PLUS, +>` | Arithmetic operator. |
| `j` | `<ID, ptr_3>` | Reuses existing symbol table pointer. |
| `;` | `<SEMI, ;>` | Statement terminator. |
| `}` | `<RBRACE, }>` | Block closer. Exits scope. |

**Critical Observations:**
- Whitespace and newlines are consumed but never emitted as tokens. The lexer skips them silently.
- `i` and `j` appear multiple times. The lexer does not create duplicate symbol table entries. It returns the same pointer, enabling the semantic analyzer to track variable usage across scopes.
- The lexer does not validate syntax. If you wrote `void sumnum(int i int j)`, the lexer would still emit valid tokens. The parser would catch the missing comma.
- Token attributes vary: keywords/punctuators often have null attributes (the token type is sufficient). Identifiers carry symbol table pointers. Literals carry numeric/string values.

---

## PART 3: COMPILER CONSTRUCTION TOOLS & LEX

### 3.1 Overview of Compiler-Construction Tools
Building a compiler from scratch is error-prone, time-consuming, and mathematically intensive. **Compiler-construction tools** automate phase implementation using formal specifications. They shift development from imperative coding to declarative specification.

1. **Scanner Generators:** Take regular expressions and produce lexical analyzers. Examples: LEX, Flex, JFlex, re2c.
2. **Parser Generators:** Take Context-Free Grammars and produce syntax analyzers. Examples: YACC, BISON, ANTLR, JavaCC, Menhir.
3. **Syntax-Directed Translation Engines:** Generate tree walkers or IR translators from grammar rules with embedded semantic actions. Examples: YACC/BISON `%code` blocks, ANTLR visitors/listeners.
4. **Code Generator Generators:** Map IR patterns to target instructions using tree-pattern matching. Examples: BURG, IBURG, LLVM TableGen, GCC machine descriptions.
5. **Data-Flow Analysis Engines:** Provide frameworks for optimization passes. Examples: control flow graph construction, liveness analysis, reaching definitions, SSA construction libraries.
6. **Compiler Driver/Build Systems:** Orchestrate toolchain execution. Examples: GCC driver, Clang, Make, CMake, Ninja.

These tools guarantee correctness (mathematically proven algorithms), improve maintainability (specifications are easier to read than C code), and enable portability (swap back-ends without rewriting front-ends).

---

### 3.2 The LEX Compiler: Architecture, Workflow, and Specification Format `[PYQ 1.3.1]`
**LEX** (and its modern GNU variant **Flex**) is a scanner generator that converts regular expression specifications into a C program implementing a DFA-based lexical analyzer. You write patterns and actions; LEX writes the scanning engine.

#### How LEX Works (Internal Workflow Explained from First Principles)
LEX does not magically understand your patterns. It applies rigorous automata theory algorithms:

1. **Input Parsing:** LEX reads your `.l` file and extracts regular expressions and associated C actions.
2. **NFA Construction (Thompson's Algorithm):** Each RE is converted to a **Nondeterministic Finite Automaton (NFA)**. An NFA allows multiple transitions for the same input and ε-transitions (moves without consuming input). Thompson's construction builds NFAs recursively:
   - `a` → two states, transition on `a`
   - `r|s` → new start state with ε-transitions to r and s NFAs
   - `rs` → connect r's accept to s's start via ε
   - `r*` → loop back with ε-transitions
3. **NFA to DFA Conversion (Subset Construction):** NFAs are inefficient for scanning (multiple paths, backtracking). LEX converts the combined NFA to a **Deterministic Finite Automaton (DFA)** using the subset construction algorithm. Each DFA state represents a set of NFA states. ε-closures are computed. Transitions are deterministic.
4. **DFA Minimization (Hopcroft's Algorithm):** The DFA may have redundant states. Hopcroft's algorithm partitions states into equivalent groups and merges them, producing the minimal DFA. This reduces memory footprint and speeds up transitions.
5. **Code Generation:** LEX emits `lex.yy.c` containing:
   - A transition table encoding the minimized DFA
   - A scanner driver loop (`yylex()`)
   - Double-buffering and sentinel management
   - User-defined actions embedded at accepting states
6. **Compilation:** You compile `lex.yy.c` with a C compiler and link it with your parser.

#### LEX Input Specification Format
A LEX file has three sections, separated by `%%`:

```
%{
  /* DEFINITIONS SECTION */
  C declarations, #includes, macros, global variables
  Regular expression definitions (name  pattern)
%}

%%
  /* RULES SECTION */
  pattern1    { action1 }
  pattern2    { action2 }
  ...
%%

  /* USER SUBROUTINES SECTION */
  Helper functions, main(), yywrap()
```

**Detailed Breakdown:**

1. **Definitions Section (`%{ ... %}` and macros)**
   - C code between `%{` and `%}` is copied verbatim to the output C file. Use it for `#include <stdio.h>`, `#include "y.tab.h"`, global variables, and helper prototypes.
   - Macros define reusable RE patterns:
     ```
     DIGIT   [0-9]
     LETTER  [a-zA-Z_]
     ID      {LETTER}({LETTER}|{DIGIT})*
     WS      [ \t\n]+
     ```
   - `%option noyywrap` disables default file-switching behavior (prevents linker errors if `yywrap` is undefined).

2. **Rules Section (`%% ... %%`)**
   - Each line: `regular_expression  { C_action }`
   - **Longest Match Rule:** LEX always matches the longest possible lexeme. If `>=` and `>` both match, `>=` wins because it's longer.
   - **Rule Priority:** If multiple patterns match the same longest length, the first rule in the file wins. Order matters.
   - Special variables available in actions:
     - `yytext`: Pointer to matched lexeme string (null-terminated)
     - `yyleng`: Length of matched lexeme
     - `yylex()`: Function that returns token to parser
     - `yylval`: Union for passing semantic values to YACC
   - Example:
     ```
     {ID}        { yylval.str = strdup(yytext); return IDENTIFIER; }
     {DIGIT}+    { yylval.num = atoi(yytext); return NUMBER; }
     {WS}        { /* ignore whitespace */ }
     .           { printf("Illegal char: %s\n", yytext); }
     ```

3. **User Subroutines Section**
   - Contains `main()` (if standalone), `yywrap()` (called at EOF; return 1 to stop, 0 to continue with new file), and helper functions.
   - Example:
     ```c
     int yywrap() { return 1; }
     int main() {
         yylex();
         return 0;
     }
     ```

#### LEX-YACC Integration
LEX rarely works alone. It pairs with **YACC** (parser generator):
- YACC defines token types in `%token` declarations.
- LEX includes `y.tab.h` (generated by YACC) to use token constants.
- LEX's `yylex()` returns tokens to YACC's `yyparse()`.
- Semantic values are passed via `yylval` union.

**Example Workflow:**
```bash
lex scanner.l        # Generates lex.yy.c
yacc -d parser.y     # Generates y.tab.c and y.tab.h
gcc lex.yy.c y.tab.c -o compiler
./compiler < source.c
```

#### Common Pitfalls & Best Practices
- **Longest Match Rule:** `>=` must appear before `>` in rules, or LEX will match `>` and leave `=` dangling. Actually, LEX handles longest match automatically, but rule order matters for equal-length matches.
- **Escape Characters:** Use `\` for regex metacharacters (`\+`, `\*`, `\.`). Inside `[]`, most metacharacters lose special meaning.
- **State Management:** Use `%x STATE` for exclusive start states (e.g., comment scanning, string literals). Example: `%x COMMENT`, then `<COMMENT>\*/ { BEGIN(INITIAL); }`.
- **Performance:** Avoid complex REs that cause DFA state explosion. Use character classes `[a-z]` instead of alternation `a|b|c|...`.
- **Error Handling:** Always include a catch-all rule `.` to report illegal characters instead of silent failures.
- **Memory Management:** `yytext` is overwritten on next call. Use `strdup()` if you need to preserve the lexeme.

---

## PART 4: PROFESSOR'S SYNTHESIS & EXAM STRATEGY

### Key Conceptual Threads
1. **Separation of Concerns:** Lexical analysis isolates character-level pattern matching from syntactic structure. This modularity is why compilers scale to millions of lines.
2. **Formal Foundations:** Regular expressions → NFA → DFA → Minimized DFA → C code. This pipeline is mathematically rigorous and guarantees correct token recognition.
3. **Performance Engineering:** Input buffering, sentinels, and DFA table compression are not academic curiosities; they are engineering necessities for scanning efficiently.
4. **Retargetability:** The front-end/back-end split via IR is the cornerstone of modern compiler ecosystems. Understand it deeply.

### How to Approach PYQs in Exams
- **Phase Walkthroughs:** Always show transformation per phase. Use TAC for IR. Mention symbol table updates. Draw trees if space permits. Trace character-by-character for lexical, show precedence for syntax, note type coercions for semantics.
- **Buffering Questions:** Draw pointer positions. Explicitly state why sentinels reduce comparisons. Justify three-buffer rejection with cache/I/O arguments. Show pseudocode structure.
- **Transition Diagrams:** Label states clearly. Mark start/accept. Show retract conditions. Explain longest-match behavior. Trace an example string.
- **LEX Specifications:** Structure answers in three sections. Explain `yytext`, `yyleng`, `yylval`. Mention longest-match and rule-priority rules. Show integration with YACC.
- **Bootstrapping:** Use T-diagram notation conceptually. Emphasize self-hosting and cross-compilation steps. Explain why it matters.

### Common Student Mistakes to Avoid
- Confusing **lexeme** (string), **token** (category), and **pattern** (rule). Memorize the definitions.
- Assuming parsers handle whitespace/comments. They don't; lexers do.
- Forgetting **retract** in transition diagrams when lookahead consumes a non-matching character.
- Claiming three buffers improve scanning performance. They don't; explain cache/IO/bottleneck reasons.
- Writing LEX rules without considering longest-match priority.
- Omitting symbol table interactions in phase walkthroughs.
- Treating phases as isolated. They are a pipeline; show data flow.

### Study Roadmap
1. Memorize the 6 phases + 2 support components. Know inputs, outputs, and tasks for each.
2. Practice tracing 5 different expressions through all phases. Write TAC manually.
3. Draw TDs for identifiers, numbers, operators, strings. Trace them.
4. Implement a minimal LEX file. Compile it. Inspect `lex.yy.c`. Understand the generated DFA table.
5. Simulate two-buffer scanning on paper. Move pointers. Handle sentinels.
6. Review PYQs. Time yourself. Write full answers, not bullet points.

---


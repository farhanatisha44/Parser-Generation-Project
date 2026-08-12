# Parser Generation Project

An ANTLR4 grammar for a small custom programming language ("Sapphire"), designed to explore how a parser generator builds a parse tree from a context-free grammar — combining syntax ideas from C, Python, and Ruby.

## Overview

This project is a learning exercise in parser generation: given a formally defined grammar, [ANTLR](https://www.antlr.org/) generates a lexer and parser that can validate source files written in the custom language and produce a parse tree showing exactly how the input was derived from the grammar rules.

The custom language mixes syntax influences from several languages:
- **C-style** blocks (`{ }`), functions declared with `@`, and header imports
- **Python-style** `if` / `elif` / `else` conditionals
- **Ruby-style** loop syntax and `br` for breaking out of loops

## Grammar (`project.g4`)

The grammar defines a program as:

```
root → library, one function, then zero or more subfunctions
```

### Key constructs

| Rule | Purpose |
|---|---|
| `library` | Header/import section — `import_sapphire`, `import_<header>`, and `def_<name>:<value>` constant definitions |
| `function` | The program entry point: `@base: { ... }` |
| `subfunction` | Additional user-defined functions: `@name(args): { ... }` |
| `block` / `body` | A `{ }`-delimited sequence of statements |
| `expst` | Expression statement |
| `ifst` | `if (expr) { } elif (expr) { } else { }` conditional |
| `loopst` | `loop(loopexpr) { }` — supports nested loops |
| `incremst` | Increment/decrement a variable (`var++`, `var--`) |
| `brkst` | `br` — break out of a loop |
| `inpst` | `in: var` — input statement |
| `outst` | `out: expr` — output/print statement |
| `funccall` | Function invocation |
| `expr` | Arithmetic (`+ - * /`), relational (`= != > >= < <= ==`), and logical (`& \|`) expressions, with parentheses |

### Tokens

- `ID` — one or more letters (`[a-zA-Z]+`)
- `LIT` — one or more digits (`[0-9]+`)
- `MIXER` — a single alphanumeric character
- Whitespace (`WS`) is skipped

## Example Program

```
import_sapphire
import_string_h

def_a:2

@base:
{
    a=1
    b=6
    if(a>b)
        {
        out:a
        }

    in:c

    loop(i = 1>> j = 10,5)
        {
        loop(i = 1>> j = 10,5)
            {
            a++
            br
            }
        }

    func1(2)

    if(a>b)
        {
        if(b>c)
            {
            c++
            }
        }
}
```

This is the contents of `Correct_input.txt`, which parses successfully against the grammar and produces the tree shown in `Correct parse tree.png`.

## Files

| File | Description |
|---|---|
| `project.g4` | The ANTLR4 grammar definition (lexer + parser rules) for the language |
| `Grammer.txt` | Plain-text copy of the same grammar, for quick reference outside an IDE |
| `Correct_input.txt` | A valid program in the custom language — parses without errors |
| `Correct parse tree.png` | The parse tree ANTLR generates for `Correct_input.txt` |
| `Wrong_input.txt` | An invalid program (missing `_` in `def`, missing `@` before `base`, missing `:`/`.` in `out`/`in` statements) — demonstrates a syntax error case |
| `Wrong parse tree.png` | Shows how the parser handles/reports the malformed input |

## How to Run

1. Install [ANTLR4](https://www.antlr.org/download.html) (requires Java) and the Python or Java runtime, depending on your target language.
2. Generate the lexer/parser from the grammar:
   ```bash
   antlr4 project.g4
   javac project*.java
   ```
3. Use ANTLR's `TestRig` (grun) to parse an input file and view the parse tree:
   ```bash
   grun project root -gui Correct_input.txt
   ```
   Replace `Correct_input.txt` with `Wrong_input.txt` to see how a malformed program is handled.
4. Compare the generated tree to `Correct parse tree.png` / `Wrong parse tree.png` to see the expected output.

## What This Demonstrates

- How a context-free grammar (CFG) written in ANTLR's `.g4` format is translated into a working lexer and parser.
- How nested rules (e.g., `ifst` containing `block` containing `body` containing more `ifst`/`loopst`) produce a hierarchical parse tree.
- The difference between a syntactically valid and invalid program under the same grammar, and how the parser surfaces that distinction.

## Possible Improvements

- Add a semantic analysis pass (type checking, scope resolution) on top of the parse tree.
- Build an interpreter or AST-based evaluator to actually execute programs written in this language.
- Add more example programs demonstrating each grammar feature (functions with arguments, nested elif chains, etc.).

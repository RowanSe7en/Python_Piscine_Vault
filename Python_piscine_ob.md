## Python Piscine

---

# Interpreting in Python

Interpreting is the process where Python executes code by translating it into instructions the Python Virtual Machine (PVM) can run, rather than compiling it directly into native machine code. Python is technically compiled to bytecode first, then interpreted by the PVM.

**Main steps in Python interpretation:**

1. Starting the Python Interpreter (CPython)
2. Lexing (Tokenization) – Converts raw source code into tokens, the smallest meaningful units of the program.
3. Parsing (Syntactic Analysis) – Checks the grammar of the token stream and builds an Abstract Syntax Tree (AST) representing program structure.
4. Bytecode Generation – Translates the AST into Python bytecode, a low-level, platform-independent instruction set.
5. Execution by PVM – The Python Virtual Machine reads and executes the bytecode.
6. Optional .pyc Compilation – Generates cached bytecode files for faster future execution without running the program.

---

# 1 — Starting the Python Interpreter (CPython)

Before any Python source code is read, the operating system starts the Python interpreter itself, which is the CPython executable. When you run `python3 script.py`, the OS locates the compiled `python3` binary (built from C code using a compiler such as GCC), loads it into memory, sets up the process environment (stack, heap, and CPU registers) like every native program (Python, Chrome, GCC, Bash, etc.), and jumps to its entry point (`main()` in C). CPython then initializes its runtime environment by setting up memory management, the garbage collector, built-in types, the import system, and the Python Virtual Machine (PVM). All of this work is performed in machine code, before any lexing, parsing, or bytecode generation of your `.py` file begins. Only after this initialization phase does CPython read and compile the Python source code.

```
1) OS starts python3 process
  |
  ├── sets up stack, heap, registers
  ├── loads python3 binary
  └── jumps to main()

2) CPython initializes runtime
  |
  ├── memory manager
  ├── garbage collector
  ├── built-in types
  ├── import system
  └── Python Virtual Machine

3) Python reads your .py file
  |
  ├── lexing
  ├── parsing
  ├── bytecode generation
  └── execution by PVM
```

---

# 2 — Lexing (Lexical Analysis / Tokenization)

Lexing is the first step in Python's frontend, performed by the Python compiler component inside the CPython interpreter, where the raw source code is converted into a stream of meaningful symbols called **tokens**. It turns the human-readable text into units the parser can understand.

You can see the lexer's output by running: `python3 -m tokenize test.py`

**What the lexer does:**
- Reads characters left to right
- Groups them into tokens based on rules
- Ignores irrelevant characters (spaces, comments)
- Tracks indentation (very important in Python)

**Token types include:**
- Keywords: `if`, `for`, `def`, `return`
- Identifiers: variable and function names
- Literals: numbers, strings
- Operators: `+ - * / =`
- Delimiters: `() [] {} , :`
- Structural tokens: `INDENT`, `DEDENT`, `NEWLINE`

**Example source file (`test.py`):**

```python
## test.py

a = 2
x = a + 3

def greet(name):
    print("Hello,", name)

greet("Alice")
```

**Tokenizer output:**

```
└─$ python3 -m tokenize test.py
0,0-0,0:            ENCODING       'utf-8'
1,0-1,9:            COMMENT        '# test.py'
1,9-1,10:           NL             '\n'
2,0-2,1:            NAME           'x'
2,2-2,3:            OP             '='
2,4-2,5:            NAME           'a'
2,6-2,7:            OP             '+'
2,8-2,9:            NUMBER         '3'
2,9-2,10:           NEWLINE        '\n'
3,0-3,1:            NL             '\n'
4,0-4,3:            NAME           'def'
4,4-4,9:            NAME           'greet'
4,9-4,10:           OP             '('
4,10-4,14:          NAME           'name'
4,14-4,15:          OP             ')'
4,15-4,16:          OP             ':'
4,16-4,17:          NEWLINE        '\n'
5,0-5,4:            INDENT         '    '
5,4-5,9:            NAME           'print'
5,9-5,10:           OP             '('
5,10-5,18:          STRING         '"Hello,"'
5,18-5,19:          OP             ','
5,20-5,24:          NAME           'name'
5,24-5,25:          OP             ')'
5,25-5,26:          NEWLINE        '\n'
6,0-6,1:            NL             '\n'
7,0-7,0:            DEDENT         ''
7,0-7,5:            NAME           'greet'
7,5-7,6:            OP             '('
7,6-7,13:           STRING         '"Alice"'
7,13-7,14:          OP             ')'
7,14-7,15:          NEWLINE        '\n'
8,0-8,0:            ENDMARKER      ''
```

---

# 3— Parsing (Syntactic Analysis)

Parsing is the second step in Python's frontend, performed by the Python compiler component inside the CPython interpreter. The stream of tokens from the lexer is analyzed according to Python's grammar rules. The parser checks that the code is structurally correct and builds a hierarchical representation called an **Abstract Syntax Tree (AST)**, which captures the meaning of the program without irrelevant details like whitespace or comments.

- **Input:** stream of tokens from the lexer
- **Output:** Abstract Syntax Tree (AST)
- Run: `python3 -m ast test.py`

**What the parser does:**
- Ensures tokens follow Python's syntax rules
- Organizes statements, expressions, and blocks hierarchically
- Detects syntax errors (e.g., missing colons, unmatched parentheses, incorrect indentation)
- Removes unnecessary details like formatting and comments

**AST output:**

```
└─$ python3 -m ast test.py
Module(
  body=[
     Assign(
        targets=[
           Name(id='x', ctx=Store())],
        value=BinOp(
           left=Name(id='a', ctx=Load()),
           op=Add(),
           right=Constant(value=3))),
     FunctionDef(
        name='greet',
        args=arguments(
           args=[
              arg(arg='name')]),
        body=[
           Expr(
              value=Call(
                 func=Name(id='print', ctx=Load()),
                 args=[
                    Constant(value='Hello,'),
                    Name(id='name', ctx=Load())]))]),
     Expr(
        value=Call(
           func=Name(id='greet', ctx=Load()),
           args=[
              Constant(value='Alice')]))])
```

---

# 4 — Bytecode Generation and Inspection

After parsing, Python converts the AST into **bytecode** — a low-level, platform-independent set of instructions that the Python Virtual Machine (PVM) can execute. Bytecode is analogous to assembly code in C, but instead of running directly on the CPU, it runs on Python's virtual machine. The PVM interprets Python bytecode and executes its semantics by invoking precompiled C functions, whose machine instructions are executed directly by the CPU.

Inspect bytecode with: `python3 -m dis test.py`

**What this shows:**
- Each line represents a single bytecode instruction.
- Instructions include operations like:
  - `LOAD_NAME` → load a variable
  - `LOAD_CONST` → load a constant value
  - `BINARY_ADD` → perform addition
  - `STORE_NAME` → store a value in a variable
  - `RETURN_VALUE` → return from a function or module

**Disassembly output:**

```
└─$ python3 -m dis test.py
 0           RESUME                   0

 2           LOAD_NAME                0 (a)
             LOAD_CONST               0 (3)
             BINARY_OP                0 (+)
             STORE_NAME               1 (x)

 4           LOAD_CONST               1 (<code object greet at 0x7f2cc09375a0, file "test.py", line 4>)
             MAKE_FUNCTION
             STORE_NAME               2 (greet)

 7           LOAD_NAME                2 (greet)
             PUSH_NULL
             LOAD_CONST               2 ('Alice')
             CALL                     1
             POP_TOP
             RETURN_CONST             3 (None)

Disassembly of <code object greet at 0x7f2cc09375a0, file "test.py", line 4>:
 4           RESUME                   0

 5           LOAD_GLOBAL              1 (print + NULL)
             LOAD_CONST               1 ('Hello,')
             LOAD_FAST                0 (name)
             CALL                     2
             POP_TOP
             RETURN_CONST             0 (None)
```

---

# 5 — Execution by the Python Virtual Machine (PVM)

After Python generates bytecode, it does not run directly on your CPU. Instead, the **Python Virtual Machine (PVM)** executes it. The PVM is the interpreter inside Python that reads bytecode instructions one by one and performs the corresponding operations in memory.

**How the PVM works:**
1. **Fetch** – Reads the next bytecode instruction.
2. **Decode** – Understands what operation it represents (addition, function call, variable assignment).
3. **Execute** – Performs the operation using Python's runtime environment:
   - Manages memory for variables and objects
   - Handles function calls and returns
   - Performs arithmetic and logical operations
   - Manages control flow (`if`, `for`, `while`)

**Key features of the PVM:**
- **Platform-independent** – Runs the same bytecode on different systems.
- **Manages memory automatically** – Includes garbage collection for unused objects.
- **Handles dynamic features** – Python can create and modify objects at runtime, including functions and classes.
- **Interpreted execution** – Unlike C, instructions are executed one at a time instead of being compiled to machine code beforehand.

**Example — how `x = a + 3` is executed:**

```python
LOAD_NAME a       # Fetch the value of 'a'
LOAD_CONST 3      # Load constant 3
BINARY_ADD        # Add them together
STORE_NAME x      # Store the result in 'x'
```

**The PVM's evaluation loop (C pseudocode):**

```c
while (1) {
   opcode = *ip++;
   switch (opcode) {
       case LOAD_FAST:
           ...
       case BINARY_ADD:
           ...
   }
}
```

When the PVM encounters the `BINARY_ADD` opcode, it invokes a C function that pops operands from the stack, performs the addition, and pushes the result back. Thus, the actual CPU instructions executed originate from precompiled C code, not from any direct translation of Python bytecode.

---

# 6 — Generate .pyc Files

In Python, you can compile a script to bytecode without executing it. This produces a `.pyc` file, which is a cached version of the bytecode. Python uses these files to speed up future executions by loading the precompiled bytecode instead of recompiling the source code each time.

```bash
python3 -m py_compile your_file.py
## Output: __pycache__/your_file.cpython-312.pyc
```

**`.pyc` is not machine code.** A `.pyc` file looks like random symbols, but it stores Python bytecode — a low-level, platform-independent instruction set for the PVM. The CPU never runs `.pyc` directly; the PVM reads and interprets the bytecode.

**The "compiler" word:** Python does have a compilation step, but it is very different from C/C++ compilation. The Python compiler converts source code (`.py`) into tokens, then into an AST, and finally into bytecode. It can optionally save this bytecode to a `.pyc` file. Unlike C or C++, Python does **not** generate machine code at this stage.

**Library modules** are included at runtime, not compile time. When the PVM encounters an `import` statement, it searches for the module in the standard library, `site-packages`, and the current working directory. If a `.pyc` file exists, Python loads the precompiled bytecode; otherwise it compiles the `.py` source first.

---

# Python Interpreter Execution Flow

```
┌─────────────────────────────────────┐
│ 1. OS starts CPython                │
│    OS locates python3, loads binary │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 2. CPython runtime initialization   │
│    Initialize memory, GC, types, PVM│
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 3. Read source code (.py file)      │
│    CPython reads Python source file │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 4. Lexing & Parsing                 │
│    Tokenize → parse into AST        │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 5. Bytecode generation              │
│    Compile AST into bytecode        │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 6. Create the __main__ module       │
│    Setup namespace, __name__="..."  │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 7. Execute bytecode (PVM)           │
│    PVM runs instructions, output    │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 7.1. Library Modules                │
│    (Imported at runtime)            │
└─────────────────────────────────────┘
```

---

# Python3 Command

`python3` is the command used to run the Python3 interpreter. It exists because Python has two major versions: Python 2 (now obsolete) and Python 3 (current and supported). On many systems, `python` may refer to Python 2 or may not exist at all, so `python3` is provided to explicitly run Python 3.

When you run `python3` and see something like `[GCC 15.2.0]`, this does not mean GCC is compiling your Python code; it simply indicates that the Python interpreter itself (CPython) was compiled using GCC 15.2.0.

On most modern systems, `python`, `python3`, and `python3.13` all ultimately execute the same binary executable, all found in `/usr/bin/`.

---

# Module

A **module** is a Python file that contains reusable code, including functions, classes, and variables, designed to organize and simplify your programs. Modules can be:
- **Standard** – built-in modules like `math`, `os`
- **Third-party** – installed via package managers like `pip`
- **Custom** – created by users

Modules promote modularity and code reuse, reducing redundancy and improving maintainability.

**Example (`Test1.py`):**

```python
def function(number):
    print(f"From Test{number}: {number * 10}")

function(1)
```

**Example (`Test2.py`):**

```python
import Test1

Test1.function(2)
```

**Output:**
```
python Test2.py

From Test1: 10
From Test2: 20
```

---

# How Python Executes a Module on Import

When a Python file is executed, the interpreter processes its statements sequentially from top to bottom. If it encounters an `import` statement, Python temporarily pauses execution and begins loading the target module. During this import process, Python:

1. Creates a module object
2. Sets the module's `__name__` variable to the module's filename (e.g., `"file"`)
3. Executes all top-level code line by line
4. Evaluates any conditional logic that depends on `__name__`
5. Stores the fully executed module in `sys.modules` for caching

Only after this execution finishes does Python make the module's functions, variables, and objects available to the importing file. **A useful mental model: "import means run the file, then give me access to its names."**

**Example (`one.py`):**

```python
print("one.py: top-level code running")

def hello():
    print("Hello from one")

print("one.py: module loaded")
```

**Example (`two.py`):**

```python
print("two.py: start")

import one

print("two.py: after import")

one.hello()
```

**Output:**
```
python two.py

two.py: start
one.py: top-level code running
one.py: module loaded
two.py: after import
Hello from one
```

**Module execution timeline (import case):**

```
two.py starts
└── two.py set (__name__="__main__")
└── import one.py
     ├── create module object and set __name__ = "one"
     ├── execute top-level code
     └── store in sys.modules
└── continue two.py
```

**What happens when you import `one` inside `two.py`:**

1. Python is already running: `two.py` is executing inside the `__main__` module object.
2. Python sees: `import one`.
3. Python checks: Is `"one"` already loaded? It looks in `sys.modules`.
4. If NOT loaded yet:
   - Python creates a new Module object named `"one"`, sets `__name__ = "one"`
   - Reads `one.py`, lexes → parses → compiles → executes it
   - Stores the executed module object in `sys.modules`
5. Execution returns to `two.py`. `one` is now a reference to that module object.
6. If imported again: Python does NOT re-run the file. It reuses the existing module object.

---

# Module Object Inspection

**`t1.py`:**

```python
#t1.py
import t2

print("Module object:", t2)
print("Type of module:", type(t2))
print("Module attributes:", dir(t2))
print("Access Module attributes:", t2.x)
print("Access Module attributes:", t2.__name__)

import sys

print("\nIs 't2' in sys.modules?", "t2" in sys.modules)
print("sys.modules['t2']:", sys.modules["t2"])
```

**`t2.py`:**

```python
#t2.py
print("t2 module is being executed")

x = 10

def greet():
    print("Hello from t2")
```

**Output:**

```
python t1.py

t2 module is being executed
Module object: <module 't2' from '/home/kali/1337/python/test/t2.py'>
Type of module: <class 'module'>
Module attributes: ['__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'greet', 'x']
Access Module attributes: 10
Access Module attributes: t2

Is 't2' in sys.modules? True
sys.modules['t2']: <module 't2' from '/home/kali/1337/python/test/t2.py'>
```

---

# `if __name__ == "__main__"`

The `if __name__ == "__main__":` check determines whether the Python script is being **run directly** or **imported as a module**.

- When run directly → `__name__` is set to `"__main__"`
- When imported → `__name__` is set to the name of the module

**`Test1.py`:**

```python
def greet():
    print("Hello from Test1!")

if __name__ == "__main__":
    greet()
    print(f'Script is being run directly, in \"{__name__}\" file')
else:
    print(f"This Code is Imported to {__name__} file")
```

**`Test2.py`:**

```python
import Test1
Test1.greet()
```

**Output when running `Test1.py` directly:**
```
pyhton Test1.py

Hello from Test1!
Script is being run directly, in "__main__" file
```

**Output when running `Test2.py` (which imports `Test1`):**
```
pyhton Test2.py

This Code is Imported to Test1 file
Hello from Test1!
```

---

# How Python Stores Variables, Functions, and Classes

Python uses both a **stack** and a **heap**, but in a highly abstracted and automatic way compared to languages like C. In Python, **everything is an object**, including integers, strings, functions, classes, modules, lists, and dictionaries.

Variables do not store values directly; they store **references (pointers)** to objects in memory. All objects are allocated in the **heap**. Variables themselves exist as references within a namespace (such as a local stack frame or module scope), pointing to those heap-allocated objects.

When `x = 10` is executed:
- Python creates (or reuses) the integer object `10` in the heap
- The name `x` becomes bound to that object
- If you write `x = 20`, Python binds `x` to a **different** integer object `20` — the original `10` object remains unchanged (because integers are immutable)

Global variables are maintained in a **module namespace dictionary** accessible via `globals()`. Every scope in Python is backed by a dictionary (`module.__dict__`, `frame.f_locals`, `class.__dict__`, or `object.__dict__`) rather than fixed memory segments. Python manages memory automatically using reference counting and a garbage collector.

**`*1` — Everything is an object:**

```python
## Integers
x = 10
print(x, "→ type:", type(x), "→ id:", hex(id(x)))

## Strings
s = "hello"
print(s, "→ type:", type(s), "→ id:", hex(id(s)))

## Lists
lst = [1, 2, 3]
print(lst, "→ type:", type(lst), "→ id:", hex(id(lst)))

## Dictionaries
d = {"a": 1, "b": 2}
print(d, "→ type:", type(d), "→ id:", hex(id(d)))

## Functions
def greet():
    return "Hello"
print(greet, "→ type:", type(greet), "→ id:", hex(id(greet)))

## Classes
class Car:
    pass
print(Car, "→ type:", type(Car), "→ id:", hex(id(Car)))

## Modules
import math
print(math, "→ type:", type(math), "→ id:", hex(id(math)))
```

**Output:**
```
10 → type: <class 'int'> → id: 0x73753836c430
hello → type: <class 'str'> → id: 0x737537a34d80
[1, 2, 3] → type: <class 'list'> → id: 0x73753794e640
{'a': 1, 'b': 2} → type: <class 'dict'> → id: 0x7375379d7fc0
<function greet at 0x7375379731a0> → type: <class 'function'> → id: 0x7375379731a0
<class '__main__.Car'> → type: <class 'type'> → id: 0x62a2f96b8c50
<module 'math' from '...'> → type: <class 'module'> → id: 0x737537a3a840
```

**`*2` — Immutable vs mutable objects:**

```python
## Immutable object (int)
x = 10
y = x

print("x id:", hex(id(x)))
print("y id:", hex(id(y)))

## Change y
y = 20

print("After changing y:")
print("x id:", hex(id(x)), "x value:", x)
print("y id:", hex(id(y)), "y value:", y)

## Mutable object (list)
a = [1, 2, 3]
b = a

print("Before change:")
print("a id:", hex(id(a)), "b id:", hex(id(b)))

## Modify b — it's like a is modified
b.append(4)

print("After change:")
print("a:", a, "a id:", hex(id(a)))
print("b:", b, "b id:", hex(id(b)))
```

**Output:**
```
x id: 0xa4fc30
y id: 0xa4fc30
After changing y:
x id: 0xa4fc30 x value: 10
y id: 0xa4fd70 y value: 20
Before change:
a id: 0x7ffa63906480 b id: 0x7ffa63906480
After change:
a: [1, 2, 3, 4] a id: 0x7ffa63906480
b: [1, 2, 3, 4] b id: 0x7ffa63906480
```

**`*3` — globals():**

```python
## Global variable
x = 42
y = "hello"

def fun():
    not_g = 15
    print(not_g)

print("Global variables stored in globals():")
print(globals())
print("Access x via globals():", globals()['x'])
print("Access y via globals():", globals()['y'])
```

**Output:**
```
{'__name__': '__main__', ..., 'x': 42, 'y': 'hello', 'fun': <function fun at 0x...>}
Access x via globals(): 42
Access y via globals(): hello
```

---

# Shebang

A **shebang** is a special line at the very beginning of a script that starts with `#!`. It is a Unix convention indicating which interpreter should be used to execute the file.

Although the shebang looks like a comment in Python and is ignored by the Python interpreter itself, it is read by the **operating system** before execution. When a script is run directly, the OS reads the shebang, locates the specified interpreter, and executes the file using that interpreter.

```python
#!/usr/bin/env python3
```

Using `#!/usr/bin/env python3` does **not** mean that the `python3` interpreter is located inside `/usr/bin/env`. Instead, `/usr/bin/env` is a standard Unix utility that searches for the specified program in the system's `PATH` environment variable.

Alternatively, you can use a direct shebang like `#!/usr/bin/python3`, which explicitly specifies the absolute path. This works on systems where Python is installed at that exact location but is less portable. Using `#!/usr/bin/env python3` is generally preferred because it allows the script to locate the appropriate Python interpreter dynamically.

---

# Try & Except

Python provides a structured mechanism to handle runtime errors using the `try` statement and its associated blocks.

- **`try` block:** Code that might cause an exception. If an error occurs, Python immediately stops executing the `try` block and looks for a matching `except` block.
- **`except` block:** Handles specific exceptions. You can catch individual exception types or multiple types in a single block. If the exception is not caught, Python propagates it up the call stack.
- **`else` block:** Optional. Runs only if the `try` block succeeds without raising any exceptions.
- **`finally` block:** Optional but always executes, regardless of whether an exception occurred. Used for cleanup tasks (closing files, releasing resources).

**Example 1 — basic try/except/else/finally:**

```python
try:
    print("Trying to water plants...")
    plant = None
    if not plant:
        print("Error: No plant to water!")
except ValueError:
    print("Caught a ValueError!")
else:
    print("Watering successful!")
finally:
    print("Closing watering system (cleanup)")
```

**Example 2 — unhandled exception (finally still runs):**

```python
try:
    x = 10 / 0  # This raises ZeroDivisionError
except ValueError:
    print("Caught a ValueError!")
finally:
    print("This always runs!")
```

---

# Custom Exception Class

A **custom exception class** is a user-defined class that inherits from the built-in `Exception` class and is used to represent application-specific error conditions. In Python, only classes that inherit from `BaseException` can be raised or caught as exceptions.

**Basic example:**

```python
class Exeption(Exception):
    ...

def set_age(age):
    if age < 0:
        raise Exeption("Age cannot be negative")
    print(f"Age set to {age}")

try:
    set_age(-5)
except Exeption as e:
    print("Error:", e)
```

---

# Exception Inheritance in Python

Exception classes fully support inheritance. All exceptions ultimately inherit from `BaseException`, with most application-level exceptions inheriting from `Exception`. Custom exceptions can inherit from other exceptions, creating a logical hierarchy.

**Example — GardenError hierarchy:**

```python
class GardenError(Exception):
    pass

class PlantError(GardenError):
    pass

class WaterError(GardenError):
    pass

errors = [PlantError(), WaterError()]

for error in errors:
    try:
        raise error
    except GardenError:
        print("Caught a garden error")
```

---

# Custom Exception Classes and Messages

Using `super().__init__(message)` within the constructor calls the parent class's constructor, ensuring the message is properly passed up the inheritance chain.

```python
## --- Custom Exception Classes ---
class GardenError(Exception):
    pass

class PlantError(GardenError):
    def __init__(self, message="There is a problem with a plant!"):
        super().__init__(message)

def check_plant():
    raise PlantError("The tomato plant is wilting!")

try:
    check_plant()
except PlantError as e:
    print(f"Caught PlantError: {e}")

try:  # Catching all garden-related errors
    check_plant()
except GardenError as e:
    print(f"Caught a garden error: {e}")
```

`super().__init__(message)` means:

> "Call the `__init__` method of the parent class of `PlantError` and pass `message` to it."

The parent here is Exception through GardenError.

## What `super()` Actually Returns

`super()` returns a **proxy object** that lets you access methods of the **next class in the Method Resolution Order (MRO)**.

For your classes, the MRO looks like this:

```text
PlantError
   ↓
GardenError
   ↓
Exception
   ↓
BaseException
   ↓
object
```

So when Python sees:

```python
super().__init__(message)
```

it interprets it as:

```python
GardenError.__init__(self, message)
```

But since `GardenError` doesn't define `__init__`, Python continues up the chain until it finds one in `Exception`.

## Why It Looks Like "Sending a Message"

It feels like sending a message because the call is **forwarded up the inheritance chain**.

Conceptually:

```text
PlantError.__init__()
      │
      ▼
super() → ask next class in MRO
      │
      ▼
Exception.__init__(message)
```

So instead of directly naming the parent class, `super()` says:

> "Let the next class in the hierarchy handle this."

When the message reaches the base exception classes, it is **stored inside the exception object** so Python can later display or retrieve it.

Your call:

```python
super().__init__(message)
```

eventually reaches the constructor of Exception (which inherits from BaseException).

## What the Base Exception Does With the Message

Internally, the constructor stores the message inside the attribute **`args`**.

Conceptually it behaves like this:

```python
class BaseException:
    def __init__(self, *args):
        self.args = args
```

So when your code runs:

```python
raise PlantError("The tomato plant is wilting!")
```

the base exception receives:

```text
args = ("The tomato plant is wilting!",)
```

and stores it inside the exception object.

## Where the Message Lives

After creation, the exception object looks roughly like this:

```text
PlantError instance
├── args = ("The tomato plant is wilting!",)
└── type = PlantError
```

## Why `print(e)` Shows the Message

When you do:

```python
except PlantError as e:
    print(e)
```

Python calls the exception's `__str__()` method (defined in BaseException).

That method returns a string based on `args`.

Conceptually:

```python
def __str__(self):
    return str(self.args[0])
```

So Python prints:

```
The tomato plant is wilting!
```

## You Can Access It Directly

Example:

```python
try:
    raise PlantError("The tomato plant is wilting!")
except PlantError as e:
    print(e.args)
```

Output:

```
('The tomato plant is wilting!',)
```

## Full Flow in Your Program

```
raise PlantError("The tomato plant is wilting!")
        │
        ▼
PlantError.__init__(message)
        │
        ▼
super().__init__(message)
        │
        ▼
Exception.__init__(message)
        │
        ▼
BaseException.__init__(message)
        │
        ▼
self.args = ("The tomato plant is wilting!",)
```

Later:

```
print(e)
   │
   ▼
BaseException.__str__()
   │
   ▼
returns "The tomato plant is wilting!"
```

 **Summary**

The base exception doesn't "handle" the message in a complex way. It simply:

1. **stores it in `args`**
2. **returns it when the exception is printed**

---

# Comprehensions & Generators

In Python there are **two ways to create generators**, and they differ mainly by **syntax**, not by concept. Both produce **lazy iterators** that generate values on demand.

  

These generators are part of the iteration system defined in the Python iterator protocol.

## 1. Generator Expressions (Comprehension-style)

  

These look like **list comprehensions but with parentheses**.

  

Example:

  

```python
gen = (x * x for x in range(5))
```

  

This creates a **generator object** that produces values one by one.

  

Equivalent step-by-step use:

  

```python

print(next(gen)) # 0

print(next(gen)) # 1

print(next(gen)) # 4

```

  

Key points:

  

* Uses **parentheses `( )`**

* Looks like a comprehension

* No `yield`

* Produces values lazily

  

Comparison:

  

```python

[x*x for x in range(5)] # list comprehension → builds full list

(x*x for x in range(5)) # generator expression → lazy generator

```

Memory difference:

```text

List comprehension → stores all values

Generator expression → produces values when needed

```



## 2. Generator Functions (`yield`)

  

These are **normal functions that use `yield`**.

  

Example:

  

```python
def squares(n):
	for i in range(n):
		yield i * i
```

  

Calling the function **does not run it immediately**:

  

```python
gen = squares(5)
```

  

Instead it returns a **generator object**.

  

Execution happens when iterated:

  

```python
for x in gen:
	print(x)
```

  

Key mechanism:

  

* `yield` **pauses the function**

* State is preserved

* Execution resumes on the next `next()` call


## Execution Model of Generator Functions

  

Normal function:

  

```text

call → run entire function → return result

```

  

Generator function:

  

```text

call → create generator object

next() → run until yield

next() → resume from yield

```


## Conceptual Equivalence

  

These two are roughly equivalent:

  

Generator expression:

  

```python
gen = (x*x for x in range(5))
```

  

Generator function:

  

```python
def gen():
	for x in range(5):
yield x*x
```

  

Both produce a **generator object**.

## Summary

  

| Feature    | Generator Expression | Generator Function |
| ---------- | -------------------- | ------------------ |
| Syntax     | `(x for x in ...)`    | `def ... yield`    |
| Complexity | Simple cases         | Complex logic      |
| State      | Automatic            | Explicit control   |
| Code size  | Short                | Longer             |
 **Key idea**

  

Both produce **generator objects** implementing the iterator protocol:

  

```text
__iter__()
__next__()
```

  

The difference is **syntax and complexity of the logic**.

  

---

**Generator vs normal function:**

```python
## Generator function example
def generate_numbers():
    print("Generator starting")
    for i in range(3):
        print(f"Producing {i}")
        yield i  # pauses here and produces value only when requested

## Create a generator object (the function will not execute)
gen_obj = generate_numbers()
print("Generator created (nothing produced yet)\n")

## Normal function example
def normal_numbers():
    print("Normal function starting")
    for i in range(3):
        print(f"Producing {i}")
        return i  # returns immediately, loop stops after first iteration

## Call normal function
normal_result = normal_numbers()
```

**Output:**
```
Generator created (nothing produced yet)

Normal function starting
Producing 0
```

---

**List comprehension vs generator expression:**

```python
## List comprehension (eager)
numbers = range(5)

## This builds the full list immediately
squares_list = [n * n for n in numbers]
print("List comprehension (all values computed at once):")
print(squares_list)

## Generator expression (lazy)
## This creates a generator object; values are computed only when requested
squares_gen = (n * n for n in numbers)
print("\nGenerator expression (values computed lazily):")
print(squares_gen)  # just shows generator object

## Iterate to get values one by one
print("Iterating generator:")
for val in squares_gen:
    print(val)
```

**Output:**
```
List comprehension (all values computed at once):
[0, 1, 4, 9, 16]

Generator expression (values computed lazily):
<generator object <genexpr> at 0x7f7a53feeb50>
Iterating generator:
0
1
4
9
16
```

---

**`lazy_squares` generator:**

```python
## Generator function that produces squares
def lazy_squares(n):
    for i in range(n):
        print(f"Computing square of {i}")
        yield i * i  # value is produced only when requested

## Create generator object
squares = lazy_squares(5)

print("Generator created")  # Nothing has been computed yet

## Request the first value
print(next(squares))

## Request the next value
print(next(squares))

## Iterate over the remaining values
for val in squares:
    print(val)
```

**Output:**
```
Generator created
Computing square of 0
0
Computing square of 1
1
Computing square of 2
4
Computing square of 3
9
Computing square of 4
16
```

---

**`counter` generator — state and StopIteration:**

```python
def counter():
    print("Start")
    yield 1
    print("Middle")
    yield 2
    print("End")
    return 99

try:
    g = counter()
    print(g)
    print(next(g))
    print(g)
    print(next(g))
    print(g)
    print(next(g))
except StopIteration:
    print("Generator finished")
```

**Output:**
```
<generator object counter at 0x776f10a402e0>
Start
1
<generator object counter at 0x776f10a402e0>
Middle
2
<generator object counter at 0x776f10a402e0>
End
Generator finished
```

A generator function returns a **single generator object** that remains the same throughout its lifetime. It does not store the values it yields. The generator preserves only its execution state: the current instruction pointer, the function's local variables, and its call stack frame.

---

**For loop vs manual `next()`:**

```python
def numbers():
    print("Generator started")
    for i in range(1, 3):
        print(f"Yielding {i}")
        yield i
    print("Generator finished")


print("=== Using for loop ===")
for n in numbers():
    print(f"Received: {n}")


print("\n=== Using manual next() ===")
gen = numbers()  # create generator once

try:
    for _ in range(10):
        value = next(gen)
        print(f"Received: {value}")
except StopIteration:
    print("Generator exhausted")
```

**Output:**
```
=== Using for loop ===
Generator started
Yielding 1
Received: 1
Yielding 2
Received: 2
Generator finished

=== Using manual next() ===
Generator started
Yielding 1
Received: 1
Yielding 2
Received: 2
Generator finished
Generator exhausted
```

---
# The `next()` Function
In Python, `next()` is a **built-in function** used to **retrieve the next item from an iterator**. It is the standard way to move through any object that implements the **iterator protocol**.


## 1. Syntax

```python id="fr5p38"
next(iterator[, default])
```

* `iterator` → an **iterator object**, like a generator or anything with `__next__()`.
* `default` → optional value returned if the iterator is exhausted (instead of raising `StopIteration`).


## 2. What It Does

* Calls the iterator's `__next__()` method internally.
* Returns the **next value** in the sequence.
* If there are no more values:

  * Without `default` → raises `StopIteration`
  * With `default` → returns the `default` value


## 3. Example With a Generator Expression

```python id="1w9e8p"
gen = (x*x for x in range(2))

print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # StopIteration
```

Here, `next()` **pauses the generator, resumes it, and returns the next value**. When no more values exist, it raises `StopIteration`.



## 4. Using Default to Avoid StopIteration and Using `next()` with a List

next() works with any iterator, not just generators. In Python, lists, tuples, dictionaries, and sets can all provide iterators.

```python id="eqm6sy"
gen = iter([1, 2])
print(next(gen, "end"))  # 1
print(next(gen, "end"))  # 2
print(next(gen, "end"))  # "end"
```
Explanation:

* `iter(numbers)` creates an iterator object.
* `next(it)` retrieves elements one by one from that iterator.
* Lists themselves are **iterable**, but not iterators — that's why we call `iter()` first.


## 5. Behind the Scenes

`next()` essentially does:

```python id="7rnwnu"
iterator.__next__()
```

For a generator, this means:

1. Resume execution from the last `yield`
2. Run until the next `yield` (or end of function)
3. Return the yielded value

For an Iterator (like list, tuple, dict, etc.) Python internally does:

1. **Look at the current position in the iterator**
   Every iterator keeps track of **where it is** in the sequence of elements.

2. **Retrieve the next element** from the underlying iterable

   * For a list, it's the next index in the list.
   * For a dictionary, it's the next key (or value/items if that iterator).
   * For a file, it's the next line.

3. **Advance the internal state** of the iterator so the next call to `next()` will get the following element.

4. **Return that element**.

5. **Raise `StopIteration`** if there are no more elements.

---

# The `iter()` Function

The `iter()` function creates an iterator from an iterable object (list, tuple, string, etc.). An iterator keeps track of its current position and can produce the next item using `next()`.

While generators are already iterators and do not require `iter()`, standard iterables cannot be advanced with `next()` unless first converted using `iter()`.

`iter()` has two forms:
- `iter(obj)` — calls `obj.__iter__()` to produce an iterator
- `iter(callable, sentinel)` — repeatedly invokes a callable until a specified sentinel value is returned

**How Python Does This Internally**

When you call:

```python id="c5jkts"
it = iter(some_iterable)
```

Python roughly does:

```python id="9hgbky"
it = some_iterable.__iter__()
```

* The iterable's `__iter__()` method returns an **iterator object**.
* The iterator object has `__next__()` to give you the next value.

**Example — converting a list to an iterator:**

```python
## A normal list is iterable, but not an iterator
numbers = [10, 20, 30]

## Convert the list into an iterator
it = iter(numbers)

## Manually access elements using next()
print(next(it))  # 10
print(next(it))  # 20
print(next(it))  # 30

## The iterator is now exhausted
## next(it)  # Would raise StopIteration
```

**How `for` uses `iter()` internally:**

```python
## This:
for x in [1, 2, 3]:
    print(x)

## Is roughly equivalent to:
it = iter([1, 2, 3])
while True:
    try:
        x = next(it)
        print(x)
    except StopIteration:
        break
```

**`iter()` with sentinel form:**

```python
it = iter([1, 2, 3])

print(next(it))
print(next(it))
print(next(it))

def f():
    return input()

for line in iter(f, "quit"):
    print("You typed:", line)

## output:
    # 1
    # 2
    # 3
    # test
    # You typed: test
    # quit
```

---

# Command-Line Arguments

**Command-line arguments** are values provided to a Python program at the moment it is executed from the terminal, allowing the user to pass information without modifying the source code. In Python, these arguments are accessed through the `sys.argv` list.

- `sys.argv[0]` → always the program name
- `sys.argv[1:]` → the arguments supplied by the user

```python
## Example command:
## python3 script.py hello 42
## Inside the program:
## sys.argv = ["script.py", "hello", "42"]
```

**Example:**

```python
import sys

print("Program name:", sys.argv[0])

if len(sys.argv) == 1:
    print("No arguments provided")
else:
    i = 1
    while i < len(sys.argv):
        print(f"Arguments {i}: {sys.argv[i]}")
        i += 1
```

**Output:**
```
└─$ python file.py hello 42
Program name: file.py
Arguments 1: hello
Arguments 2: 42

└─$ python file.py
Program name: file.py
No arguments provided
```

---

# Packing & Unpacking

**Packing** collects multiple values into a single composite object. This occurs in function definitions using `*args` and `**kwargs`:
- `*args` → gathers all extra positional arguments into a **tuple**
- `**kwargs` → gathers extra keyword arguments into a **dictionary**

**Unpacking** is the expansion of a composite object into individual elements:
- `*` for iterables
- `**` for mappings

Conceptually, `a, b = iterable` is roughly equivalent to:
```python
it = iter(iterable)
a = next(it)
b = next(it)
```

**Summary:**

```python
## Packing — Positional (*args):
def func(*args):
    print(args)

## Packing — Keyword (**kwargs):
def func(**kwargs):
    print(kwargs)

## Unpacking in Assignment:
a, b, c = [1, 2, 3]

## Extended Unpacking:
a, *middle, c = [1, 2, 3, 4, 5]
all_unique = set().union(*player_sets.values())
```

---

# `isinstance()`

`isinstance()` is a built-in Python function used for **runtime type checking**. It determines whether an object is an instance of a specified class (or a subclass thereof).

**Syntax:** `isinstance(object, classinfo)`
- `object` — the value you want to check
- `classinfo` — a type, class, or a tuple of types
- Returns `True` if the object is an instance of the given type (or its subclass)

```python
x = 10
print(isinstance(x, int))          # True
print(isinstance(x, str))          # False

x = 3.14
print(isinstance(x, (int, float))) # True

class Animal:
    pass

class Dog(Animal):
    pass

d = Dog()

print(isinstance(d, Dog))          # True
print(isinstance(d, Animal))       # True (because Dog inherits from Animal)
```

---

# `all()`

The `all()` function evaluates whether **every element** in an iterable is truthy. It returns `True` if all elements are considered true in a boolean context, and `False` if at least one element is falsy. Falsy values include `False`, `None`, `0`, empty sequences like `[]` or `""`, and empty collections like `{}` or `set()`. If the iterable is empty, `all()` returns `True` by default (vacuous truth).

`all()` has **short-circuit behavior** — evaluation stops as soon as a falsy value is encountered.

```python
print(all([True, True, True]))        # True
print(all([True, False, True]))       # False

nums = [2, 4, 6, 8]
print(all(n % 2 == 0 for n in nums)) # True

def check(x):
    print(f"Checking {x}")
    return x > 0

print(all(check(n) for n in [1, 2, -3, 4]))
```

---

# `.strip()`

The `.strip()` method removes unwanted characters from the **beginning and end** of a string. By default, it strips all types of whitespace (spaces, tabs, newlines, carriage returns, vertical tabs, form feeds). It can also accept a custom string of characters to remove.

```python
## Example 1: Default stripping (whitespace)
user_input = "   Alice \n"
clean_input = user_input.strip()
print(repr(clean_input))  # Output: 'Alice'

## Example 2: Removing specific characters
filename = ">>>report.txt<<<"
clean_filename = filename.strip("><")
print(clean_filename)     # Output: 'report.txt'
```

---

# `sys.stdin`, `sys.stdout`, `sys.stderr`

Special file-like objects from the `sys` module that handle input and output at the system level.

- **`sys.stdin`** — Standard input (usually keyboard). Read data from the user through it.
- **`sys.stdout`** — Standard output (usually the console). `print()` actually writes to `sys.stdout`.
- **`sys.stderr`** — Standard error. Used for error messages or logs. Can be redirected separately from `sys.stdout`.

`readline()` reads a single line from input; `read()` reads all remaining data from the input stream.

**`flush()`** forces any buffered data to be written immediately instead of waiting. Without `flush()`, "Processing..." might appear only after the sleep or after a newline is printed.

**Example:**

```python
import sys

## 1. sys.stdin → reading input
print("Enter your name:", end=' ')
name = sys.stdin.readline().strip()  # reads one line from standard input
print(f"Hello, {name}!\n")

## 2. sys.stdout → writing output
sys.stdout.write("This is written directly to standard output.\n")

## 3. sys.stderr → writing error messages
sys.stderr.write("Warning: This is an error message!\n")
```

**Using flush:**

```python
import sys
import time

sys.stdout.write("Processing...")
sys.stdout.flush()  # forces it to show
time.sleep(2)       # simulate a delay
print("Done")
```

---

# Package

A **package** is a directory that organizes related modules into a structured namespace, allowing large programs to be divided into logical, reusable components. Packages group multiple modules and subpackages together so they can be imported using hierarchical names.

A package typically contains an `__init__.py` file — the **package initializer**. It is executed automatically when the package is imported, allowing Python to recognize the folder as an importable package and to perform setup actions.

**Main roles of `__init__.py`:**
- Control the package's public interface (decides which functions, classes, or variables are exposed)
- Run initialization code
- Define metadata (e.g., `version`, `author`)

---

### How `__init__.py` Controls What the Package Exposes

**`printer.py`:**

```python
def print_hello():
    print("Hello!")

def print_secret():
    print("This is a secret message!")
```

**`__init__.py`:**

```python
from .printer import print_hello
```

**`main.py`:**

```python
import mypackage

mypackage.print_hello()    # works
mypackage.print_secret()   # ERROR
```

`__init__.py` does **not** hide functions completely; it simply controls what is conveniently exposed through the package namespace. Direct module access remains possible (e.g., `mypackage.printer.print_secret()`).

---

### What Happens During Import

When Python sees `import alchemy`, it:
1. Locates folder `alchemy`
2. Executes `alchemy/__init__.py`
3. Creates package object
4. Exposes names defined there

---

### Namespace and API

**Namespace:** A container that holds names (identifiers) and maps them to objects.

```python
import mymodule
mymodule.foo()    # Python looks in the mymodule namespace
print(mymodule.bar)
```

**API (Application Programming Interface):** The set of functions, classes, and variables a module or package exposes for others to use.

```python
## mypackage/__init__.py
from .printer import print_hello
## print_hello() is part of the package's API
## print_secret() in printer.py is internal, not part of the API
```

---

### Sacred Scroll

In the subject, the **"Sacred Scroll"** is a fancy name for `__init__.py`:
- Transforms a folder into a Python package
- Controls the public API
- Can contain metadata (author, version)
- Decides which functions are exposed and which stay hidden

---

### Alchemy Package Example

**`elements.py`:**

```python
def create_fire():
    return "Fire element created"

def create_water():
    return "Water element created"

def create_earth():
    return "Earth element created"

def create_air():
    return "Air element created"
```

**`__init__.py`:**

```python
from .elements import create_fire, create_water
```

**`ft_sacred_scroll.py`:**

```python
import alchemy

## Accessible because they were exposed in __init__.py
alchemy.create_fire()
alchemy.create_water()

## Not accessible at package level
## alchemy.secret_spell()  -> AttributeError

## But direct module access still works
alchemy.elements.secret_spell()
```

---

# Import Styles

### 1️⃣ Basic `import module`

Imports the whole module. Access functions with the module name as a prefix.

```python
import math
print(math.sqrt(16))  # Access function using module name
```

✅ Avoids name conflicts, clear which module a function comes from.
❌ Must type module name every time.

---

### 2️⃣ `import module as alias`

Imports the whole module, but gives it a shorter or custom name.

```python
import math as m
print(m.sqrt(16))
```

✅ Shorter, convenient for long module names.

---

### 3️⃣ `from module import name`

Imports specific functions, classes, or variables from a module.

```python
from math import sqrt
print(sqrt(16))  # Use directly, no module prefix
```

✅ Cleaner code (no prefix), only imports what you need.
❌ Can cause name conflicts if different modules have the same function names.

---

### 4️⃣ `from module import name as alias`

Imports a specific function or variable with a custom name.

```python
from math import sqrt as square_root
print(square_root(16))
```

✅ Avoids conflicts, can give meaningful names.

---

### 5️⃣ `from module import *`

Imports everything from the module into the current namespace.

```python
from math import *
print(sqrt(16))   # sqrt is directly accessible
print(pi)         # pi is directly accessible
```

❌ Can overwrite existing names, hard to track where names come from. Not recommended in production code.

---

### 6️⃣ Importing Submodules in Packages

```python
## package/
## └── subpackage/
##     └── module.py

import package.subpackage.module
package.subpackage.module.func()

## Or, using from:
from package.subpackage.module import func
func()
```

---

# Circular Dependency

A **circular dependency** happens when two or more modules try to import each other, creating a loop that Python can't resolve during loading.

```python
## module a.py
from b import func_b

def func_a():
    print("Function A")
    func_b()
```

```python
## module b.py
from a import func_a

def func_b():
    print("Function B")
    func_a()
```

If you try to `import a` or `import b`, Python gets stuck because:
- `a.py` imports `b.py`
- `b.py` imports `a.py`
- Python is still trying to finish loading `a.py` → `func_a` doesn't exist yet
- Crash or `ImportError` occurs

---

### How to Fix Circular Dependencies

**1️⃣ Late Import (Recommended)**

Import the module inside a function instead of at the top.

```python
## spellbook.py
def record_spell(spell_name, ingredients):
    from validator import validate_ingredients  # imported here
    result = validate_ingredients(ingredients)
    return f"Spell recorded: {spell_name} ({result})"
```

Python doesn't import `validator` until the function runs → no circular problem.

**2️⃣ Dependency Injection**

Pass the function or object from one module to another instead of importing.

```python
## spellbook.py
def record_spell(spell_name, ingredients, validator_func):
    result = validator_func(ingredients)
    return f"Spell recorded: {spell_name} ({result})"
```

Now `spellbook` doesn't need to import `validator`. The caller provides `validate_ingredients` as a parameter.

**3️⃣ Separate Shared Module**

Move shared functions to a new module that both modules can safely import.

```python
## utils.py
def validate_ingredients(ingredients):
    ...
```

Both `spellbook.py` and `validator.py` import `utils.py`. No circular import occurs.

---

# Absolute & Relative Imports

Both are ways to tell Python where to find a module or function.

---

### 1️⃣ Absolute Import

Specifies the full path from the top-level package.

```python
## alchemy/
##     __init__.py
##     elements.py
##     transmutation/
##         __init__.py
##         basic.py

## absolute import
from alchemy.elements import create_fire

def lead_to_gold():
    return f"Lead turned to gold using {create_fire()}"
```

✅ Clear and unambiguous, works no matter where the importing file is located.
❌ Can be long in deep hierarchies.

---

### 2️⃣ Relative Import

Specifies the path relative to the current module using dots:
- `.` → current package
- `..` → parent package
- `...` → grandparent, etc.

```python
## relative import
from .basic import lead_to_gold
from ..elements import create_water

def philosophers_stone():
    return f"Philosopher's stone made using {lead_to_gold()} and {create_water()}"
```

✅ Shorter paths in deeply nested packages, easier to reorganize packages internally.
❌ Can be confusing if you move files around, only works inside a package (not from top-level scripts).

---

### How Relative Imports Work

When a module is imported, Python automatically sets `__package__` to store the full package path of that module. For example, if `advanced.py` is imported as `alchemy.transmutation.advanced`, Python sets `__package__ = "alchemy.transmutation"` inside that file.

When `from .basic import lead_to_gold` is used, the `.` tells Python to start from the current package defined in `__package__`. Python appends `basic`, producing the full path `alchemy.transmutation.basic`, and directly loads `alchemy/transmutation/basic.py`.

---

### Parent Relative Import (`from ..basic import ...`)

```
alchemy/
├── elements.py
├── potions.py
└── transmutation/
    ├── basic.py         # <-- basic is here
    └── potions/
        └── codes/
            └── advanced.py
```

In `advanced.py`, `from ..basic import lead_to_gold` uses `..` to move one package level up from the current module (`alchemy.transmutation.potions.codes`), navigating up to `alchemy.transmutation.potions`, then looking for `basic` relative to that level.

---

# Protocol

In Python, a **Protocol** is a type-hinting mechanism that defines a set of methods or behaviors an object must implement so Python knows how to interact with it. It is essentially an interface defined by **behavior**, not by inheritance. This idea is called **duck typing**.

Protocols do **not** enforce anything at runtime — Python does not inject methods or check types while the program runs. Instead, Protocols are used by static type checkers like `mypy` to verify that objects you pass to functions match the expected "shape."

**Benefits:**
- **Static analysis:** Tools like `mypy` check types before running the code.
- **Clear documentation:** A type hint like `obj: Reader` clearly communicates that `obj` must have a `.read()` method that returns a string.
- **IDE support:** Autocomplete can suggest `.read()` on `obj`, reducing guesswork.
- **Flexible duck typing:** Any object with the required methods satisfies the Protocol — no inheritance needed.

**Example:**

```python
from typing import Protocol

## 1 Define a Protocol
class Reader(Protocol):
    def read(self) -> str:
        ...
    def tt(self) -> str:
        ...

## 2 Classes remain the same
class FileReader:
    def read(self) -> str:
        return "file data"

class APIReader:
    def read(self) -> str:
        return "API data"

## 3 Function uses the Protocol in type hint
def process(obj: Reader) -> None:
    data = obj.read()
    print(data)

process(FileReader())  # Works
process(APIReader())   # Works

process(123)           # ❌ Runtime error: int has no read()
```

**Output:**
```
└─$ python3 three.py
file data
API data
Traceback (most recent call last):
  File "three.py", line 26, in <module>
    process(123)
AttributeError: 'int' object has no attribute 'read'. Did you mean: 'real'?

└─$ mypy three.py
three.py:26: error: Argument 1 to "process" has incompatible type "int"; expected "Reader"  [arg-type]
Found 1 error in 1 file (checked 1 source file)
```

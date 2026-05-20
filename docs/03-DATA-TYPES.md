# 03: Data Types

## Type System Overview

RASLang has **8 primitive data types** (defined in lexer.c):

| Type | Token | Range | Size | Default | Example |
|------|-------|-------|------|---------|---------|
| `int` | TOK_INT | -2^31 to 2^31-1 | 4 bytes | 0 | `int x = 42;` |
| `deci` | TOK_DECI | IEEE 754 double | 8 bytes | 0.0 | `deci pi = 3.14159;` |
| `char` | TOK_CHAR | ASCII 0-255 | 1 byte | 0 | `char c = 'A';` |
| `str` | TOK_STR | UTF-8 string | varies | "" | `str s = "hello";` |
| `bool` | TOK_BOOL | true/false | 1 byte | false | `bool b = true;` |
| `byte` | TOK_BYTE | 0-255 | 1 byte | 0 | `byte b = 255;` |
| `ubyte` | TOK_UBYTE | 0-255 | 1 byte | 0 | `ubyte ub = 127;` |
| `none` | TOK_NONE | void/null | 0 bytes | - | Return type only |

## Primitive Type Details

### int (Integer) - `TOK_INT`
32-bit signed integer, standard C `int`

```ras
int x = 42;
int neg = -100;
int hex = 0xFF;           // 255
int octal = 0755;         // 493
int binary = 0b1010;      // 10
```

Operators:
- Arithmetic: `+`, `-`, `*`, `/`, `%`
- Bitwise: `&`, `|`, `^`, `~`, `<<`, `>>`
- Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`
- Logical: `&&`, `||`, `!`

### deci (Decimal/Float) - `TOK_DECI`
64-bit IEEE 754 double floating point

```ras
deci pi = 3.14159;
deci neg = -2.71828;
deci small = 0.00001;
deci science = 1.23e-4;     // Scientific notation? Check lexer
```

Operators: Same as int (arithmetic, comparison, logical)

### char (Character) - `TOK_CHAR`
Single ASCII character (8-bit)

```ras
char a = 'A';              // Letter
char digit = '5';
char space = ' ';
char newline = '\n';       // Escape sequence
char tab = '\t';
```

Escape sequences (from lexer):
- `\n` - newline
- `\t` - tab
- `\"` - double quote
- `\\` - backslash
- `\'` - single quote

#### Character-to-Integer Conversion
Character literals are automatically converted to ASCII int in parse_primary:
```ras
char c = 'A';              // Actually stores 65 (ASCII value)
int ascii = @ord['A'];     // Get ASCII int from char
char back = @chr[65];      // Get char from ASCII int
```

### str (String) - `TOK_STR`
UTF-8 encoded text string

```ras
str greeting = "Hello";
str multi = "Line 1\nLine 2";      // With escape sequences
str empty = "";
str interpolated = "Hello $name";
str complex = "Result: ${x + y}";
```

**String Operations:**
- Concatenation: `@concat[s1, s2]`
- Length: `@len[s]`
- Substring: `@substr[s, start, length]`
- Comparison: `@strcmp[s1, s2]`
- Conversion from int: `@type[42]::str`

**Interpolation (Strings Only):**
- Simple: `"Hello $name"` → outputs value of `name`
- Expression: `"Sum: ${a + b}"` → outputs result of `a + b`
- Expands to multiple `show` statements internally

### bool (Boolean) - `TOK_BOOL`
Logical true/false value

```ras
bool flag = true;
bool check = false;
bool result = x > 5;       // Result of comparison
bool logic = true && false; // Result of logical operation
```

Values:
- `true` - boolean true literal
- `false` - boolean false literal

Operators:
- Logical: `&&` (AND), `||` (OR), `!` (NOT), `xor` (XOR)
- Comparison: `==`, `!=`

### byte (Unsigned Byte) - `TOK_BYTE`
8-bit unsigned integer (0-255)

```ras
byte b = 255;
byte hex = 0xFF;
byte small = 0;
```

Use for:
- Memory operations (peek/poke)
- Binary data
- When you need small unsigned values

Difference from `ubyte`:
- `byte` and `ubyte` appear to be equivalent in parser
- Both are 8-bit unsigned
- Distinction unclear from source; use consistently

### ubyte (Unsigned Byte) - `TOK_UBYTE`
8-bit unsigned integer (0-255) - same as byte

```ras
ubyte b = 255;
ubyte b2 = 0x80;
```

### none (Void) - `TOK_NONE`
Special type for functions with no meaningful return value

```ras
fnc print_only[msg]::none {
    show[msg];
}
```

Functions returning `none`:
- Do not use `get[value];`
- Implicitly return without a value
- Cannot be used in expressions that need a value

## Composite Types

### Arrays
See [07-ARRAYS.md](07-ARRAYS.md) for details

**Declaration:**
```ras
arr{int, 10} numbers;         // Array of 10 ints
arr{str, 5} names;
arr{deci, 100} values;
```

### Maps
See [08-MAPS.md](08-MAPS.md) for details

**Declaration:**
```ras
map{str, int} ages;           // Map: str keys → int values
map{int, deci} lookup;        // Map: int keys → deci values
```

### Groups (Custom Types)
See [09-GROUPS.md](09-GROUPS.md) for details

**Definition:**
```ras
group Point {
    int x;
    int y;
}
```

**Usage:**
```ras
Point p;                      // Declare variable of custom type
p.x = 10;
p.y = 20;
```

## Type Conversions

### Implicit (Automatic)

The parser doesn't show implicit conversion rules clearly, so assume **NO implicit conversions** — use explicit conversion with `@type` builtin.

### Explicit Conversion

**Builtin Function: `@type`** (from builtins.c, category BUILTIN_CAT_CONVERSION)

```
@type[value]::target_type
@type[value, additional_args]::type1,type2
```

Examples:
```ras
int x = 42;
deci f = @type[x]::deci;      // int → deci
str s = @type[x]::str;         // int → str
int i = @type[3.14]::int;      // deci → int (truncates)
bool b = @type[1]::bool;       // int → bool
```

**Deprecated Conversion Functions** (still in BUILTIN_REGISTRY):
- `@to_int[x]`
- `@to_deci[x]`
- `@to_byte[x]`
- `@to_bool[x]`
- `@to_str[x]`

Recommendation: Use `@type[x]::targettype` instead.

### Character-Integer Conversion

**`@ord` builtin:** Character → Integer (ASCII value)
```ras
char c = 'A';
int ascii = @ord[c];           // Returns 65
```

**`@chr` builtin:** Integer → Character
```ras
int ascii = 65;
char c = @chr[ascii];          // Returns 'A'
```

## Type Checking and Inference

RASLang is **statically typed** — all variables must have explicit types.

```ras
int x = 5;                     // ✓ Type explicit
x = 10;                        // ✓ Matches declared type
x = "string";                  // ✗ Type mismatch (should error)

str s = @type[42]::str;        // ✓ Explicit conversion
```

**No type inference** for variable declarations:
```ras
x = 5;                         // ✗ Error: x not declared
int x = y + z;                 // ✗ Error if y,z not declared before use
```

## Special Values

### Default Values

When a variable is declared without initialization, it gets a default:

```ras
int x;                         // x = 0
deci d;                        // d = 0.0
char c;                        // c = '\0' (null char, int 0)
str s;                         // s = "" (empty string)
bool b;                        // b = false
byte by;                       // by = 0
```

### Null and Undefined

- **Null pointers**: Addresses are just `int` values, no special null type
- **Undefined**: Variables automatically zero-initialized if not set

## Size Information

**`@sizeof` builtin** — Get type size in bytes

```ras
int int_size = @sizeof[int];      // Returns 4
int deci_size = @sizeof[deci];    // Returns 8
int byte_size = @sizeof[byte];    // Returns 1
int str_size = @sizeof[str];      // Returns ??? (dynamic)
```

## Related Documentation
- [04-VARIABLES.md](04-VARIABLES.md) — Variable declaration and scope
- [06-OPERATORS.md](06-OPERATORS.md) — Type-specific operations
- [16-BUILTINS.md](16-BUILTINS.md) — All conversion functions

---
name: magik
description: This skill should be used when the user is writing, reviewing, or asking questions about Magik code — the programming language used in GE Smallworld GIS. Use this skill when the user mentions Magik, Smallworld, exemplars, def_slotted_exemplar, _method, _pragma, sw:rope, sw:property_list, or any Smallworld-specific constructs. Also use it when the user asks about GIS application development, Smallworld modules, or Java interop in a Magik context.
---


## What Is Magik?
Magik is a dynamically typed, object-oriented programming language built for Smallworld GIS (now GE Smallworld Geo Network Management). It compiles to Java bytecode and runs on the JVM. It is strictly OO and imperative — similar in spirit to Smalltalk — and is used for enterprise GIS applications in utilities and telecoms.

---

## Core Syntax

### Comments
```magik
# This is a single-line comment
## This is a method documentation comment (used by tooling)
```

### Assignment
```magik
a << 1.234         # "a becomes 1.234"
b +<< a            # b << b + a  (compound assignment)
c << "foo" + "bar" # string concatenation
```
**Never use `=` for assignment.** `=` is equality comparison only.

### Output
```magik
write("Hello, world!")      # prints with newline
show(some_variable)         # inspect/debug print
```

### Data Types
```magik
# Integer
x << 42

# Float
y << 3.14

# String
name << "Alice"

# Symbol (unique token, like an interned string — used heavily for identifiers)
sym << :my_symbol
escaped_sym << :|hello world|

# Boolean — NOTE: _true and _false, NOT true/false
flag << _true
flag << _false

# Null equivalent
nothing << _unset    # like null/nil in other languages

# Character literal
ch << %A             # the character 'A'

# Simple vector (array literal)
v << {1, 2, 3}

# Property list (ordered key-value, preserves insertion order)
pl << sw:property_list.new_with(:key1, "val1", :key2, "val2")

# Hash table (unordered)
ht << sw:hash_table.new()

# Concurrent hash map (preferred in Smallworld 5 — thread-safe and fastest)
chm << sw:concurrent_hash_map.new()
```

### Boolean Keywords
Magik uses keyword booleans — NEVER use bare `true` / `false`:
```magik
_true
_false
_maybe    # tri-state Kleenean: _true, _maybe, or _false
```

Methods returning a Kleenean result are named with `??` suffix (e.g. `inside??()`). Do not use the result directly as a boolean condition — compare explicitly with `_is _true` / `_is _false` / `_is _maybe`.

Logical operators:
```magik
_and   _or   _not   _xor
```

---

## Variables

Local variables must be declared with `_local`:
```magik
_local my_count << 0
_local result    << _unset
```

Global variables use `_global` (avoid in production — causes namespace pollution):
```magik
_global my_global << "some value"
```

Dynamic (thread-local) variables:
```magik
_dynamic !my_dynamic!
```

**Convention:** use `snake_case` with descriptive nouns. No camelCase. No type prefixes.

---

## Naming Conventions

### Methods and Procedures

- **Consistency:** The same function should have the same name across classes; different functions should have different names.
- **Get/set pairs:** If an attribute is read via `foo`, its setter must be named `foo<<`.
- **Boolean-returning routines** end in `?` and can be used directly as conditions:
  ```magik
  _if i.odd? _then ... _endif
  ```
- **Kleenean-returning routines** (those that may return `_true`, `_false`, or `_maybe`) end in `??`. Compare explicitly rather than using as a bare condition:
  ```magik
  # inside?? returns _true if strictly inside, _false if any part outside, _maybe if on edge
  _if rect.inside??(other_rect) _is _true _then ... _endif
  ```
- **Friend / internal-public methods** that cannot be strictly private should contain `!` within their names (e.g. `int!method()`). This makes them easy to identify and allows special treatment by method browsers. All such methods must use `classify_level=restricted`.
- **Slot-like methods** (no side effects, always returns the same single result on an unmodified object) must be defined **without** brackets. Methods whose primary purpose is not slot-like access **must** include brackets. Note that brackets are part of the method name — `sqrt` and `sqrt()` are entirely different methods:
  ```magik
  # Slot-like — no brackets (behaves like reading a property):
  my_obj.name
  my_obj.size

  # Not slot-like — with brackets (does work, has side effects, or returns multiple values):
  my_obj.calculate()
  my_obj.open_stream()
  ```

### Arguments

- Argument names should indicate role or type.
- Use an indefinite article prefix for names indicating type: `a_coordinate`, `a_rope`, `an_integer`.
- If multiple arguments have the same type, append numbers: `new_with_corners(coord1, coord2)`.

### Dynamic Variables

- Dynamic variable names must **start and end with `!`**: `!output!`, `!print_length!`.

### Class Design

- **Single initialisation method:** Define only one `init()` method per class. Multiple initialisers make subclassing much harder, as every subclass must find and override each one.
- **Link related classes via shared constants** rather than accessing globals — this improves efficiency and simplifies subclassing:
  ```magik
  my_object.define_shared_constant(:related_class, other_exemplar, :public)
  $
  ```

---

## Conditionals

```magik
_if condition
_then
    # ...
_endif

_if x > 0
_then
    write("positive")
_elif x = 0
_then
    write("zero")
_else
    write("negative")
_endif
```

**Note:** Use `_is` for identity (same object reference), `=` for value equality:
```magik
_if a _is _unset _then write("a is null") _endif
_if a = b _then write("a equals b") _endif
```

---

## Loops

```magik
# While loop
_local i << 0
_while i < 10
_loop
    i +<< 1
_endloop

# For-over loop (iterate a collection)
_for item _over my_collection.elements()
_loop
    write(item)
_endloop

# For-over with index and value
_for k, v _over my_property_list.fast_keys_and_elements()
_loop
    write(k, " => ", v)
_endloop

# General loop with _leave to break, _continue to skip
_loop
    _if some_condition _then _leave _endif
    _if skip_condition _then _continue _endif
_endloop
```

---

## Procedures (Standalone Functions)

Procedures are first-class objects, assigned to variables:
```magik
my_procedure << _proc @my_procedure(a, b, c)
    _return a + b + c
_endproc

x << my_procedure(1, 2, 3)   # x = 6
```

Use `>>` as a short form of `_return`:
```magik
my_proc << _proc(x)
    >> x * 2
_endproc
```

---

## Exemplars (Classes)

Magik does **not** have classes. It uses **exemplars** — a prototype-based system where new instances are clones of the exemplar.

### Define an Exemplar
```magik
def_slotted_exemplar(
    :my_object,
    {
        {:slot_a, _unset},
        {:slot_b, "default_value"}
    },
    {:parent_exemplar_a, :parent_exemplar_b}  # inheritance list
)
$
```

### Slot Accessors

Always define slot accessors using `slotted_format_mixin.define_slot_access()` rather than writing accessor methods by hand. Slots must **only** be accessed directly (`.slot_name`) inside `init()` and the generated accessor methods — all other code must go through the accessors.

```magik
my_object.define_slot_access(:slot_a, :writable, :public)
my_object.define_slot_access(:slot_b, :readable, :public)
$
```

### Constructor Pattern (new/init)
```magik
_method my_object.new(val_a, val_b)
    ## Creates a new MY_OBJECT with VAL_A and VAL_B.
    ## Returns a new my_object instance.
    >> _clone.init(val_a, val_b)
_endmethod
$

_private _method my_object.init(val_a, val_b)
    ##
    .slot_a << val_a
    .slot_b << val_b
    >> _self
_endmethod
$
```

- `_clone` creates a new instance
- `_self` refers to the current object
- `.slot_name` accesses a slot (field) via dot notation — only in `init()` and accessors
- `_super` calls the parent's implementation

### Shared Constants

Prefer `define_shared_constant` over defining a constant inside a method body:
```magik
my_object.define_shared_constant(:my_const, 42, :public)
$

# Use :private for internal constants:
my_object.define_shared_constant(:int!default_size, 16, :private)
$
```

### Pragma Annotations

All method definitions must have a `_pragma` immediately before them with a `classify_level`:
```magik
_pragma(classify_level=basic, topic={my_topic})
_method my_object.do_something(param)
    ## Does something useful with PARAM.
    ## Returns the result of the operation.
    _local result << .slot_a + param
    >> result
_endmethod
$
```

Valid classify levels: `basic`, `advanced`, `restricted`.

- Public API methods: `basic` or `advanced`
- Internal/non-public methods that are still publicly callable: `restricted`, and the method name **must** start with `int!`
- Private methods (`_private`): `restricted`

```magik
_pragma(classify_level=restricted)
_method my_object.int!internal_helper()
    ## Internal helper — not part of the public API.
    >> .slot_a * 2
_endmethod
$

_pragma(classify_level=restricted)
_private _method my_object.compute_value()
    ##
    >> .slot_b.size
_endmethod
$
```

Use `topic={code}` for methods that generate bytecode or perform code generation:
```magik
_pragma(classify_level=advanced, topic={code})
_method my_object.define_dynamic_method()
    ##
    # ...
_endmethod
$
```

### Method Documentation

Every `_method` definition must include a `##` docstring immediately after the method signature (even if empty). Documentation rules:
- Parameter names must be referenced **fully UPPERCASE** in the description
- Return value must be described as "Returns ..."

```magik
_pragma(classify_level=basic)
_method my_object.greet(_optional name)
    ## Greets the user by NAME.
    ## If NAME is not given, defaults to "World".
    ## Returns _unset.
    _if name _is _unset
    _then
        name << "World"
    _endif
    write("Hello, ", name)
_endmethod
$
```

### Private Methods
```magik
_pragma(classify_level=restricted)
_private _method my_object.internal_helper()
    ## Internal helper — not part of the public API.
    >> .slot_a * 2
_endmethod
$
```

### Optional Parameters
```magik
_pragma(classify_level=basic)
_method my_object.greet(_optional name)
    ## Greets the user by NAME (defaults to "World").
    _if name _is _unset
    _then
        name << "World"
    _endif
    write("Hello, ", name)
_endmethod
$
```

### Variadic Parameters
```magik
_pragma(classify_level=basic)
_method my_object.sum(_gather values)
    ## Returns the sum of all VALUES.
    _local total << 0
    _for v _over values.elements()
    _loop
        total +<< v
    _endloop
    >> total
_endmethod
$
```

---

## Mixins

Mixins add behaviour without data. They cannot be instantiated directly:
```magik
def_mixin(:my_mixin)
$

_pragma(classify_level=basic)
_method my_mixin.shared_behaviour()
    ## Demonstrates shared mixin behaviour.
    write("I am a mixin method")
_endmethod
$

# Use in an exemplar:
sw:def_slotted_exemplar(:my_class, {}, {:my_mixin})
$
```

---

## Iterator Methods

Custom iterators use `_iter` and `_loopbody`:
```magik
_pragma(classify_level=basic, topic={iter})
_iter _method my_object.even_elements()
    ## Yields each even element from _self.
    _for a _over _self.elements()
    _loop
        _if a.even? _is _true
        _then
            _loopbody(a)
        _endif
    _endloop
_endmethod
$
```

Usage:
```magik
_for val _over my_obj.even_elements()
_loop
    write(val)
_endloop
```

---

## Error Handling

Use `_protect` / `_protection` for cleanup (like `finally`):
```magik
_protect
    # code that may fail
    risky_operation()
_protection
    # always runs — cleanup here
    cleanup()
_endprotect
```

Use `_try` / `_when` to catch conditions:
```magik
_try
    risky_operation()
_when error
    write("An error occurred")
_endtry
```

Raise a condition:
```magik
sw:condition.raise(:my_error, :string, "Something went wrong")
```

**Always wrap production code in error handling.** End users must never see raw tracebacks.

### Resource Cleanup

Java closeable resources (streams, writers, etc.) must be closed unconditionally. Put the close/finish call in the `_protection` block, not as the last step of the try body — so it runs even when an exception is raised. An empty `_protection` block provides no cleanup and is a bug.

```magik
_local stream << some_java_resource.open()
_protect
    stream.write("data")
_protection
    stream.close()
_endprotect
```

---

## Collections Reference

| Collection | Ordered? | Thread-safe? | Notes |
|---|---|---|---|
| `simple_vector` | Yes | No | Fixed size array |
| `rope` | Yes | No | Dynamic list |
| `property_list` | Yes | No | Ordered key-value |
| `hash_table` | No | No | Unordered key-value |
| `concurrent_hash_map` | No | Yes | **Preferred in SW5** |
| `set` | No | No | Unique elements |
| `sorted_collection` | Yes (custom) | No | Sorted by order proc |

### Common Collection Methods (from `basic_collection_mixin`)

These are available on all standard collection types:

```magik
col.empty?                          # true if no elements (does NOT clear)
col.includes?(item)                 # true if item is in the collection
col.includes_all?(another)          # true if all elements of another are in col
col.an_element()                    # returns one arbitrary element, or _unset
col.elements()                      # iter: yields each element (safe copy)
col.fast_elements()                 # iter: fast, undefined if modified during loop
col.elements_satisfying(pred)       # iter: yields elements where pred.invoke(e) is true
col.select(pred)                    # returns new collection with matching elements
col.map(fn)                         # returns new collection with fn.invoke(e) applied
col.reduce(fn)                      # folds: fn.invoke(acc, e) over all elements
col.as_simple_vector()              # converts to simple_vector
col.as_sorted_collection(order)     # converts to sorted_collection
```

**Note:** `empty()` (no `?`) **clears** the collection. `empty?` (with `?`) **tests** if it is empty.

### rope (dynamic list)
```magik
r << sw:rope.new()
r.add_last("item1")
r.add_last("item2")
r.add_first("item0")
r.size                              # number of elements
r[1]                                # 1-origin indexed access
r.remove_first()                    # remove and return first element
r.remove_last()                     # remove and return last element
r.as_simple_vector()                # convert to simple_vector

_for item _over r.elements()
_loop
    write(item)
_endloop
```

### property_list (ordered key-value)
```magik
pl << sw:property_list.new()
pl << sw:property_list.new_with(:a, 1, :b, 2)

pl[:key] << "value"                 # store
pl[:key]                            # retrieve
pl.size                             # count
pl.empty?                           # test if empty
pl.empty()                          # clear all entries
pl.includes?(value)                 # test value membership
pl.key_of(value)                    # find key for a value
pl.lookup_key(key)                  # returns item, success_flag
pl.remove_key(key)                  # remove entry

_for k, v _over pl.keys_and_elements()
_loop
    write(k, " => ", v)
_endloop

_for k, v _over pl.fast_keys_and_elements()   # fast, don't modify during loop
_loop
    write(k, " => ", v)
_endloop
```

### hash_table / concurrent_hash_map
```magik
ht << sw:hash_table.new()
chm << sw:concurrent_hash_map.new()
chm << sw:concurrent_hash_map.new_with(:a, 1, :b, 2)

chm[:key] << "value"               # store
chm[:key]                          # retrieve
chm.size                           # count
chm.empty?                         # test if empty
chm.empty()                        # clear
chm.includes?(value)               # test value membership
chm.lookup_key(key)                # returns item, success_flag
chm.remove_key(key)                # remove entry
chm.put_if_absent(key, value)      # only insert if key not already present
chm.add_all(another)               # merge in all entries from another map

_for k, v _over chm.fast_keys_and_elements()
_loop
    write(k, " => ", v)
_endloop
```

### set
```magik
s << sw:set.new()
s.add(item)                        # add (no-op if already present)
s.remove(item)                     # returns success flag
s.includes?(item)
s.size
s.intersection(another)
s.difference(another)
s.symmetric_difference(another)
```

### sorted_collection
```magik
sc << sw:sorted_collection.new()
sc << sw:sorted_collection.new(_unset, _proc(a, b) >> a _cf b _endproc)
sc << sw:sorted_collection.new_from(any_collection)

sc.add(item)
sc.remove(item)
sc.includes?(item)
sc.size
sc.nth(n)                          # 1-origin access
sc.order_proc << new_proc          # re-sort with new order

_for item _over sc.elements()
_loop
    write(item)
_endloop
```

---

## String Operations

Prefer `sw:write_string(a, b, c)` over string concatenation (`a + b + c`) — it avoids intermediate allocations and works with any printable type:

```magik
sw:write_string("Hello, ", name, "!")
```

```magik
s << "hello world"
s.size                              # length
s.uppercase                         # "HELLO WORLD"
s.lowercase                         # "hello world"
s.titlecase                         # "Hello World"
s.trim_spaces                       # strip leading/trailing whitespace
s.split_by(%,)                      # split by comma character, returns rope
s.index_of_seq("world")            # find substring (1-origin), _unset if not found
s.substitute_string("hello", "goodbye")  # replace all occurrences
s.as_symbol()                       # convert to symbol
s.includes_seq?("world")           # test substring presence
```

---

## Pathname Handling

Use Smallworld pathname utilities for building file paths — never string concatenation:

```magik
# Build a path by descending into subdirectories:
sw:system.pathname_down(base_dir, "subdir", "file.txt")

# Build from a vector of components:
sw:system.pathname_from_components({"C:", "workspace", "myfile"})

# Appending a file extension is NOT path manipulation — sw:write_string is fine:
_local jar_name << sw:write_string(module_name, ".jar")
```

---

## British English Spellings

Magik's core libraries use British English. Always use:
- `initialise` (not `initialize`)
- `colour` (not `color`)
- `neighbour` (not `neighbor`)

---

## File Structure & Chunk Terminator

Each top-level statement or definition in a `.magik` file must end with `$` on its own line:
```magik
def_slotted_exemplar(:my_obj, {}, {})
$

_pragma(classify_level=basic)
_method my_obj.hello()
    ## Writes a greeting.
    write("hi")
_endmethod
$
```

In the REPL prompt, `$` submits the block. In files, it acts as a statement delimiter.

---

## Numeric Operations

### Integer
```magik
n << 42
n.shift(3)          # n * 2^3 (bit shift)
n.factorial()       # n!
n.power2()          # smallest power of 2 >= n
n.as_float()        # convert to float
n.as_character()    # e.g. 65 -> %A
```

### Float
```magik
f << 3.14
f.rounded()                         # nearest integer (banker's rounding at .5)
f.truncated()                       # toward zero
f.floor()                           # largest integer <= f
f.ceiling()                         # smallest integer >= f
f.as_rational()                     # exact rational representation
f.is_nan?()                         # IEEE NaN test
f.infinite?()                       # +/- infinity test
f.finite?()                         # finite test
float.pi()                          # Pi constant
float.infinity()                    # +Infinity
```

---

## Common Patterns

### Singleton Access
```magik
# Many system objects are singletons accessed directly:
gis_program_manager   # the main application manager
ds_environment        # database environment
smallworld_product    # product info
```

### Checking for Unset Before Use
```magik
_if .my_slot _isnt _unset
_then
    .my_slot.do_something()
_endif
```

### Method Chaining / Message Passing
```magik
result << my_obj.get_collection().elements().size
```

### Loading Code Dynamically
```magik
magik_rep.load_chunck(some_string.read_stream())
```

---

## Anti-Patterns to Avoid

- **Never use `=` for assignment** — it's comparison only; use `<<`
- **Never use `true`/`false`** — always `_true`/`_false`
- **Never use `null`/`nil`** — always `_unset`
- **Don't use camelCase** — use `snake_case` with underscores
- **Don't use type-prefix naming** (e.g., `strName`, `intCount`) — use descriptive names like `first_name`, `item_count`
- **Don't use `_global` casually** — globals pollute the namespace; prefer locals and passed arguments
- **Don't leave production code without error handling** — always protect against tracebacks reaching the end user
- **Don't use `hash_table` in Smallworld 5 when not needed** — prefer `concurrent_hash_map` for performance and thread safety; use `property_list` only when insertion order matters
- **Don't concatenate strings with `+`** — use `sw:write_string(a, b, c)` instead
- **Don't access slots directly outside `init()` and accessor methods** — always go through `define_slot_access`-generated accessors
- **Don't define class constants inside method bodies** — use `define_shared_constant()`
- **Don't leave `_protection` blocks empty** when a resource must be closed — an empty `_protection` block is a bug
- **Don't confuse `empty?` and `empty()`** — `empty?` tests emptiness; `empty()` clears the collection
- **Don't use string concatenation for file paths** — use `sw:system.pathname_down()` and related utilities
- **Don't define multiple `init()` methods** — a single initialisation entry point makes subclassing far simpler
- **Don't use `!name!` naming for globals** — the `!name!` convention is reserved for dynamic (thread-local) variables
- **Don't define methods with brackets just because they are slow** — if the behaviour is slot-like, leave out the brackets and document the cost instead
- **Don't link related classes via globals** — use `define_shared_constant()` to hold a reference to the related exemplar

---

## Tooling

- **IDEs:** MDT (Eclipse-based), VS Code with `smallworld-magik-vscode` extension (Systemap)
- **Language Server / Linter:** `magik-tools` by StevenLooman (GitHub); see also the `magik-lsp` plugin in this repository (`plugins/magik-lsp/`) for Claude Code integration
- **REPL:** The Magik prompt inside a running Smallworld session
- **Compilation:** F9 (compile buffer), F2-b, or `magik_rep.load_chunck()`
- **Session start (SW5):** `gis_aliases` file via `F2-z` in VS Code extension

---

## Quick Reference Card

```
Assignment:        a << value
Compound assign:   a +<< value
Equality:          a = b
Identity:          a _is b    /    a _isnt b
Not equal:         a ~= b
Boolean:           _true  _false  _maybe (Kleenean)
Null:              _unset
Self:              _self
Return:            _return val   or   >> val
New instance:      my_exemplar.new(...)  →  internally _clone.init(...)
Slot access:       .slot_name  (only in init/accessors)
Slot accessors:    my_obj.define_slot_access(:slot, :writable, :public)
Shared constant:   my_obj.define_shared_constant(:name, value, :public)
Private method:    _private _method ...
Internal-public:   _method my_obj.int!name()  (classify_level=restricted)
Optional param:    _optional param_name
Varargs:           _gather param_name
Iterator method:   _iter _method ... _loopbody(val) ...
Loop break:        _leave
Loop continue:     _continue
Pragma:            _pragma(classify_level=basic)
Docstring:         ## Description with PARAMS uppercase. Returns ...
Boolean method:    name ending in ?   (returns _true/_false, usable as condition)
Kleenean method:   name ending in ??  (returns _true/_maybe/_false, compare with _is)
Slot-like method:  no brackets — my_obj.name
Non-slot method:   brackets — my_obj.calculate()
Setter:            foo<<  (paired with slot-like getter foo)
String concat:     sw:write_string(a, b, c)   (not a + b + c)
Empty test:        col.empty?     (not col.empty() which clears!)
```

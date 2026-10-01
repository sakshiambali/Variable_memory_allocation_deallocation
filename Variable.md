# Python Variables, Objects, and Memory

This guide explains how Python names refer to objects, how common built-in data types behave, and how Python manages memory. The memory diagrams are conceptual; Python implementations may manage memory differently internally.

## 1. Variables Are Names for Objects

In Python, a variable is a name bound to an object. It is useful to picture a name as a label or reference, rather than a box that contains a value.

```python
x = 10
```

Conceptually, `x` refers to the integer object `10`:

```text
x -> 10
```

Python values are objects. An object has an identity, a type, and a value. You can inspect these with `id()`, `type()`, and `print()`:

```python
x = 10
print(id(x))   # identity (an integer identifying the object during its lifetime)
print(type(x)) # <class 'int'>
print(x)       # 10
```

`id()` reports an object's identity. Do not assume it is a memory address; that is an implementation detail.

## 2. Common Built-in Data Types

| Category | Types | Example |
| --- | --- | --- |
| Numeric | `int`, `float`, `complex` | `25`, `99.5`, `3 + 4j` |
| Boolean | `bool` | `True`, `False` |
| Text | `str` | `"Python"` |
| Sequence | `list`, `tuple`, `range` | `[1, 2]`, `(1, 2)`, `range(3)` |
| Set | `set`, `frozenset` | `{1, 2}`, `frozenset({1, 2})` |
| Mapping | `dict` | `{"id": 101}` |
| Binary | `bytes`, `bytearray`, `memoryview` | `b"data"` |
| Special | `NoneType` | `None` |

### Numeric Types

```python
age = 25                 # int
temperature = -10        # int
price = 99.50             # float
percentage = 85.75        # float
complex_number = 3 + 4j   # complex
```

### Boolean Values and Truthiness

Python's Boolean values are spelled `True` and `False` (with capital first letters). The `bool()` function converts a value to its truth value.

```python
is_active = True
is_logged_in = False

print(bool(0))       # False
print(bool(1))       # True
print(bool(""))      # False
print(bool("Hello")) # True
print(bool([]))      # False
```

### Strings

A string is an immutable sequence of characters. Individual characters can be accessed by index:

```python
name = "Python"
print(name[0]) # P
print(name[1]) # y
```

### Lists

Lists are ordered and mutable. They allow duplicate values and can contain values of different types.

```python
numbers = [10, 20, 30]
data = [10, "Python", 25.5, True]
```

### Tuples

Tuples are ordered and immutable. They allow duplicate values.

```python
point = (10, 20)
```

### Sets

Sets are mutable collections of unique values. They are not used for positional indexing, and their iteration order should not be relied upon.

```python
numbers = {10, 20, 20, 30}
print(numbers) # contains 10, 20, and 30; the duplicate 20 is removed
```

`frozenset` is the immutable set type.

### Dictionaries

Dictionaries store key-value pairs:

```python
student = {"id": 101, "name": "Yati", "marks": 85}
```

### `None`

`None` represents the absence of a value and has type `NoneType`.

```python
result = None
```

`None`, `0`, `False`, `""`, and `[]` are distinct values, even though several of them evaluate as false in a Boolean context.

## 3. Mutable and Immutable Objects

An immutable object cannot be changed after it is created. A mutable object can be changed.

| Generally immutable | Mutable |
| --- | --- |
| `int`, `float`, `complex`, `bool`, `str`, `tuple`, `frozenset`, `bytes` | `list`, `dict`, `set`, `bytearray` |

A tuple itself is immutable, though it can contain a mutable object. In that case, the tuple's references cannot be replaced, but the referenced mutable object may still change.

### Rebinding a Name

Assigning a new value to a name rebinds that name; it does not modify the original immutable object.

```python
a = 10
b = a
a = 20

print(a) # 20
print(b) # 10
```

After `b = a`, both names refer to the value `10`. The later assignment makes `a` refer to `20`; it does not change `b`.

### Multiple Names Referring to a Mutable Object

Assigning one name to another does not copy a list. Both names refer to the same list, so a mutation through either name is visible through the other.

```python
a = [10, 20]
b = a
b.append(30)

print(a) # [10, 20, 30]
```

Conceptually:

```text
a ----> [10, 20, 30]
b ----/        (one list object)
```

## 4. Equality and Identity: `==` vs `is`

- `==` checks whether two objects have equal values.
- `is` checks whether two names refer to the exact same object.

```python
a = [1, 2]
b = [1, 2]

print(a == b) # True: same contents
print(a is b) # False: separate list objects
```

Use `is` most commonly to check for a singleton such as `None` (`result is None`). Use `==` when comparing values.

## 5. Memory Allocation in Python

Creating values and objects requires memory. Python manages that memory for you; you do not normally allocate or free memory for each variable yourself.

At a conceptual level, a program's memory can be used for objects such as integers, strings, lists, dictionaries, and functions. Names refer to objects, while the Python runtime manages their storage. The exact representation and allocation strategy depend on the Python implementation.

> Python names refer to objects. Python manages object memory dynamically, but the details are implementation-dependent.

## 6. Reference Counting and Garbage Collection

### Reference Counting in CPython

CPython, the most widely used Python implementation, primarily uses reference counting. Conceptually, an object remains referenced while names or other objects point to it.

```python
a = [1, 2, 3]
b = a
```

Conceptually, both `a` and `b` refer to the same list. Removing one reference leaves the other:

```python
a = [1, 2, 3]
b = a
del b
print(a) # [1, 2, 3]
```

This is a conceptual explanation, not a recommendation to rely on an exact reference count.

### Cyclic Garbage Collection

Reference counting alone cannot reclaim an unreachable group of objects that refer to one another. CPython's cyclic garbage collector can detect and handle such unreachable cycles.

```python
cycle = []
cycle.append(cycle) # the list refers to itself
del cycle           # removes the name; the cycle is now unreachable here
```

The cycle can be reclaimed by the cyclic garbage collector. Collection timing is not guaranteed.

## 7. What `del` Does (and Does Not Do)

The `del` statement removes a name or another reference; it does not directly command Python to free a specific block of memory.

```python
numbers = [1, 2, 3]
other_name = numbers
del numbers

print(other_name) # [1, 2, 3]
```

The list is still reachable through `other_name`. When no references remain, an object may become eligible for reclamation. The exact time it is reclaimed, and whether the memory is returned to the operating system or reused internally, depends on the implementation and runtime.

### Why Doesn't `del numbers` Necessarily Destroy the Object Immediately?

Because `del numbers` removes the name `numbers`; it does not necessarily remove every reference to the object. Another name, container, or part of the program may still refer to it. Even if no references remain, Python does not promise that memory will be reclaimed or returned to the operating system immediately.

## 8. Key Idea

```text
name -> object (identity, type, value) -> runtime-managed memory
                                  |
                    no longer reachable
                                  v
                      eligible for reclamation
```

- Variables are names bound to objects, not boxes that contain values.
- Assignment can create another reference; it does not necessarily copy an object.
- Mutating a shared mutable object is visible through every reference to it.
- Python manages memory automatically. In CPython, reference counting works alongside cyclic garbage collection.
- Removing a name does not guarantee immediate object destruction or an immediate decrease in process memory.
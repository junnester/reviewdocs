# Python Advanced Tutorial

## Table of Contents
1. [Metaclasses](#metaclasses)
2. [Descriptors](#descriptors)
3. [Coroutines and Async/Await](#coroutines-and-asyncawait)
4. [Type Hints and Annotations](#type-hints-and-annotations)
5. [Advanced Decorators](#advanced-decorators)
6. [Memory Management and Performance](#memory-management-and-performance)
7. [Metaclasses and Dynamic Programming](#metaclasses-and-dynamic-programming)
8. [Working with C Extensions](#working-with-c-extensions)

---

## Metaclasses

### Understanding Metaclasses
```python
# Metaclasses are classes whose instances are classes
class Meta(type):
    def __new__(mcs, name, bases, namespace):
        print(f"Creating class {name}")
        namespace['custom_attr'] = "Added by metaclass"
        return super().__new__(mcs, name, bases, namespace)

class MyClass(metaclass=Meta):
    pass

print(MyClass.custom_attr)  # Output: Added by metaclass
```

### Controlling Class Creation
```python
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        self.connection = "Connected"

db1 = Database()
db2 = Database()
print(db1 is db2)  # Output: True
```

### Class Registration Pattern
```python
class PluginRegistry(type):
    plugins = {}
    
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if 'plugin_name' in namespace:
            mcs.plugins[namespace['plugin_name']] = cls
        return cls

class Plugin(metaclass=PluginRegistry):
    pass

class PDFPlugin(Plugin):
    plugin_name = "pdf"

class ImagePlugin(Plugin):
    plugin_name = "image"

print(PluginRegistry.plugins)
# Output: {'pdf': <class 'PDFPlugin'>, 'image': <class 'ImagePlugin'>}
```

---

## Descriptors

### Data Descriptors
```python
class Descriptor:
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name, None)
    
    def __set__(self, obj, value):
        obj.__dict__[self.name] = value

class Person:
    age = Descriptor()
    
    def __init__(self, name, age):
        self.name = name
        self.age = age

p = Person("Alice", 30)
print(p.age)  # Output: 30
p.age = 31
print(p.age)  # Output: 31
```

### Property Validation
```python
class ValidatedProperty:
    def __init__(self, min_value=None, max_value=None):
        self.min_value = min_value
        self.max_value = max_value
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name, None)
    
    def __set__(self, obj, value):
        if self.min_value is not None and value < self.min_value:
            raise ValueError(f"Value must be >= {self.min_value}")
        if self.max_value is not None and value > self.max_value:
            raise ValueError(f"Value must be <= {self.max_value}")
        obj.__dict__[self.name] = value

class Temperature:
    celsius = ValidatedProperty(min_value=-273.15)
    
    def __init__(self, celsius):
        self.celsius = celsius

temp = Temperature(25)
print(temp.celsius)  # Output: 25
# temp.celsius = -300  # Raises ValueError
```

### Lazy Loading with Descriptors
```python
class LazyProperty:
    def __init__(self, func):
        self.func = func
        self.name = func.__name__
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        value = self.func(obj)
        setattr(obj, self.name, value)
        return value

class DataLoader:
    @LazyProperty
    def expensive_data(self):
        print("Loading expensive data...")
        import time
        time.sleep(2)
        return "Data loaded"

loader = DataLoader()
print(loader.expensive_data)  # First call: loads data
print(loader.expensive_data)  # Second call: returns cached value
```

---

## Coroutines and Async/Await

### Basic Async Functions
```python
import asyncio

async def fetch_data(name, delay):
    print(f"Fetching {name}...")
    await asyncio.sleep(delay)
    return f"Data from {name}"

async def main():
    result1 = await fetch_data("Source1", 2)
    result2 = await fetch_data("Source2", 1)
    print(result1)
    print(result2)

asyncio.run(main())
```

### Running Concurrent Tasks
```python
import asyncio

async def task(name, delay):
    print(f"Task {name} started")
    await asyncio.sleep(delay)
    print(f"Task {name} finished")
    return f"Result from {name}"

async def main():
    # Run tasks concurrently
    results = await asyncio.gather(
        task("A", 2),
        task("B", 1),
        task("C", 3)
    )
    print(results)

asyncio.run(main())
```

### Task Cancellation and Timeouts
```python
import asyncio

async def long_running_task():
    try:
        await asyncio.sleep(10)
        return "Completed"
    except asyncio.CancelledError:
        print("Task was cancelled")
        raise

async def main():
    task = asyncio.create_task(long_running_task())
    try:
        result = await asyncio.wait_for(task, timeout=2)
    except asyncio.TimeoutError:
        print("Task timed out")
        task.cancel()

asyncio.run(main())
```

### Async Context Managers
```python
import asyncio

class AsyncResource:
    async def __aenter__(self):
        print("Acquiring resource...")
        await asyncio.sleep(1)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource...")
        await asyncio.sleep(1)

async def main():
    async with AsyncResource() as resource:
        print("Using resource...")

asyncio.run(main())
```

---

## Type Hints and Annotations

### Basic Type Hints
```python
from typing import List, Dict, Tuple, Optional, Union

def greet(name: str) -> str:
    return f"Hello, {name}!"

def add_numbers(a: int, b: int) -> int:
    return a + b

def process_items(items: List[int]) -> Dict[str, int]:
    return {"sum": sum(items), "count": len(items)}

def get_coordinates() -> Tuple[float, float]:
    return (10.5, 20.3)

def get_value(key: str) -> Optional[str]:
    data = {"a": "value_a"}
    return data.get(key)
```

### Advanced Type Hints
```python
from typing import Callable, TypeVar, Generic, Protocol

# Callable
def apply_operation(a: int, b: int, op: Callable[[int, int], int]) -> int:
    return op(a, b)

result = apply_operation(5, 3, lambda x, y: x + y)

# TypeVar for generic types
T = TypeVar('T')

def get_first_element(items: List[T]) -> T:
    return items[0]

# Generic class
class Container(Generic[T]):
    def __init__(self, item: T):
        self.item = item
    
    def get(self) -> T:
        return self.item

# Protocol for structural subtyping
class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())
```

---

## Advanced Decorators

### Function Wrapping with functools
```python
from functools import wraps
import time

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "Done"

slow_function()
```

### Class Decorators
```python
from functools import wraps

def add_repr(cls):
    original_init = cls.__init__
    
    @wraps(original_init)
    def new_init(self, *args, **kwargs):
        original_init(self, *args, **kwargs)
    
    cls.__init__ = new_init
    
    def __repr__(self):
        attrs = ", ".join(f"{k}={v!r}" for k, v in self.__dict__.items())
        return f"{cls.__name__}({attrs})"
    
    cls.__repr__ = __repr__
    return cls

@add_repr
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

p = Person("Alice", 30)
print(p)  # Output: Person(name='Alice', age=30)
```

### Stacking Decorators
```python
def bold(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

@bold
@italic
def format_text(text):
    return text

print(format_text("Hello"))  # Output: <b><i>Hello</i></b>
```

---

## Memory Management and Performance

### Understanding Memory Usage
```python
import sys
import gc

# Check object size
text = "Hello"
print(sys.getsizeof(text))  # Size in bytes

# Garbage collection
gc.collect()  # Force garbage collection
print(gc.get_stats())  # Get garbage collection stats

# Weak references to avoid circular references
from weakref import ref

class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

node1 = Node(1)
node2 = Node(2)
node1.next = ref(node2)  # Weak reference
```

### Profiling Code Performance
```python
import cProfile
import pstats
from io import StringIO

def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

profiler = cProfile.Profile()
profiler.enable()

fibonacci(30)

profiler.disable()
stream = StringIO()
stats = pstats.Stats(profiler, stream=stream)
stats.print_stats()
print(stream.getvalue())
```

### Optimization Tips
```python
# Use list comprehension instead of append
# Fast:
squares = [x**2 for x in range(1000)]

# Slow:
squares = []
for x in range(1000):
    squares.append(x**2)

# Use built-in functions instead of loops
# Fast:
total = sum(range(1000))

# Slow:
total = 0
for i in range(1000):
    total += i

# Use generators for large datasets
def large_dataset():
    for i in range(1000000):
        yield i

for value in large_dataset():
    process(value)  # Process one at a time
```

---

## Metaclasses and Dynamic Programming

### Dynamic Class Creation
```python
# Create a class dynamically
def create_person_class(name_attr):
    def __init__(self, value):
        setattr(self, name_attr, value)
    
    return type('Person', (), {'__init__': __init__})

PersonClass = create_person_class('full_name')
person = PersonClass("Alice")
print(person.full_name)  # Output: Alice
```

### Method Resolution Order (MRO)
```python
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):
    pass

print(D.mro())  # Shows method resolution order
# [<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>]

d = D()
print(d.method())  # Output: B
```

### Using __getattr__ and __setattr__
```python
class DynamicAttributes:
    def __init__(self):
        self._data = {}
    
    def __getattr__(self, name):
        print(f"Accessing {name}")
        return self._data.get(name, None)
    
    def __setattr__(self, name, value):
        if name == '_data':
            super().__setattr__(name, value)
        else:
            print(f"Setting {name} = {value}")
            self._data[name] = value

obj = DynamicAttributes()
obj.x = 10
print(obj.x)
```

---

## Working with C Extensions

### Using ctypes for C Libraries
```python
from ctypes import CDLL, c_int, c_char_p
import platform

# Load a C library (example varies by OS)
if platform.system() == "Linux":
    libc = CDLL("libc.so.6")
elif platform.system() == "Darwin":
    libc = CDLL("libSystem.dylib")
else:
    libc = CDLL("msvcrt.dll")

# Use C functions
strlen = libc.strlen
strlen.argtypes = [c_char_p]
strlen.restype = c_int

text = b"Hello"
print(strlen(text))  # Output: 5
```

### Creating Python C Extensions
```python
# example.c (simplified pseudocode)
// PyObject* example_add(PyObject* self, PyObject* args) {
//     int a, b;
//     if (!PyArg_ParseTuple(args, "ii", &a, &b))
//         return NULL;
//     return PyLong_FromLong(a + b);
// }
```

---

## Best Practices for Advanced Python

1. **Use type hints**: Make code more maintainable and catch errors early.
2. **Profile before optimizing**: Identify actual bottlenecks with profiling tools.
3. **Avoid premature optimization**: Keep code readable unless performance is critical.
4. **Use async/await for I/O-bound operations**: Don't use it for CPU-bound tasks.
5. **Understand metaclasses before using them**: They add complexity and should be avoided when possible.
6. **Use descriptors for reusable validation logic**: Cleaner than repeating validation code.
7. **Document complex code thoroughly**: Advanced patterns can be hard to understand.
8. **Test extensively**: Advanced features increase the chance of subtle bugs.
9. **Keep memory usage in mind**: Use generators and weak references when appropriate.
10. **Follow PEP 20 (The Zen of Python)**: "Simple is better than complex."

---

## Conclusion

Advanced Python programming involves understanding deep language internals and using advanced patterns for specific problems. Master these concepts to write sophisticated, efficient, and maintainable Python applications.

For more information, refer to the [Python Documentation](https://docs.python.org/).

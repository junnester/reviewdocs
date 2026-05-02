# Python Intermediate Tutorial

## Table of Contents
1. [Advanced Functions](#advanced-functions)
2. [Object-Oriented Programming](#object-oriented-programming)
3. [Decorators](#decorators)
4. [Context Managers](#context-managers)
5. [Generators and Iterators](#generators-and-iterators)
6. [Exception Handling](#exception-handling)
7. [Working with Modules and Packages](#working-with-modules-and-packages)

---

## Advanced Functions

### Default Arguments
```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

print(greet("Alice"))  # Output: Hello, Alice!
print(greet("Bob", "Hi"))  # Output: Hi, Bob!
```

### *args and **kwargs
```python
def print_args(*args, **kwargs):
    for arg in args:
        print(f"Argument: {arg}")
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_args(1, 2, 3, name="Alice", age=30)
```

### Lambda Functions
```python
# Lambda with map
numbers = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, numbers))
print(doubled)  # Output: [2, 4, 6, 8, 10]

# Lambda with filter
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # Output: [2, 4]
```

### Comprehensions
```python
# List comprehension
squares = [x**2 for x in range(5)]
print(squares)  # Output: [0, 1, 4, 9, 16]

# Dictionary comprehension
squares_dict = {x: x**2 for x in range(5)}
print(squares_dict)  # Output: {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Set comprehension
unique_squares = {x**2 for x in [1, 1, 2, 2, 3]}
print(unique_squares)  # Output: {1, 4, 9}
```

---

## Object-Oriented Programming

### Classes and Objects
```python
class Dog:
    species = "Canis familiaris"
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __str__(self):
        return f"{self.name} is {self.age} years old"
    
    def bark(self):
        return f"{self.name} says Woof!"

my_dog = Dog("Buddy", 3)
print(my_dog)  # Output: Buddy is 3 years old
print(my_dog.bark())  # Output: Buddy says Woof!
```

### Inheritance
```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        raise NotImplementedError("Subclass must implement this method")

class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"

class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"

pets = [Cat("Whiskers"), Dog("Rex")]
for pet in pets:
    print(pet.speak())
```

### Class Methods and Static Methods
```python
class Circle:
    pi = 3.14159
    
    def __init__(self, radius):
        self.radius = radius
    
    @classmethod
    def from_diameter(cls, diameter):
        return cls(diameter / 2)
    
    @staticmethod
    def is_positive(value):
        return value > 0
    
    def area(self):
        return self.pi * self.radius ** 2

c1 = Circle(5)
c2 = Circle.from_diameter(10)
print(Circle.is_positive(-5))  # Output: False
```

---

## Decorators

### Basic Decorators
```python
def decorator(func):
    def wrapper(*args, **kwargs):
        print("Function is being called")
        result = func(*args, **kwargs)
        print("Function call completed")
        return result
    return wrapper

@decorator
def say_hello(name):
    return f"Hello, {name}!"

say_hello("Alice")
# Output:
# Function is being called
# Hello, Alice!
# Function call completed
```

### Decorators with Parameters
```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    return f"Hello, {name}!"

print(greet("Bob"))
# Output: ['Hello, Bob!', 'Hello, Bob!', 'Hello, Bob!']
```

---

## Context Managers

### Using Context Managers
```python
# File handling with context manager
with open("example.txt", "r") as file:
    content = file.read()
    print(content)
# File is automatically closed

# Creating custom context manager
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        return False

with FileManager("test.txt", "w") as f:
    f.write("Hello, World!")
```

---

## Generators and Iterators

### Generators
```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for num in countdown(5):
    print(num)
# Output: 5 4 3 2 1

# Generator expression
gen = (x**2 for x in range(5))
print(next(gen))  # Output: 0
print(next(gen))  # Output: 1
```

### Custom Iterators
```python
class CountUp:
    def __init__(self, max):
        self.max = max
        self.current = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current < self.max:
            self.current += 1
            return self.current
        else:
            raise StopIteration

counter = CountUp(3)
for num in counter:
    print(num)
# Output: 1 2 3
```

---

## Exception Handling

### Try, Except, Else, Finally
```python
def divide(a, b):
    try:
        result = a / b
    except ZeroDivisionError:
        print("Cannot divide by zero!")
        result = None
    except TypeError:
        print("Invalid input type!")
        result = None
    else:
        print(f"Division successful: {result}")
    finally:
        print("Operation completed")
    
    return result

divide(10, 2)
divide(10, 0)
```

### Custom Exceptions
```python
class InvalidAgeError(Exception):
    pass

def set_age(age):
    if age < 0 or age > 150:
        raise InvalidAgeError(f"Age {age} is not valid!")
    return age

try:
    set_age(200)
except InvalidAgeError as e:
    print(f"Error: {e}")
```

---

## Working with Modules and Packages

### Creating Modules
```python
# math_operations.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

# Using the module
import math_operations
result = math_operations.add(5, 3)
print(result)  # Output: 8

# Or use from import
from math_operations import add
result = add(5, 3)
```

### Creating Packages
```
mypackage/
    __init__.py
    module1.py
    module2.py
    subpackage/
        __init__.py
        module3.py
```

### Using functools and itertools
```python
from functools import reduce
from itertools import combinations, permutations

# reduce
numbers = [1, 2, 3, 4, 5]
product = reduce(lambda x, y: x * y, numbers)
print(product)  # Output: 120

# combinations and permutations
items = [1, 2, 3]
print(list(combinations(items, 2)))  # Output: [(1, 2), (1, 3), (2, 3)]
print(list(permutations(items, 2)))  # Output: [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]
```

---

## Best Practices

1. **Write clean code**: Follow PEP 8 style guide for Python code.
2. **Use meaningful names**: Choose descriptive variable and function names.
3. **Document your code**: Write docstrings and comments for clarity.
4. **Handle exceptions properly**: Use specific exception types, not generic ones.
5. **Use virtual environments**: Isolate project dependencies.
6. **Write tests**: Create unit tests to verify your code works correctly.
7. **Avoid mutable default arguments**: Use `None` as default instead.
8. **Use list comprehensions**: More efficient and readable than loops.

---

## Conclusion

This tutorial covers essential intermediate Python concepts. Practice these patterns regularly to master them and write more efficient, maintainable Python code.

For more information, check out the [official Python documentation](https://docs.python.org/).

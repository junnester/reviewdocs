# Python Basics Tutorial

## Table of Contents
1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Variables and Data Types](#variables-and-data-types)
4. [Operators](#operators)
5. [Control Flow](#control-flow)
6. [Functions](#functions)
7. [Data Structures](#data-structures)
8. [File Handling](#file-handling)
9. [Best Practices](#best-practices)

## Introduction

Python is a high-level, interpreted programming language known for its simplicity and readability. It's widely used in web development, data science, machine learning, automation, and more.

### Why Python?
- **Easy to Learn**: Simple syntax similar to English
- **Versatile**: Used in many different domains
- **Large Community**: Extensive libraries and frameworks
- **Highly Readable**: Clean code structure

## Getting Started

### Installation

Download Python from [python.org](https://www.python.org/downloads/) and follow the installation instructions for your operating system.

### Your First Program

```python
print("Hello, World!")
```

Run this in your terminal:
```bash
python hello.py
```

## Variables and Data Types

### Variables

Variables are containers for storing data values. Python doesn't require you to declare the type of a variable.

```python
# String
name = "Alice"

# Integer
age = 25

# Float
height = 5.6

# Boolean
is_student = True
```

### Data Types

| Type | Description | Example |
|------|-------------|---------|
| `str` | String (text) | `"Hello"` |
| `int` | Integer (whole number) | `42` |
| `float` | Float (decimal number) | `3.14` |
| `bool` | Boolean (True/False) | `True` |
| `list` | Ordered collection | `[1, 2, 3]` |
| `dict` | Key-value pairs | `{"name": "Alice"}` |
| `tuple` | Immutable collection | `(1, 2, 3)` |
| `set` | Unordered unique values | `{1, 2, 3}` |

### Type Conversion

```python
# Convert to string
num_str = str(42)

# Convert to integer
int_num = int("25")

# Convert to float
float_num = float("3.14")

# Convert to boolean
bool_val = bool(1)  # True
```

## Operators

### Arithmetic Operators

```python
a = 10
b = 3

print(a + b)   # Addition: 13
print(a - b)   # Subtraction: 7
print(a * b)   # Multiplication: 30
print(a / b)   # Division: 3.333...
print(a // b)  # Floor Division: 3
print(a % b)   # Modulus: 1
print(a ** b)  # Exponentiation: 1000
```

### Comparison Operators

```python
x = 5
y = 10

print(x == y)  # Equal to: False
print(x != y)  # Not equal to: True
print(x < y)   # Less than: True
print(x > y)   # Greater than: False
print(x <= y)  # Less than or equal to: True
print(x >= y)  # Greater than or equal to: False
```

### Logical Operators

```python
a = True
b = False

print(a and b)  # Logical AND: False
print(a or b)   # Logical OR: True
print(not a)    # Logical NOT: False
```

## Control Flow

### if Statement

```python
age = 18

if age >= 18:
    print("You are an adult")
elif age >= 13:
    print("You are a teenager")
else:
    print("You are a child")
```

### for Loop

```python
# Loop through a range
for i in range(5):
    print(i)  # Prints 0, 1, 2, 3, 4

# Loop through a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# Loop with index
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
```

### while Loop

```python
count = 0
while count < 5:
    print(count)
    count += 1
```

### break and continue

```python
# break: Exit the loop
for i in range(10):
    if i == 5:
        break
    print(i)  # Prints 0, 1, 2, 3, 4

# continue: Skip to next iteration
for i in range(5):
    if i == 2:
        continue
    print(i)  # Prints 0, 1, 3, 4
```

## Functions

### Defining Functions

```python
def greet(name):
    """This function greets someone"""
    return f"Hello, {name}!"

print(greet("Alice"))  # Hello, Alice!
```

### Function Parameters

```python
# Default parameters
def greet(name="Guest"):
    return f"Hello, {name}!"

# Keyword arguments
def introduce(name, age, city="Unknown"):
    return f"{name} is {age} years old and lives in {city}"

print(introduce("Bob", 30, city="New York"))

# Variable-length arguments
def add(*numbers):
    return sum(numbers)

print(add(1, 2, 3, 4))  # 10

# Keyword variable-length arguments
def print_info(**info):
    for key, value in info.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=25, city="NYC")
```

### Lambda Functions

```python
# Anonymous function
square = lambda x: x ** 2
print(square(5))  # 25

# Used with built-in functions
numbers = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, numbers))
print(doubled)  # [2, 4, 6, 8, 10]
```

## Data Structures

### Lists

```python
# Create a list
fruits = ["apple", "banana", "cherry"]

# Access elements
print(fruits[0])   # apple
print(fruits[-1])  # cherry (last element)

# Add elements
fruits.append("date")
fruits.extend(["fig", "grape"])

# Remove elements
fruits.remove("banana")
popped = fruits.pop()  # Removes and returns last element

# List methods
fruits.sort()
fruits.reverse()
length = len(fruits)

# List comprehension
squares = [x ** 2 for x in range(5)]
print(squares)  # [0, 1, 4, 9, 16]
```

### Dictionaries

```python
# Create a dictionary
person = {
    "name": "Alice",
    "age": 25,
    "city": "New York"
}

# Access values
print(person["name"])  # Alice

# Add/Update
person["email"] = "alice@example.com"
person["age"] = 26

# Remove
del person["city"]

# Dictionary methods
keys = person.keys()
values = person.values()
items = person.items()

# Get with default
city = person.get("city", "Unknown")  # Unknown
```

### Tuples

```python
# Create a tuple (immutable)
coordinates = (10, 20, 30)

# Access elements
print(coordinates[0])  # 10

# Tuple unpacking
x, y, z = coordinates

# Tuples are immutable - this will cause an error:
# coordinates[0] = 5  # TypeError
```

### Sets

```python
# Create a set
unique_numbers = {1, 2, 3, 4, 5}

# Add elements
unique_numbers.add(6)

# Remove elements
unique_numbers.remove(3)

# Set operations
set1 = {1, 2, 3}
set2 = {3, 4, 5}

union = set1 | set2          # {1, 2, 3, 4, 5}
intersection = set1 & set2   # {3}
difference = set1 - set2     # {1, 2}
```

## File Handling

### Reading Files

```python
# Read entire file
with open("file.txt", "r") as file:
    content = file.read()

# Read line by line
with open("file.txt", "r") as file:
    for line in file:
        print(line.strip())

# Read all lines into a list
with open("file.txt", "r") as file:
    lines = file.readlines()
```

### Writing Files

```python
# Write to file (overwrites)
with open("file.txt", "w") as file:
    file.write("Hello, World!")

# Append to file
with open("file.txt", "a") as file:
    file.write("\nNew line")

# Write multiple lines
with open("file.txt", "w") as file:
    file.writelines(["Line 1\n", "Line 2\n", "Line 3\n"])
```

## Best Practices

### 1. **Use Meaningful Names**
```python
# Good
user_age = 25
is_active = True

# Avoid
ua = 25
ia = True
```

### 2. **Write Comments and Docstrings**
```python
def calculate_total(items):
    """
    Calculate the total price of items.
    
    Args:
        items (list): List of item prices
    
    Returns:
        float: Total price
    """
    return sum(items)
```

### 3. **Follow PEP 8 Style Guide**
- Use 4 spaces for indentation
- Maximum line length of 79 characters
- Use lowercase for variable names with underscores

### 4. **Handle Exceptions**
```python
try:
    number = int("not a number")
except ValueError:
    print("Invalid input")
finally:
    print("Execution complete")
```

### 5. **Use List Comprehensions**
```python
# Instead of:
squares = []
for x in range(10):
    squares.append(x ** 2)

# Use:
squares = [x ** 2 for x in range(10)]
```

### 6. **Test Your Code**
```python
def add(a, b):
    return a + b

# Test
assert add(2, 3) == 5
assert add(-1, 1) == 0
print("All tests passed!")
```

## Next Steps

- Practice writing Python scripts
- Explore libraries like `requests`, `numpy`, and `pandas`
- Learn about object-oriented programming (OOP)
- Try building projects like web apps, data analysis tools, or automation scripts

---

**Happy Coding!** 🐍

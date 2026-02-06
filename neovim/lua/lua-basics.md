# Lua Basics

A comprehensive guide to Lua programming fundamentals.

## Introduction

Lua is a lightweight, embeddable scripting language. Neovim uses LuaJIT, a Just-In-Time compiler that makes Lua very fast.

## Comments

```lua
-- Single line comment

--[[
  Multi-line comment
  More lines...
]]

--[=[
  Multi-line comment with nested [[ ]]
  Can contain [[nested]] brackets
]=]
```

## Variables

### Declaration and Assignment

```lua
local x = 10              -- Local variable (preferred)
y = 20                    -- Global variable (avoid in Neovim)
local name = "Neovim"     -- String
local isEnabled = true    -- Boolean
```

### Naming Conventions
- Use `snake_case` for variables and functions
- Use `PascalCase` for classes/modules
- Start with letter or underscore
- Can contain letters, digits, underscores

### Scope
```lua
local outer = 1

if true then
  local inner = 2
  print(outer)  -- 1 (accessible)
  print(inner)  -- 2
end

print(outer)    -- 1
print(inner)    -- nil (not accessible)
```

**Important**: Always use `local` in Neovim configs to avoid polluting global namespace!

## Data Types

### Nil
```lua
local x = nil     -- Absence of value
local y           -- Also nil (uninitialized)

if x == nil then
  print("x is nil")
end
```

### Boolean
```lua
local isTrue = true
local isFalse = false

-- Falsy values: false and nil
-- Everything else is truthy (including 0 and "")
if 0 then          -- This is TRUE!
  print("0 is truthy in Lua")
end
```

### Numbers
```lua
local integer = 42
local float = 3.14
local scientific = 1.5e10
local hex = 0xFF          -- 255 in decimal

-- Math operations
local sum = 10 + 5        -- 15
local difference = 10 - 5 -- 5
local product = 10 * 5    -- 50
local quotient = 10 / 5   -- 2 (always returns float)
local modulo = 10 % 3     -- 1
local power = 2 ^ 3       -- 8
```

### Strings
```lua
local str1 = "double quotes"
local str2 = 'single quotes'
local str3 = [[
  Multi-line string
  preserves formatting
]]

-- String concatenation
local hello = "Hello"
local world = "World"
local greeting = hello .. " " .. world  -- "Hello World"

-- String length
local len = #"Hello"  -- 5

-- Common string functions
local upper = string.upper("hello")     -- "HELLO"
local lower = string.lower("HELLO")     -- "hello"
local sub = string.sub("Hello", 1, 3)   -- "Hel"
local find = string.find("Hello", "ll") -- 3, 4
local gsub = string.gsub("Hello", "l", "L")  -- "HeLLo"

-- String formatting
local formatted = string.format("%s is %d years old", "Lua", 30)
```

### Tables

Tables are the only data structure in Lua (like arrays, dictionaries, objects combined).

#### Arrays (1-indexed!)
```lua
local fruits = { "apple", "banana", "cherry" }

print(fruits[1])  -- "apple" (1-indexed!)
print(fruits[2])  -- "banana"

-- Length operator
print(#fruits)    -- 3

-- Iterate
for i, fruit in ipairs(fruits) do
  print(i, fruit)
end
```

#### Dictionaries (Hash Tables)
```lua
local person = {
  name = "John",
  age = 30,
  city = "Tokyo"
}

-- Access
print(person.name)       -- "John"
print(person["age"])     -- 30

-- Add/modify
person.job = "Engineer"
person["salary"] = 50000

-- Iterate
for key, value in pairs(person) do
  print(key, value)
end
```

#### Mixed Tables
```lua
local mixed = {
  "first",           -- [1] = "first"
  "second",          -- [2] = "second"
  key = "value",     -- ["key"] = "value"
  [10] = "tenth",    -- [10] = "tenth"
}

print(mixed[1])        -- "first"
print(mixed.key)       -- "value"
print(mixed[10])       -- "tenth"
```

#### Nested Tables
```lua
local config = {
  ui = {
    width = 80,
    height = 24,
  },
  keymaps = {
    save = "<C-s>",
    quit = "<C-q>",
  },
}

print(config.ui.width)     -- 80
print(config.keymaps.save) -- "<C-s>"
```

#### Table Operations
```lua
-- Insert at end
table.insert(fruits, "orange")

-- Insert at position
table.insert(fruits, 2, "grape")  -- Insert at index 2

-- Remove last element
local last = table.remove(fruits)

-- Remove at position
local second = table.remove(fruits, 2)

-- Concatenate array elements
local str = table.concat(fruits, ", ")  -- "apple, banana, cherry"

-- Sort
table.sort(fruits)
```

## Operators

### Arithmetic
```lua
+    -- Addition
-    -- Subtraction
*    -- Multiplication
/    -- Division (always float)
%    -- Modulo
^    -- Exponentiation
-    -- Unary minus
```

### Relational
```lua
==   -- Equal to
~=   -- Not equal to (not !=)
<    -- Less than
>    -- Greater than
<=   -- Less than or equal
>=   -- Greater than or equal
```

### Logical
```lua
and  -- Logical AND
or   -- Logical OR
not  -- Logical NOT

-- Short-circuit evaluation
local result = false and error("This won't execute")
local default = value or "default value"
```

### Length
```lua
#    -- Length of string or array portion of table

local str = "Hello"
local arr = {1, 2, 3, 4, 5}

print(#str)  -- 5
print(#arr)  -- 5
```

### Concatenation
```lua
..   -- String concatenation

local name = "Lua"
local version = 5.1
local info = name .. " " .. version  -- "Lua 5.1"
```

## Control Structures

### If-Else
```lua
local score = 85

if score >= 90 then
  print("A")
elseif score >= 80 then
  print("B")
elseif score >= 70 then
  print("C")
else
  print("F")
end
```

### While Loop
```lua
local i = 1
while i <= 5 do
  print(i)
  i = i + 1
end
```

### Repeat-Until Loop
```lua
local i = 1
repeat
  print(i)
  i = i + 1
until i > 5
```

### For Loops

#### Numeric For
```lua
-- for var = start, end, step do
for i = 1, 5 do        -- step defaults to 1
  print(i)             -- 1, 2, 3, 4, 5
end

for i = 5, 1, -1 do    -- Count down
  print(i)             -- 5, 4, 3, 2, 1
end

for i = 0, 10, 2 do    -- Even numbers
  print(i)             -- 0, 2, 4, 6, 8, 10
end
```

#### Generic For (Iterator)
```lua
-- ipairs: for arrays (sequential indices)
local fruits = {"apple", "banana", "cherry"}
for i, fruit in ipairs(fruits) do
  print(i, fruit)
end

-- pairs: for all key-value pairs
local person = {name = "John", age = 30}
for key, value in pairs(person) do
  print(key, value)
end
```

### Break and Return
```lua
-- Break: exit loop
for i = 1, 10 do
  if i == 5 then
    break
  end
  print(i)  -- 1, 2, 3, 4
end

-- Note: Lua doesn't have 'continue'
-- Use nested if or goto instead
for i = 1, 5 do
  if i ~= 3 then
    print(i)  -- 1, 2, 4, 5
  end
end
```

## Functions

### Basic Function
```lua
local function greet(name)
  return "Hello, " .. name
end

print(greet("World"))  -- Hello, World
```

### Multiple Return Values
```lua
local function min_max(a, b, c)
  local min = math.min(a, b, c)
  local max = math.max(a, b, c)
  return min, max
end

local minimum, maximum = min_max(5, 2, 8)
print(minimum, maximum)  -- 2, 8
```

### Variable Arguments
```lua
local function sum(...)
  local total = 0
  for _, num in ipairs({...}) do
    total = total + num
  end
  return total
end

print(sum(1, 2, 3, 4, 5))  -- 15
```

### Anonymous Functions
```lua
local square = function(x)
  return x * x
end

print(square(5))  -- 25

-- Inline usage
local numbers = {1, 2, 3, 4, 5}
local squares = {}
for i, num in ipairs(numbers) do
  squares[i] = (function(x) return x * x end)(num)
end
```

### Closures
```lua
local function make_counter()
  local count = 0
  return function()
    count = count + 1
    return count
  end
end

local counter = make_counter()
print(counter())  -- 1
print(counter())  -- 2
print(counter())  -- 3
```

### Optional Parameters
```lua
local function greet(name, greeting)
  greeting = greeting or "Hello"
  return greeting .. ", " .. name
end

print(greet("World"))           -- Hello, World
print(greet("World", "Hi"))     -- Hi, World
```

## Modules

### Creating a Module
```lua
-- file: mymodule.lua
local M = {}

M.version = "1.0"

function M.hello(name)
  return "Hello, " .. name
end

local function private_function()
  -- Only accessible within this file
  return "private"
end

function M.use_private()
  return private_function()
end

return M
```

### Using a Module
```lua
local mymodule = require('mymodule')

print(mymodule.version)        -- 1.0
print(mymodule.hello("Lua"))   -- Hello, Lua
print(mymodule.use_private())  -- private
```

## Metatables and Metamethods

Metatables allow you to change table behavior.

```lua
local vector = {x = 10, y = 20}

-- Create metatable
local mt = {
  -- Add two vectors
  __add = function(a, b)
    return {x = a.x + b.x, y = a.y + b.y}
  end,

  -- String representation
  __tostring = function(v)
    return "Vector(" .. v.x .. ", " .. v.y .. ")"
  end,
}

-- Set metatable
setmetatable(vector, mt)

local v1 = {x = 1, y = 2}
local v2 = {x = 3, y = 4}
setmetatable(v1, mt)
setmetatable(v2, mt)

local v3 = v1 + v2  -- Uses __add
print(v3.x, v3.y)   -- 4, 6
```

### Common Metamethods
- `__add` - Addition (+)
- `__sub` - Subtraction (-)
- `__mul` - Multiplication (*)
- `__div` - Division (/)
- `__eq` - Equality (==)
- `__lt` - Less than (<)
- `__le` - Less than or equal (<=)
- `__index` - Reading missing key
- `__newindex` - Writing to missing key
- `__call` - Call table as function
- `__tostring` - String representation

## Error Handling

### pcall (Protected Call)
```lua
local function risky_operation()
  error("Something went wrong!")
end

local success, result = pcall(risky_operation)

if success then
  print("Success:", result)
else
  print("Error:", result)
end
```

### xpcall (Extended Protected Call)
```lua
local function error_handler(err)
  return "Error: " .. err
end

local function risky()
  error("Failed!")
end

local success, result = xpcall(risky, error_handler)
print(result)  -- Error: Failed!
```

### assert
```lua
local function divide(a, b)
  assert(b ~= 0, "Cannot divide by zero")
  return a / b
end

divide(10, 2)  -- OK: 5
divide(10, 0)  -- Error: Cannot divide by zero
```

## Common Patterns

### Ternary Operator (Lua style)
```lua
-- condition and true_value or false_value
local result = (score >= 60) and "Pass" or "Fail"

-- But be careful with falsy values!
-- This is safer:
local function ternary(cond, if_true, if_false)
  if cond then return if_true else return if_false end
end
```

### Default Values
```lua
local config = config or {}
config.width = config.width or 80
config.height = config.height or 24
```

### Swapping Variables
```lua
local a, b = 1, 2
a, b = b, a
print(a, b)  -- 2, 1
```

### Array Mapping
```lua
local function map(arr, fn)
  local result = {}
  for i, v in ipairs(arr) do
    result[i] = fn(v)
  end
  return result
end

local numbers = {1, 2, 3, 4, 5}
local squares = map(numbers, function(x) return x * x end)
-- squares = {1, 4, 9, 16, 25}
```

### Array Filtering
```lua
local function filter(arr, predicate)
  local result = {}
  for _, v in ipairs(arr) do
    if predicate(v) then
      table.insert(result, v)
    end
  end
  return result
end

local numbers = {1, 2, 3, 4, 5, 6}
local evens = filter(numbers, function(x) return x % 2 == 0 end)
-- evens = {2, 4, 6}
```

## Standard Library Highlights

### Math
```lua
math.abs(-5)        -- 5
math.ceil(4.3)      -- 5
math.floor(4.7)     -- 4
math.max(1, 5, 3)   -- 5
math.min(1, 5, 3)   -- 1
math.random()       -- Random float between 0 and 1
math.random(10)     -- Random integer between 1 and 10
math.sqrt(16)       -- 4
math.pi             -- 3.14159...
```

### String
```lua
string.len(s)       -- Length
string.upper(s)     -- Uppercase
string.lower(s)     -- Lowercase
string.sub(s, i, j) -- Substring
string.find(s, pattern)  -- Find pattern
string.gsub(s, pattern, replacement)  -- Global substitution
string.match(s, pattern)  -- Match pattern
string.format(fmt, ...)   -- Format string
```

### Table
```lua
table.insert(t, v)       -- Insert at end
table.insert(t, i, v)    -- Insert at position
table.remove(t)          -- Remove last
table.remove(t, i)       -- Remove at position
table.concat(t, sep)     -- Join array with separator
table.sort(t, comp)      -- Sort table
```

### OS/IO (Use carefully in Neovim)
```lua
os.date()           -- Current date/time
os.time()           -- Unix timestamp
os.execute(cmd)     -- Execute shell command
```

## Best Practices for Neovim

1. **Always use `local`** - Avoid global variables
2. **Return modules** - Structure code as modules
3. **Use snake_case** - Follow Lua conventions
4. **Early returns** - Reduce nesting
5. **Check nil** - Always validate before use
6. **Prefer ipairs for arrays** - More efficient
7. **Use vim.inspect()** - For debugging tables

## Next Steps

- Learn [Lua in Neovim](./lua-neovim.md) - How Lua integrates with Neovim
- Explore [Neovim API](./neovim-api.md) - Neovim-specific Lua functions
- Study [Examples](./examples.md) - Real-world configurations

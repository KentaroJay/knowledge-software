# Lua Basics

## Introduction

Lua is a lightweight, high-level scripting language. In Neovim, Lua is used for:
- Plugin configuration
- Writing plugins
- Improving performance (faster than VimScript)

## Comments

```lua
-- Single line comment

--[[
  Multi-line
  comment
]]
```

## Variables

```lua
-- Global variable
name = "John"

-- Local variable (preferred)
local age = 25

-- Multiple assignment
local x, y, z = 1, 2, 3

-- Nil (undefined/null)
local nothing = nil
```

**Scoping:**
- Global variables are accessible everywhere
- Local variables are scoped to their block
- Always use `local` unless you need global scope

## Data Types

### 1. Nil
```lua
local x = nil  -- Represents absence of value
```

### 2. Boolean
```lua
local is_true = true
local is_false = false
```

### 3. Number
```lua
local integer = 42
local float = 3.14
local scientific = 1.5e10
```

### 4. String
```lua
local single = 'Hello'
local double = "World"
local multiline = [[
  Multi-line
  string
]]

-- String concatenation
local full = "Hello" .. " " .. "World"

-- String length
local len = #"Hello"  -- 5

-- String interpolation (Lua 5.4+)
local name = "Lua"
local msg = string.format("Hello, %s!", name)
```

### 5. Table
Tables are the only data structure in Lua. They work as arrays, dictionaries, and objects.

```lua
-- Array (1-indexed!)
local fruits = { "apple", "banana", "orange" }
print(fruits[1])  -- "apple" (NOT 0!)

-- Dictionary / Hash Map
local person = {
  name = "John",
  age = 30,
  city = "Tokyo"
}
print(person.name)    -- "John"
print(person["age"])  -- 30

-- Mixed
local mixed = {
  "first",              -- [1]
  "second",             -- [2]
  key = "value",        -- ["key"]
  nested = {            -- ["nested"]
    inner = "data"
  }
}

-- Empty table
local empty = {}

-- Table length (only for array part)
local count = #fruits  -- 3
```

### 6. Function
```lua
-- Function is a first-class value
local function greet(name)
  return "Hello, " .. name
end

-- Anonymous function
local add = function(a, b)
  return a + b
end

-- Multiple return values
local function minmax(a, b)
  if a < b then
    return a, b
  else
    return b, a
  end
end

local min, max = minmax(5, 3)  -- min=3, max=5
```

## Operators

### Arithmetic
```lua
local a = 10 + 5   -- Addition
local b = 10 - 5   -- Subtraction
local c = 10 * 5   -- Multiplication
local d = 10 / 5   -- Division
local e = 10 % 3   -- Modulo (remainder)
local f = 10 ^ 2   -- Exponentiation (power)
```

### Comparison
```lua
local equal = (10 == 10)        -- true
local not_equal = (10 ~= 5)     -- true (note: ~= not !=)
local less = (5 < 10)           -- true
local greater = (10 > 5)        -- true
local less_or_eq = (5 <= 5)     -- true
local greater_or_eq = (10 >= 5) -- true
```

### Logical
```lua
local and_op = true and false   -- false
local or_op = true or false     -- true
local not_op = not true         -- false

-- Short-circuit evaluation
local x = nil or "default"      -- "default"
local y = false or "value"      -- "value"
```

### String Concatenation
```lua
local str = "Hello" .. " " .. "World"
```

### Length
```lua
local len = #"Hello"     -- 5
local count = #{1, 2, 3} -- 3
```

## Control Structures

### If Statement
```lua
local age = 20

if age < 18 then
  print("Minor")
elseif age < 65 then
  print("Adult")
else
  print("Senior")
end

-- Ternary-like (using 'and'/'or')
local status = (age >= 18) and "Adult" or "Minor"
```

### Loops

**While Loop:**
```lua
local i = 1
while i <= 5 do
  print(i)
  i = i + 1
end
```

**Repeat-Until Loop:**
```lua
local i = 1
repeat
  print(i)
  i = i + 1
until i > 5
```

**For Loop (Numeric):**
```lua
-- for var = start, end, step
for i = 1, 5 do
  print(i)  -- 1, 2, 3, 4, 5
end

for i = 5, 1, -1 do
  print(i)  -- 5, 4, 3, 2, 1
end
```

**For Loop (Iterator):**
```lua
-- Array iteration
local fruits = { "apple", "banana", "orange" }
for index, value in ipairs(fruits) do
  print(index, value)
end

-- Table iteration
local person = { name = "John", age = 30 }
for key, value in pairs(person) do
  print(key, value)
end
```

**Break:**
```lua
for i = 1, 10 do
  if i == 5 then
    break
  end
  print(i)
end
```

Note: Lua doesn't have a `continue` statement. Use workarounds:
```lua
for i = 1, 10 do
  if i % 2 == 0 then
    goto continue
  end
  print(i)
  ::continue::
end
```

## Functions

### Basic Function
```lua
local function add(a, b)
  return a + b
end

local result = add(5, 3)  -- 8
```

### Multiple Returns
```lua
local function stats(a, b)
  return a + b, a - b, a * b
end

local sum, diff, product = stats(10, 5)
```

### Variable Arguments
```lua
local function sum(...)
  local args = {...}
  local total = 0
  for _, v in ipairs(args) do
    total = total + v
  end
  return total
end

print(sum(1, 2, 3, 4, 5))  -- 15
```

### Default Parameters (Manual)
```lua
local function greet(name)
  name = name or "Guest"
  return "Hello, " .. name
end

print(greet())        -- "Hello, Guest"
print(greet("John"))  -- "Hello, John"
```

### Anonymous Functions
```lua
local numbers = {1, 2, 3, 4, 5}
local doubled = {}

for i, v in ipairs(numbers) do
  doubled[i] = (function(x) return x * 2 end)(v)
end
```

### Closures
```lua
local function counter()
  local count = 0
  return function()
    count = count + 1
    return count
  end
end

local c = counter()
print(c())  -- 1
print(c())  -- 2
print(c())  -- 3
```

## Error Handling

```lua
-- pcall (protected call)
local success, result = pcall(function()
  -- Code that might error
  error("Something went wrong")
end)

if success then
  print("Success:", result)
else
  print("Error:", result)
end

-- assert
local function divide(a, b)
  assert(b ~= 0, "Division by zero")
  return a / b
end
```

## Modules

### Creating a Module
```lua
-- mymodule.lua
local M = {}

M.greet = function(name)
  return "Hello, " .. name
end

M.add = function(a, b)
  return a + b
end

return M
```

### Using a Module
```lua
local mymodule = require('mymodule')

print(mymodule.greet("Lua"))  -- "Hello, Lua"
print(mymodule.add(5, 3))     -- 8
```

## Important Gotchas

### 1. Arrays are 1-indexed
```lua
local arr = {"a", "b", "c"}
print(arr[1])  -- "a" (NOT arr[0])
```

### 2. Length operator with holes
```lua
local arr = {1, 2, nil, 4}
print(#arr)  -- Undefined behavior! Could be 2 or 4
```

### 3. Global by default
```lua
x = 10        -- Global (bad!)
local x = 10  -- Local (good!)
```

### 4. Not equal is ~=
```lua
if x ~= y then  -- Correct
  -- ...
end

if x != y then  -- SYNTAX ERROR!
  -- ...
end
```

### 5. 'nil' and 'false' are falsy, everything else is truthy
```lua
if 0 then       -- TRUE! (0 is truthy)
  print("yes")
end

if "" then      -- TRUE! (empty string is truthy)
  print("yes")
end

if nil then     -- FALSE
  print("no")
end

if false then   -- FALSE
  print("no")
end
```

## Next Steps

- Learn [Lua Tables in Depth](./02-lua-tables.md)
- Learn [Lua in Neovim](./03-lua-in-neovim.md)

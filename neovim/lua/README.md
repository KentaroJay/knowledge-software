# Lua in Neovim

A comprehensive guide to understanding Lua and its integration with Neovim.

## Contents

1. [Lua Basics](./lua-basics.md) - Fundamental Lua programming concepts
2. [Lua in Neovim](./lua-neovim.md) - How Lua integrates with Neovim
3. [Neovim Lua API](./neovim-api.md) - Using Neovim's Lua API
4. [Examples](./examples.md) - Practical Lua configuration examples

## Why Lua in Neovim?

### Advantages
- **Fast**: Lua is significantly faster than Vimscript
- **Modern**: Better language design and features
- **First-class support**: Neovim's core APIs are written in Lua
- **Rich ecosystem**: Access to LuaJIT and Lua libraries
- **Better tooling**: More IDE support for Lua than Vimscript

### When to Use Lua
- Plugin configuration (especially Lua-based plugins)
- Custom functions and complex logic
- Performance-critical operations
- New Neovim features (many are Lua-only)

### When to Use Vimscript
- Simple settings and mappings
- Backwards compatibility with Vim
- Legacy plugin configuration
- Quick one-liners

## Quick Comparison

### Setting Options

**Vimscript**:
```vim
set number
set tabstop=4
set expandtab
```

**Lua**:
```lua
vim.opt.number = true
vim.opt.tabstop = 4
vim.opt.expandtab = true
```

### Key Mappings

**Vimscript**:
```vim
nnoremap <leader>w :write<CR>
```

**Lua**:
```lua
vim.keymap.set('n', '<leader>w', ':write<CR>')
```

### Autocommands

**Vimscript**:
```vim
augroup MyGroup
  autocmd!
  autocmd BufWritePre * echo "Saving..."
augroup END
```

**Lua**:
```lua
local augroup = vim.api.nvim_create_augroup('MyGroup', { clear = true })
vim.api.nvim_create_autocmd('BufWritePre', {
  group = augroup,
  callback = function()
    print('Saving...')
  end,
})
```

## Using Lua in Your Config

### Method 1: Inline Lua in init.vim

```vim
lua <<EOF
vim.opt.number = true
print('Hello from Lua!')
EOF
```

### Method 2: Single Lua Commands

```vim
lua vim.opt.number = true
```

### Method 3: init.lua Instead of init.vim

Replace `~/.config/nvim/init.vim` with `~/.config/nvim/init.lua`:

```lua
-- All Lua configuration
vim.opt.number = true
vim.keymap.set('n', '<leader>w', ':write<CR>')
```

### Method 4: Require Lua Modules

Create `~/.config/nvim/lua/config/settings.lua`:
```lua
vim.opt.number = true
vim.opt.expandtab = true
```

Load it in `init.vim`:
```vim
lua require('config.settings')
```

Or in `init.lua`:
```lua
require('config.settings')
```

## Neovim Lua Namespaces

- `vim.api` - Neovim API functions (low-level)
- `vim.fn` - Vimscript functions
- `vim.opt` - Option handling (modern)
- `vim.o` - Get/set options (global)
- `vim.bo` - Buffer-local options
- `vim.wo` - Window-local options
- `vim.g` - Global variables
- `vim.b` - Buffer variables
- `vim.w` - Window variables
- `vim.t` - Tabpage variables
- `vim.keymap` - Keymapping functions
- `vim.cmd` - Execute Vimscript commands
- `vim.loop` - Libuv async I/O

## Learning Path

1. **Start here**: [Lua Basics](./lua-basics.md)
   - Learn fundamental Lua syntax and concepts

2. **Then**: [Lua in Neovim](./lua-neovim.md)
   - Understand how to use Lua within Neovim

3. **Next**: [Neovim API](./neovim-api.md)
   - Master Neovim's Lua API functions

4. **Practice**: [Examples](./examples.md)
   - Study real-world configuration examples

## Quick Reference

### Running Lua Commands

```vim
:lua print('Hello')           " Execute Lua code
:lua =vim.version()          " Print expression result
:lua vim.cmd('echo "Hi"')    " Run Vimscript from Lua
```

### Debugging

```lua
print(vim.inspect(variable))  -- Pretty-print tables
vim.notify('Message')         -- Show notification
```

## Resources

- [Neovim Lua Guide](https://neovim.io/doc/user/lua-guide.html)
- [Lua 5.1 Reference Manual](https://www.lua.org/manual/5.1/)
- [Learn Lua in Y Minutes](https://learnxinyminutes.com/docs/lua/)
- [Nanotee's Lua Guide](https://github.com/nanotee/nvim-lua-guide)

## Next Steps

Start with [Lua Basics](./lua-basics.md) to learn the language fundamentals.

# Lua in Neovim

How Lua integrates with Neovim and how to use it effectively in your configuration.

## Overview

Neovim embeds LuaJIT, providing fast script execution and access to Neovim's internal APIs. Lua can be used for configuration, plugins, and extending Neovim functionality.

## Using Lua in Neovim

### 1. init.lua - Full Lua Configuration

Replace `~/.config/nvim/init.vim` with `~/.config/nvim/init.lua`:

```lua
-- Set options
vim.opt.number = true
vim.opt.expandtab = true
vim.opt.shiftwidth = 2

-- Set keymaps
vim.keymap.set('n', '<leader>w', ':write<CR>')

-- Load other modules
require('plugins')
require('config')
```

### 2. Inline Lua in init.vim

Embed Lua code in your Vimscript config:

```vim
lua <<EOF
vim.opt.number = true
print('Hello from Lua!')
EOF
```

Single-line Lua:
```vim
lua vim.opt.number = true
```

### 3. Lua Files in lua/ Directory

Create modules in `~/.config/nvim/lua/`:

```
~/.config/nvim/
├── init.lua
└── lua/
    ├── config/
    │   ├── settings.lua
    │   ├── keymaps.lua
    │   └── autocmds.lua
    └── plugins/
        ├── init.lua
        ├── treesitter.lua
        └── lsp.lua
```

Load with `require()`:
```lua
require('config.settings')    -- Loads lua/config/settings.lua
require('plugins')            -- Loads lua/plugins/init.lua or lua/plugins.lua
```

**Important**:
- Use dots for path separators (not slashes)
- Omit `.lua` extension
- `init.lua` in a directory is loaded when requiring the directory name

## Executing Vimscript from Lua

### vim.cmd() - Execute Vimscript

```lua
-- Single command
vim.cmd('colorscheme tokyonight')

-- Multiple commands
vim.cmd([[
  syntax on
  set hlsearch
  highlight Comment guifg=#5C6370
]])

-- With string concatenation
local colorscheme = 'tokyonight'
vim.cmd('colorscheme ' .. colorscheme)
```

### vim.fn - Call Vimscript Functions

```lua
-- Call Vim functions
local has_node = vim.fn.executable('node')  -- 1 or 0
local cwd = vim.fn.getcwd()
local file_readable = vim.fn.filereadable('init.lua')

-- Expand paths
local config_path = vim.fn.stdpath('config')  -- ~/.config/nvim
local data_path = vim.fn.stdpath('data')      -- ~/.local/share/nvim

-- Get user input
local name = vim.fn.input('Enter name: ')
local choice = vim.fn.confirm('Continue?', '&Yes\n&No', 1)
```

### vim.api - Low-level API

```lua
-- Get buffer content
local lines = vim.api.nvim_buf_get_lines(0, 0, -1, false)

-- Set buffer content
vim.api.nvim_buf_set_lines(0, 0, -1, false, {'line 1', 'line 2'})

-- Execute Ex command
vim.api.nvim_command('edit file.txt')

-- Evaluate Vimscript expression
local result = vim.api.nvim_eval('2 + 2')
```

## Setting Options

Neovim provides multiple ways to set options in Lua.

### vim.opt (Preferred)

Most convenient way, handles conversions automatically:

```lua
-- Boolean options
vim.opt.number = true
vim.opt.relativenumber = true

-- String options
vim.opt.encoding = 'utf-8'
vim.opt.fileformat = 'unix'

-- Number options
vim.opt.tabstop = 4
vim.opt.shiftwidth = 4

-- List options (append/prepend/remove)
vim.opt.wildignore:append({ '*.pyc', '*.o' })
vim.opt.path:prepend('.')

-- Comma-separated options
vim.opt.shortmess:append('c')
vim.opt.whichwrap:append('<,>,[,]')
```

### vim.o, vim.go, vim.bo, vim.wo

Direct option access (less convenient):

```lua
-- vim.o - Global options (set)
vim.o.number = true
vim.o.tabstop = 4

-- vim.go - Global-only options (setglobal)
vim.go.background = 'dark'

-- vim.bo - Buffer-local options (setlocal for buffer)
vim.bo.expandtab = true
vim.bo.filetype = 'lua'

-- vim.wo - Window-local options (setlocal for window)
vim.wo.wrap = false
vim.wo.cursorline = true
```

### Equivalence Table

| Vimscript | Lua (vim.opt) | Lua (vim.o) |
|-----------|---------------|-------------|
| `set number` | `vim.opt.number = true` | `vim.o.number = true` |
| `set nonumber` | `vim.opt.number = false` | `vim.o.number = false` |
| `set tabstop=4` | `vim.opt.tabstop = 4` | `vim.o.tabstop = 4` |
| `set path+=.` | `vim.opt.path:append('.')` | `vim.o.path = vim.o.path .. ',.'` |

## Variables

### Global Variables

```lua
-- vim.g - Global variables (g:)
vim.g.mapleader = ' '
vim.g.loaded_netrw = 1
vim.g.python3_host_prog = '/usr/bin/python3'

-- Access in Vimscript: g:mapleader
```

### Buffer Variables

```lua
-- vim.b - Buffer variables (b:)
vim.b.current_syntax = 'lua'

-- For specific buffer
vim.b[bufnr].my_var = 'value'
```

### Window Variables

```lua
-- vim.w - Window variables (w:)
vim.w.my_window_var = true
```

### Tabpage Variables

```lua
-- vim.t - Tabpage variables (t:)
vim.t.my_tab_var = 'value'
```

### Vim Variables

```lua
-- vim.v - Vim variables (v:)
local count = vim.v.count       -- Count for current command
local error = vim.v.errmsg      -- Last error message
```

### Environment Variables

```lua
-- vim.env - Environment variables
local home = vim.env.HOME
local path = vim.env.PATH

-- Set environment variable
vim.env.MY_VAR = 'value'
```

## Keymaps

### vim.keymap.set() (Preferred)

Modern, flexible keymap API:

```lua
-- Basic mapping: mode, lhs, rhs
vim.keymap.set('n', '<leader>w', ':write<CR>')

-- Multiple modes
vim.keymap.set({'n', 'v'}, '<leader>y', '"+y')

-- With options
vim.keymap.set('n', '<leader>f', ':Files<CR>', {
  noremap = true,
  silent = true,
  desc = 'Find files',
})

-- Lua function as rhs
vim.keymap.set('n', '<leader>t', function()
  print('Hello from Lua!')
end, { desc = 'Test mapping' })

-- Buffer-local mapping
vim.keymap.set('n', 'K', vim.lsp.buf.hover, {
  buffer = 0,  -- 0 = current buffer
  desc = 'Show hover documentation',
})
```

### Common Options

```lua
{
  noremap = true,   -- Non-recursive (default: true)
  silent = true,    -- Don't echo command
  expr = false,     -- Expression mapping
  nowait = false,   -- Don't wait for more keys
  buffer = nil,     -- Buffer number (nil = global)
  desc = '',        -- Description (shown in which-key, etc.)
}
```

### Mode Shortcuts

| Mode | Shortcut | Description |
|------|----------|-------------|
| Normal | `'n'` | Normal mode |
| Insert | `'i'` | Insert mode |
| Visual | `'v'` | Visual and Select mode |
| Visual only | `'x'` | Visual mode only |
| Terminal | `'t'` | Terminal mode |
| Command | `'c'` | Command-line mode |
| Operator | `'o'` | Operator-pending mode |
| All | `''` | All modes |

### vim.api.nvim_set_keymap() (Old API)

Lower-level API, still works but less convenient:

```lua
vim.api.nvim_set_keymap('n', '<leader>w', ':write<CR>', {
  noremap = true,
  silent = true,
})
```

### Deleting Keymaps

```lua
-- Delete keymap
vim.keymap.del('n', '<leader>w')

-- Delete buffer-local keymap
vim.keymap.del('n', 'K', { buffer = 0 })
```

### Getting Keymap Info

```lua
-- Get keymaps for mode
local maps = vim.api.nvim_get_keymap('n')

-- Get buffer keymaps
local buf_maps = vim.api.nvim_buf_get_keymap(0, 'n')
```

## Autocommands

### vim.api.nvim_create_autocmd() (Modern API)

```lua
-- Single event, single pattern
vim.api.nvim_create_autocmd('BufWritePre', {
  pattern = '*.lua',
  callback = function()
    print('Saving Lua file...')
  end,
})

-- Multiple events
vim.api.nvim_create_autocmd({'BufEnter', 'BufWinEnter'}, {
  pattern = '*.py',
  callback = function()
    vim.opt_local.tabstop = 4
  end,
})

-- Command instead of callback
vim.api.nvim_create_autocmd('FileType', {
  pattern = 'lua',
  command = 'setlocal shiftwidth=2',
})

-- With group
local group = vim.api.nvim_create_augroup('MyGroup', { clear = true })

vim.api.nvim_create_autocmd('BufWritePre', {
  group = group,
  pattern = '*',
  callback = function()
    -- Remove trailing whitespace
    vim.cmd([[%s/\s\+$//e]])
  end,
})
```

### Autocommand Groups

```lua
-- Create group (clear existing)
local group = vim.api.nvim_create_augroup('MyLuaGroup', { clear = true })

-- Add autocmds to group
vim.api.nvim_create_autocmd('BufWritePre', {
  group = group,
  pattern = '*.lua',
  callback = function()
    vim.lsp.buf.format({ async = false })
  end,
})

vim.api.nvim_create_autocmd('BufWritePost', {
  group = group,
  pattern = '*.lua',
  command = 'source %',
})
```

### Common Events

```lua
-- Buffer events
'BufNewFile'      -- New file
'BufRead'         -- Read buffer
'BufEnter'        -- Enter buffer
'BufLeave'        -- Leave buffer
'BufWritePre'     -- Before writing
'BufWritePost'    -- After writing

-- File events
'FileType'        -- Filetype set
'FileReadPost'    -- After reading file

-- UI events
'VimEnter'        -- Vim started
'VimLeave'        -- Vim exiting
'CursorMoved'     -- Cursor moved
'CursorHold'      -- Cursor held still
'InsertEnter'     -- Enter insert mode
'InsertLeave'     -- Leave insert mode

-- LSP events
'LspAttach'       -- LSP attached to buffer
'LspDetach'       -- LSP detached
```

### Event Callback Details

```lua
vim.api.nvim_create_autocmd('BufEnter', {
  callback = function(args)
    print(vim.inspect(args))
    -- args contains:
    -- {
    --   buf = buffer number,
    --   file = filename,
    --   match = matched pattern,
    --   event = event name,
    -- }
  end,
})
```

### Deleting Autocommands

```lua
-- Clear all autocmds in group
vim.api.nvim_clear_autocmds({ group = 'MyGroup' })

-- Clear by event and pattern
vim.api.nvim_clear_autocmds({
  event = 'BufWritePre',
  pattern = '*.lua',
})
```

## User Commands

### vim.api.nvim_create_user_command()

```lua
-- Simple command
vim.api.nvim_create_user_command('Hello', function()
  print('Hello, World!')
end, {})

-- With arguments
vim.api.nvim_create_user_command('Greet', function(opts)
  print('Hello, ' .. opts.args)
end, { nargs = 1 })

-- With completion
vim.api.nvim_create_user_command('LoadPlugin', function(opts)
  require('plugins.' .. opts.args)
end, {
  nargs = 1,
  complete = function()
    return { 'treesitter', 'lsp', 'telescope' }
  end,
})

-- Range command
vim.api.nvim_create_user_command('SortLines', function(opts)
  vim.cmd(opts.line1 .. ',' .. opts.line2 .. 'sort')
end, { range = true })

-- Buffer-local command
vim.api.nvim_buf_create_user_command(0, 'BufferInfo', function()
  print('Buffer number: ' .. vim.api.nvim_get_current_buf())
end, {})
```

### Command Options

```lua
{
  nargs = 0,       -- Number of arguments: 0, 1, '+', '*', '?'
  complete = fn,   -- Completion function
  range = false,   -- Accept range (true, % , N)
  bang = false,    -- Accept bang (!)
  bar = false,     -- Command can be followed by |
  desc = '',       -- Description
}
```

## Highlights

### Define Highlight Groups

```lua
-- Set highlight
vim.api.nvim_set_hl(0, 'MyHighlight', {
  fg = '#FF0000',
  bg = '#000000',
  bold = true,
  italic = true,
})

-- Link highlight groups
vim.api.nvim_set_hl(0, 'MyComment', { link = 'Comment' })

-- Get highlight
local hl = vim.api.nvim_get_hl(0, { name = 'Normal' })
print(vim.inspect(hl))
```

### Highlight Attributes

```lua
{
  fg = '#RRGGBB',         -- Foreground color
  bg = '#RRGGBB',         -- Background color
  sp = '#RRGGBB',         -- Special color (underline)
  bold = true,
  italic = true,
  underline = true,
  undercurl = true,
  strikethrough = true,
  reverse = true,
  standout = true,
  link = 'GroupName',     -- Link to another group
}
```

## Working with Buffers, Windows, Tabs

### Buffers

```lua
-- Get current buffer
local buf = vim.api.nvim_get_current_buf()

-- Get all buffers
local bufs = vim.api.nvim_list_bufs()

-- Create new buffer
local new_buf = vim.api.nvim_create_buf(false, true)  -- (listed, scratch)

-- Get buffer lines
local lines = vim.api.nvim_buf_get_lines(buf, 0, -1, false)

-- Set buffer lines
vim.api.nvim_buf_set_lines(buf, 0, -1, false, {'line 1', 'line 2'})

-- Buffer option
vim.api.nvim_buf_set_option(buf, 'filetype', 'lua')

-- Delete buffer
vim.api.nvim_buf_delete(buf, { force = true })
```

### Windows

```lua
-- Get current window
local win = vim.api.nvim_get_current_win()

-- Get all windows
local wins = vim.api.nvim_list_wins()

-- Open new window
vim.cmd('split')
vim.cmd('vsplit')

-- Window option
vim.api.nvim_win_set_option(win, 'number', true)

-- Get/set cursor position
local pos = vim.api.nvim_win_get_cursor(win)  -- {row, col}
vim.api.nvim_win_set_cursor(win, {10, 5})

-- Get window buffer
local buf = vim.api.nvim_win_get_buf(win)

-- Set window buffer
vim.api.nvim_win_set_buf(win, buf)
```

### Tabs

```lua
-- Get current tab
local tab = vim.api.nvim_get_current_tabpage()

-- Get all tabs
local tabs = vim.api.nvim_list_tabpages()

-- Open new tab
vim.cmd('tabnew')

-- Tab variable
vim.api.nvim_tabpage_set_var(tab, 'my_var', 'value')
```

## Printing and Debugging

```lua
-- Print (converts to string)
print('Hello')
print(123)

-- Pretty-print tables
local config = { width = 80, height = 24 }
print(vim.inspect(config))

-- Vim notify (shows in :messages and as notification)
vim.notify('Operation complete', vim.log.levels.INFO)
vim.notify('Warning!', vim.log.levels.WARN)
vim.notify('Error!', vim.log.levels.ERROR)

-- Echo (shows in command line)
vim.cmd('echo "Hello"')
vim.api.nvim_echo({{'Hello', 'Normal'}}, false, {})

-- Error message
vim.api.nvim_err_writeln('Error occurred!')
```

## Scheduling and Deferring

```lua
-- Schedule callback for next event loop
vim.schedule(function()
  print('This runs in next iteration')
end)

-- Defer callback until safe
vim.defer_fn(function()
  print('Deferred action')
end, 100)  -- 100ms delay
```

## Best Practices

1. **Use `local`** - Always declare variables as local
2. **Use `vim.opt`** - Preferred way to set options
3. **Use `vim.keymap.set()`** - Modern keymap API
4. **Group autocommands** - Use `nvim_create_augroup`
5. **Error handling** - Use `pcall` for risky operations
6. **Schedule UI updates** - Use `vim.schedule` for UI changes
7. **Check before use** - Validate functions/plugins exist

## Example Configuration Structure

```lua
-- ~/.config/nvim/init.lua
require('config.options')
require('config.keymaps')
require('config.autocmds')
require('plugins')

-- ~/.config/nvim/lua/config/options.lua
local opt = vim.opt

opt.number = true
opt.relativenumber = true
opt.expandtab = true
opt.shiftwidth = 2
opt.tabstop = 2

-- ~/.config/nvim/lua/config/keymaps.lua
local keymap = vim.keymap.set

vim.g.mapleader = ' '

keymap('n', '<leader>w', ':write<CR>', { desc = 'Save file' })
keymap('n', '<leader>q', ':quit<CR>', { desc = 'Quit' })

-- ~/.config/nvim/lua/config/autocmds.lua
local autocmd = vim.api.nvim_create_autocmd
local augroup = vim.api.nvim_create_augroup

local group = augroup('MyAutocommands', { clear = true })

autocmd('BufWritePre', {
  group = group,
  pattern = '*.lua',
  callback = function()
    vim.lsp.buf.format({ async = false })
  end,
})
```

## Next Steps

- Study [Neovim Lua API](./neovim-api.md) for detailed API reference
- Explore [Examples](./examples.md) for real-world configurations
- Review [Lua Basics](./lua-basics.md) if needed

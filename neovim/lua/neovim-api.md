# Neovim Lua API Reference

Comprehensive reference for Neovim's Lua API.

## API Namespaces

Neovim organizes its Lua API into several namespaces:

- **vim.api** - Core Neovim API (nvim_* functions)
- **vim.fn** - Vimscript functions
- **vim.opt** - Option handling
- **vim.keymap** - Keymap management
- **vim.loop** - Libuv event loop (async I/O)
- **vim.lsp** - LSP client
- **vim.treesitter** - Tree-sitter integration
- **vim.diagnostic** - Diagnostic handling

## vim.api - Core API

### Buffer Functions

```lua
-- Get/create buffers
vim.api.nvim_get_current_buf()                    -- Get current buffer number
vim.api.nvim_list_bufs()                          -- List all buffers
vim.api.nvim_create_buf(listed, scratch)          -- Create new buffer
vim.api.nvim_buf_is_loaded(buffer)                -- Check if buffer is loaded
vim.api.nvim_buf_is_valid(buffer)                 -- Check if buffer is valid
vim.api.nvim_buf_delete(buffer, {force=true})     -- Delete buffer

-- Buffer content
vim.api.nvim_buf_get_lines(buffer, start, end, strict)      -- Get lines
vim.api.nvim_buf_set_lines(buffer, start, end, strict, lines) -- Set lines
vim.api.nvim_buf_get_text(buffer, start_row, start_col, end_row, end_col, {}) -- Get text
vim.api.nvim_buf_set_text(buffer, start_row, start_col, end_row, end_col, lines) -- Set text
vim.api.nvim_buf_line_count(buffer)               -- Get line count

-- Buffer info
vim.api.nvim_buf_get_name(buffer)                 -- Get buffer filename
vim.api.nvim_buf_set_name(buffer, name)           -- Set buffer filename
vim.api.nvim_buf_get_offset(buffer, line)         -- Get byte offset of line

-- Buffer options
vim.api.nvim_buf_get_option(buffer, name)         -- Get buffer option
vim.api.nvim_buf_set_option(buffer, name, value)  -- Set buffer option

-- Buffer variables
vim.api.nvim_buf_get_var(buffer, name)            -- Get buffer variable
vim.api.nvim_buf_set_var(buffer, name, value)     -- Set buffer variable
vim.api.nvim_buf_del_var(buffer, name)            -- Delete buffer variable

-- Buffer marks
vim.api.nvim_buf_get_mark(buffer, name)           -- Get mark position
vim.api.nvim_buf_set_mark(buffer, name, line, col, {}) -- Set mark

-- Buffer keymaps
vim.api.nvim_buf_get_keymap(buffer, mode)         -- Get buffer keymaps
vim.api.nvim_buf_set_keymap(buffer, mode, lhs, rhs, opts) -- Set buffer keymap
vim.api.nvim_buf_del_keymap(buffer, mode, lhs)    -- Delete buffer keymap

-- User commands
vim.api.nvim_buf_create_user_command(buffer, name, command, opts) -- Create buffer command
vim.api.nvim_buf_del_user_command(buffer, name)   -- Delete buffer command

-- Attach/detach
vim.api.nvim_buf_attach(buffer, send_buffer, opts) -- Attach to buffer events
```

### Window Functions

```lua
-- Get/create windows
vim.api.nvim_get_current_win()                    -- Get current window
vim.api.nvim_list_wins()                          -- List all windows
vim.api.nvim_open_win(buffer, enter, config)      -- Open floating window
vim.api.nvim_win_is_valid(window)                 -- Check if window is valid
vim.api.nvim_win_close(window, force)             -- Close window

-- Window info
vim.api.nvim_win_get_buf(window)                  -- Get window buffer
vim.api.nvim_win_set_buf(window, buffer)          -- Set window buffer
vim.api.nvim_win_get_height(window)               -- Get window height
vim.api.nvim_win_set_height(window, height)       -- Set window height
vim.api.nvim_win_get_width(window)                -- Get window width
vim.api.nvim_win_set_width(window, width)         -- Set window width
vim.api.nvim_win_get_position(window)             -- Get window position

-- Cursor
vim.api.nvim_win_get_cursor(window)               -- Get cursor position {row, col}
vim.api.nvim_win_set_cursor(window, {row, col})   -- Set cursor position

-- Window options
vim.api.nvim_win_get_option(window, name)         -- Get window option
vim.api.nvim_win_set_option(window, name, value)  -- Set window option

-- Window variables
vim.api.nvim_win_get_var(window, name)            -- Get window variable
vim.api.nvim_win_set_var(window, name, value)     -- Set window variable
vim.api.nvim_win_del_var(window, name)            -- Delete window variable

-- Floating window config
vim.api.nvim_win_get_config(window)               -- Get floating win config
vim.api.nvim_win_set_config(window, config)       -- Set floating win config
```

### Tabpage Functions

```lua
vim.api.nvim_get_current_tabpage()                -- Get current tab
vim.api.nvim_list_tabpages()                      -- List all tabs
vim.api.nvim_tabpage_is_valid(tabpage)            -- Check if tab is valid
vim.api.nvim_tabpage_get_number(tabpage)          -- Get tab number
vim.api.nvim_tabpage_list_wins(tabpage)           -- List windows in tab
vim.api.nvim_tabpage_get_win(tabpage)             -- Get current window in tab
vim.api.nvim_tabpage_get_var(tabpage, name)       -- Get tab variable
vim.api.nvim_tabpage_set_var(tabpage, name, value) -- Set tab variable
vim.api.nvim_tabpage_del_var(tabpage, name)       -- Delete tab variable
```

### UI Functions

```lua
-- Input/Output
vim.api.nvim_out_write(str)                       -- Write to stdout
vim.api.nvim_err_write(str)                       -- Write to stderr
vim.api.nvim_err_writeln(str)                     -- Write line to stderr

-- Echo
vim.api.nvim_echo(chunks, history, opts)          -- Echo message
vim.api.nvim_notify(msg, log_level, opts)         -- Show notification

-- UI info
vim.api.nvim_list_uis()                           -- List attached UIs
vim.api.nvim_ui_attach(width, height, options)    -- Attach UI
vim.api.nvim_ui_detach()                          -- Detach UI

-- Input
vim.api.nvim_input(keys)                          -- Send input keys
vim.api.nvim_feedkeys(keys, mode, escape_csi)     -- Feed keys
```

### Command Functions

```lua
-- Execute commands
vim.api.nvim_command(command)                     -- Execute Ex command
vim.api.nvim_exec(src, output)                    -- Execute Vimscript
vim.api.nvim_eval(expr)                           -- Evaluate Vimscript expression
vim.api.nvim_call_function(fn, args)              -- Call Vimscript function

-- Parse commands
vim.api.nvim_parse_cmd(str, opts)                 -- Parse Ex command
vim.api.nvim_cmd(cmd, opts)                       -- Execute parsed command

-- User commands
vim.api.nvim_create_user_command(name, command, opts) -- Create user command
vim.api.nvim_del_user_command(name)               -- Delete user command
vim.api.nvim_get_commands(opts)                   -- Get commands
```

### Option Functions

```lua
vim.api.nvim_get_option(name)                     -- Get global option
vim.api.nvim_set_option(name, value)              -- Set global option
vim.api.nvim_get_all_options_info()               -- Get all options info
vim.api.nvim_get_option_info(name)                -- Get option info
```

### Variable Functions

```lua
vim.api.nvim_get_var(name)                        -- Get global variable
vim.api.nvim_set_var(name, value)                 -- Set global variable
vim.api.nvim_del_var(name)                        -- Delete global variable
```

### Keymap Functions

```lua
vim.api.nvim_set_keymap(mode, lhs, rhs, opts)     -- Set global keymap
vim.api.nvim_del_keymap(mode, lhs)                -- Delete global keymap
vim.api.nvim_get_keymap(mode)                     -- Get keymaps for mode
```

### Autocommand Functions

```lua
vim.api.nvim_create_autocmd(event, opts)          -- Create autocommand
vim.api.nvim_del_autocmd(id)                      -- Delete autocommand
vim.api.nvim_clear_autocmds(opts)                 -- Clear autocommands
vim.api.nvim_get_autocmds(opts)                   -- Get autocommands

-- Autocommand groups
vim.api.nvim_create_augroup(name, opts)           -- Create autocommand group
vim.api.nvim_del_augroup_by_id(id)                -- Delete group by ID
vim.api.nvim_del_augroup_by_name(name)            -- Delete group by name
```

### Highlight Functions

```lua
vim.api.nvim_get_hl(ns_id, opts)                  -- Get highlights
vim.api.nvim_set_hl(ns_id, name, val)             -- Set highlight
vim.api.nvim_get_hl_by_id(hl_id, rgb)             -- Get highlight by ID (deprecated)
vim.api.nvim_get_hl_by_name(name, rgb)            -- Get highlight by name (deprecated)
```

### Namespace Functions

```lua
vim.api.nvim_create_namespace(name)               -- Create namespace
vim.api.nvim_get_namespaces()                     -- Get all namespaces
```

### Virtual Text / Extmarks

```lua
-- Set virtual text (extmark)
vim.api.nvim_buf_set_extmark(buffer, ns_id, line, col, opts)
vim.api.nvim_buf_get_extmark_by_id(buffer, ns_id, id, opts)
vim.api.nvim_buf_get_extmarks(buffer, ns_id, start, end, opts)
vim.api.nvim_buf_del_extmark(buffer, ns_id, id)
vim.api.nvim_buf_clear_namespace(buffer, ns_id, line_start, line_end)
```

### Miscellaneous

```lua
vim.api.nvim_get_runtime_file(name, all)          -- Get runtime files
vim.api.nvim_get_mode()                           -- Get current mode
vim.api.nvim_get_context(opts)                    -- Get context
vim.api.nvim_load_context(dict)                   -- Load context
```

## vim.fn - Vimscript Functions

Access any Vimscript function through `vim.fn`:

```lua
-- File operations
vim.fn.expand('%')                  -- Current filename
vim.fn.expand('%:p')                -- Full path
vim.fn.expand('%:t')                -- Filename only
vim.fn.expand('%:h')                -- Directory
vim.fn.fnamemodify(file, mods)      -- Modify filename

vim.fn.filereadable(file)           -- Check if file is readable
vim.fn.isdirectory(path)            -- Check if path is directory
vim.fn.mkdir(path, 'p')             -- Create directory
vim.fn.delete(file)                 -- Delete file

-- Path
vim.fn.getcwd()                     -- Get current directory
vim.fn.chdir(path)                  -- Change directory
vim.fn.stdpath('config')            -- ~/.config/nvim
vim.fn.stdpath('data')              -- ~/.local/share/nvim
vim.fn.stdpath('cache')             -- ~/.cache/nvim

-- System
vim.fn.executable(name)             -- Check if executable exists
vim.fn.system(cmd)                  -- Execute system command
vim.fn.systemlist(cmd)              -- Execute and return list

-- Buffer/Window
vim.fn.bufnr('%')                   -- Current buffer number
vim.fn.bufname('%')                 -- Current buffer name
vim.fn.winnr()                      -- Current window number
vim.fn.line('.')                    -- Current line number
vim.fn.col('.')                     -- Current column number
vim.fn.getline(line)                -- Get line content
vim.fn.setline(line, text)          -- Set line content

-- Search
vim.fn.search(pattern, flags)       -- Search for pattern
vim.fn.searchpos(pattern, flags)    -- Search and return position
vim.fn.match(str, pattern)          -- Match string against pattern
vim.fn.matchstr(str, pattern)       -- Extract match from string

-- Lists
vim.fn.len(list)                    -- Length of list
vim.fn.empty(var)                   -- Check if empty
vim.fn.add(list, item)              -- Add item to list
vim.fn.extend(list1, list2)         -- Extend list
vim.fn.index(list, item)            -- Find index of item
vim.fn.join(list, sep)              -- Join list with separator

-- Strings
vim.fn.toupper(str)                 -- Convert to uppercase
vim.fn.tolower(str)                 -- Convert to lowercase
vim.fn.trim(str)                    -- Trim whitespace
vim.fn.split(str, sep)              -- Split string
vim.fn.substitute(str, pat, rep, flags) -- Substitute in string

-- Input
vim.fn.input(prompt, default)       -- Get user input
vim.fn.confirm(msg, choices, default) -- Show confirmation dialog
vim.fn.getchar()                    -- Get single character

-- Time
vim.fn.strftime(format)             -- Format current time
vim.fn.localtime()                  -- Current Unix timestamp

-- Misc
vim.fn.has(feature)                 -- Check for feature
vim.fn.exists(var)                  -- Check if variable exists
vim.fn.eval(expr)                   -- Evaluate expression
```

## vim.opt - Option Management

```lua
-- Get option value
local value = vim.opt.number:get()

-- Set option
vim.opt.number = true
vim.opt.tabstop = 4

-- Append to list/string option
vim.opt.wildignore:append('*.pyc')
vim.opt.path:append('/usr/local/include')

-- Prepend to list/string option
vim.opt.path:prepend('.')

-- Remove from list/string option
vim.opt.wildignore:remove('*.o')

-- Common options
vim.opt.number = true                    -- Line numbers
vim.opt.relativenumber = true            -- Relative line numbers
vim.opt.tabstop = 4                      -- Tab width
vim.opt.shiftwidth = 4                   -- Indent width
vim.opt.expandtab = true                 -- Use spaces instead of tabs
vim.opt.smartindent = true               -- Smart indentation
vim.opt.wrap = false                     -- Don't wrap lines
vim.opt.swapfile = false                 -- No swap file
vim.opt.backup = false                   -- No backup file
vim.opt.undofile = true                  -- Persistent undo
vim.opt.hlsearch = true                  -- Highlight search
vim.opt.incsearch = true                 -- Incremental search
vim.opt.ignorecase = true                -- Case-insensitive search
vim.opt.smartcase = true                 -- Smart case
vim.opt.termguicolors = true             -- True color support
vim.opt.signcolumn = 'yes'               -- Always show sign column
vim.opt.updatetime = 300                 -- Faster completion
vim.opt.timeoutlen = 500                 -- Key timeout
vim.opt.clipboard = 'unnamedplus'        -- System clipboard
vim.opt.splitright = true                -- Split right
vim.opt.splitbelow = true                -- Split below
```

## vim.keymap - Keymap Management

```lua
-- Set keymap
vim.keymap.set(mode, lhs, rhs, opts)

-- Examples
vim.keymap.set('n', '<leader>w', ':write<CR>', { desc = 'Save file' })
vim.keymap.set({'n', 'v'}, '<leader>y', '"+y', { desc = 'Copy to clipboard' })
vim.keymap.set('n', '<leader>f', function()
  print('Custom function')
end, { desc = 'Run function' })

-- Delete keymap
vim.keymap.del(mode, lhs, opts)
vim.keymap.del('n', '<leader>w')

-- Common options
{
  noremap = true,      -- Non-recursive (default)
  silent = true,       -- Silent
  expr = false,        -- Expression mapping
  buffer = nil,        -- Buffer number (nil = global)
  nowait = false,      -- Don't wait for more keys
  desc = '',           -- Description
}
```

## vim.loop - Libuv Event Loop

Async I/O operations using libuv:

```lua
-- File system
local uv = vim.loop

-- Read file
uv.fs_open(path, 'r', 438, function(err, fd)
  if err then
    print('Error:', err)
    return
  end

  uv.fs_read(fd, 1024, 0, function(err, data)
    if err then
      print('Error:', err)
      return
    end
    print('Content:', data)
    uv.fs_close(fd)
  end)
end)

-- Write file
uv.fs_open(path, 'w', 438, function(err, fd)
  uv.fs_write(fd, 'Hello, World!', 0, function(err)
    uv.fs_close(fd)
  end)
end)

-- Check file stats
uv.fs_stat(path, function(err, stat)
  if err then return end
  print('Size:', stat.size)
  print('Modified:', stat.mtime.sec)
end)

-- Timers
local timer = uv.new_timer()
timer:start(1000, 1000, function()  -- delay, repeat, callback
  print('Timer tick')
end)
timer:stop()
timer:close()

-- Process
local handle, pid = uv.spawn('ls', {
  args = {'-la'},
  stdio = {nil, pipe, nil}
}, function(code, signal)
  print('Exit code:', code)
end)
```

## vim.lsp - LSP Client

```lua
-- Start LSP client
vim.lsp.start({
  name = 'my-lsp',
  cmd = {'language-server'},
  root_dir = vim.fn.getcwd(),
})

-- LSP actions
vim.lsp.buf.definition()            -- Go to definition
vim.lsp.buf.references()            -- Find references
vim.lsp.buf.hover()                 -- Show hover
vim.lsp.buf.signature_help()        -- Show signature
vim.lsp.buf.rename()                -- Rename symbol
vim.lsp.buf.code_action()           -- Code actions
vim.lsp.buf.format({ async = true }) -- Format document

-- Diagnostics
vim.lsp.diagnostic.get_all()        -- Get all diagnostics
vim.lsp.diagnostic.get_line_diagnostics() -- Current line
vim.diagnostic.goto_next()          -- Next diagnostic
vim.diagnostic.goto_prev()          -- Previous diagnostic
vim.diagnostic.open_float()         -- Show in floating window

-- Configuration
vim.lsp.handlers['textDocument/hover'] = vim.lsp.with(
  vim.lsp.handlers.hover, {
    border = 'rounded',
  }
)
```

## vim.treesitter - Tree-sitter

```lua
-- Get parser
local parser = vim.treesitter.get_parser(bufnr, 'lua')

-- Parse buffer
local tree = parser:parse()[1]
local root = tree:root()

-- Query
local query = vim.treesitter.query.parse('lua', [[
  (function_declaration name: (identifier) @function.name)
]])

for id, node in query:iter_captures(root, bufnr) do
  local name = vim.treesitter.get_node_text(node, bufnr)
  print('Function:', name)
end

-- Highlighting
vim.treesitter.start(bufnr, 'lua')
vim.treesitter.stop(bufnr)

-- Node info
local node = vim.treesitter.get_node()  -- Node at cursor
print('Type:', node:type())
print('Text:', vim.treesitter.get_node_text(node, 0))
```

## Utility Functions

```lua
-- vim.inspect - Pretty print tables
print(vim.inspect({ foo = 'bar', baz = 123 }))

-- vim.notify - Show notification
vim.notify('Message', vim.log.levels.INFO)

-- Log levels
vim.log.levels.DEBUG
vim.log.levels.INFO
vim.log.levels.WARN
vim.log.levels.ERROR

-- vim.schedule - Schedule function
vim.schedule(function()
  print('Scheduled')
end)

-- vim.defer_fn - Defer function with delay
vim.defer_fn(function()
  print('Deferred')
end, 1000)  -- 1 second

-- vim.split - Split string
local parts = vim.split('a,b,c', ',')

-- vim.trim - Trim whitespace
local trimmed = vim.trim('  hello  ')

-- vim.tbl_* - Table utilities
vim.tbl_extend('force', t1, t2)    -- Merge tables
vim.tbl_deep_extend('force', t1, t2) -- Deep merge
vim.tbl_keys(table)                -- Get keys
vim.tbl_values(table)              -- Get values
vim.tbl_count(table)               -- Count entries
vim.tbl_isempty(table)             -- Check if empty
vim.tbl_contains(table, value)     -- Check if contains

-- vim.validate - Validate arguments
vim.validate({
  arg1 = {value, 'string'},
  arg2 = {value, 'number', true},  -- Optional
})

-- vim.deepcopy - Deep copy table
local copy = vim.deepcopy(original)
```

## Next Steps

- Practice with [Examples](./examples.md)
- Review [Lua in Neovim](./lua-neovim.md) for integration details
- Check [official API docs](https://neovim.io/doc/user/api.html)

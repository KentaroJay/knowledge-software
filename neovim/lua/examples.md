# Neovim Lua Configuration Examples

Practical examples of Lua configurations for common use cases.

## Basic Configuration Structure

### Recommended Directory Structure

```
~/.config/nvim/
├── init.lua                    # Main entry point
├── lua/
│   ├── config/
│   │   ├── init.lua           # Load all config modules
│   │   ├── options.lua        # Neovim options
│   │   ├── keymaps.lua        # Keymaps
│   │   ├── autocmds.lua       # Autocommands
│   │   └── lazy.lua           # Plugin manager setup
│   ├── plugins/
│   │   ├── init.lua           # Plugin specifications
│   │   ├── treesitter.lua     # TreeSitter config
│   │   ├── lsp.lua            # LSP config
│   │   └── telescope.lua      # Telescope config
│   └── utils/
│       └── init.lua           # Utility functions
```

### Main Entry Point (init.lua)

```lua
-- ~/.config/nvim/init.lua

-- Set leader key before loading plugins
vim.g.mapleader = ' '
vim.g.maplocalleader = ' '

-- Load configuration modules
require('config')
```

### Config Module Loader (lua/config/init.lua)

```lua
-- ~/.config/nvim/lua/config/init.lua

-- Load all configuration modules in order
require('config.options')
require('config.keymaps')
require('config.autocmds')
require('config.lazy')  -- Plugin manager
```

## Options Configuration

### lua/config/options.lua

```lua
local opt = vim.opt

-- Line numbers
opt.number = true
opt.relativenumber = true

-- Tabs & indentation
opt.tabstop = 2
opt.shiftwidth = 2
opt.expandtab = true
opt.autoindent = true
opt.smartindent = true

-- Line wrapping
opt.wrap = false
opt.linebreak = true  -- Break at word boundaries

-- Search
opt.ignorecase = true
opt.smartcase = true
opt.hlsearch = true
opt.incsearch = true

-- Appearance
opt.termguicolors = true
opt.background = 'dark'
opt.signcolumn = 'yes'
opt.cursorline = true
opt.colorcolumn = '80'

-- Behavior
opt.mouse = 'a'
opt.clipboard = 'unnamedplus'  -- Use system clipboard
opt.completeopt = 'menu,menuone,noselect'
opt.updatetime = 300
opt.timeoutlen = 500

-- Splits
opt.splitright = true
opt.splitbelow = true

-- Files
opt.fileencoding = 'utf-8'
opt.backup = false
opt.writebackup = false
opt.swapfile = false
opt.undofile = true
opt.undodir = vim.fn.stdpath('cache') .. '/undo'

-- Folding (with TreeSitter)
opt.foldmethod = 'expr'
opt.foldexpr = 'nvim_treesitter#foldexpr()'
opt.foldenable = false  -- Don't fold by default

-- Performance
opt.lazyredraw = false
opt.ttyfast = true

-- Miscellaneous
opt.showmode = false  -- Don't show mode (status line shows it)
opt.scrolloff = 8     -- Keep 8 lines above/below cursor
opt.sidescrolloff = 8
opt.iskeyword:append('-')  -- Treat dash as part of word
```

## Keymaps Configuration

### lua/config/keymaps.lua

```lua
local keymap = vim.keymap.set
local opts = { noremap = true, silent = true }

-- Leader key (set in init.lua before loading plugins)
-- vim.g.mapleader = ' '

-- General
keymap('n', '<leader>w', ':write<CR>', { desc = 'Save file' })
keymap('n', '<leader>q', ':quit<CR>', { desc = 'Quit' })
keymap('n', '<leader>x', ':wq<CR>', { desc = 'Save and quit' })
keymap('n', '<Esc><Esc>', ':nohlsearch<CR>', { desc = 'Clear search highlight' })

-- Better window navigation
keymap('n', '<C-h>', '<C-w>h', { desc = 'Go to left window' })
keymap('n', '<C-j>', '<C-w>j', { desc = 'Go to lower window' })
keymap('n', '<C-k>', '<C-w>k', { desc = 'Go to upper window' })
keymap('n', '<C-l>', '<C-w>l', { desc = 'Go to right window' })

-- Resize windows
keymap('n', '<C-Up>', ':resize -2<CR>', opts)
keymap('n', '<C-Down>', ':resize +2<CR>', opts)
keymap('n', '<C-Left>', ':vertical resize -2<CR>', opts)
keymap('n', '<C-Right>', ':vertical resize +2<CR>', opts)

-- Buffer navigation
keymap('n', '<S-l>', ':bnext<CR>', { desc = 'Next buffer' })
keymap('n', '<S-h>', ':bprevious<CR>', { desc = 'Previous buffer' })
keymap('n', '<leader>bd', ':bdelete<CR>', { desc = 'Delete buffer' })

-- Tab navigation
keymap('n', '<leader>tn', ':tabnew<CR>', { desc = 'New tab' })
keymap('n', '<leader>tc', ':tabclose<CR>', { desc = 'Close tab' })
keymap('n', '<leader>to', ':tabonly<CR>', { desc = 'Only this tab' })

-- Move text up and down
keymap('n', '<A-j>', ':m .+1<CR>==', { desc = 'Move line down' })
keymap('n', '<A-k>', ':m .-2<CR>==', { desc = 'Move line up' })
keymap('v', '<A-j>', ":m '>+1<CR>gv=gv", { desc = 'Move selection down' })
keymap('v', '<A-k>', ":m '<-2<CR>gv=gv", { desc = 'Move selection up' })

-- Indentation
keymap('v', '<', '<gv', { desc = 'Indent left' })
keymap('v', '>', '>gv', { desc = 'Indent right' })

-- Better paste
keymap('v', 'p', '"_dP', { desc = 'Paste without yanking' })

-- System clipboard
keymap({'n', 'v'}, '<leader>y', '"+y', { desc = 'Copy to clipboard' })
keymap({'n', 'v'}, '<leader>Y', '"+Y', { desc = 'Copy line to clipboard' })
keymap({'n', 'v'}, '<leader>p', '"+p', { desc = 'Paste from clipboard' })

-- Quick command mode
keymap('n', ';', ':', { desc = 'Enter command mode' })

-- Select all
keymap('n', '<C-a>', 'ggVG', { desc = 'Select all' })

-- Center screen after movements
keymap('n', '<C-d>', '<C-d>zz', { desc = 'Scroll down and center' })
keymap('n', '<C-u>', '<C-u>zz', { desc = 'Scroll up and center' })
keymap('n', 'n', 'nzzzv', { desc = 'Next search result and center' })
keymap('n', 'N', 'Nzzzv', { desc = 'Previous search result and center' })

-- Terminal mode
keymap('t', '<Esc>', '<C-\\><C-n>', { desc = 'Exit terminal mode' })
keymap('t', '<C-h>', '<C-\\><C-n><C-w>h', opts)
keymap('t', '<C-j>', '<C-\\><C-n><C-w>j', opts)
keymap('t', '<C-k>', '<C-\\><C-n><C-w>k', opts)
keymap('t', '<C-l>', '<C-\\><C-n><C-w>l', opts)

-- Quick fix list
keymap('n', '<leader>co', ':copen<CR>', { desc = 'Open quickfix' })
keymap('n', '<leader>cc', ':cclose<CR>', { desc = 'Close quickfix' })
keymap('n', '[q', ':cprev<CR>', { desc = 'Previous quickfix' })
keymap('n', ']q', ':cnext<CR>', { desc = 'Next quickfix' })
```

## Autocommands Configuration

### lua/config/autocmds.lua

```lua
local autocmd = vim.api.nvim_create_autocmd
local augroup = vim.api.nvim_create_augroup

-- General settings group
local general = augroup('General', { clear = true })

-- Highlight on yank
autocmd('TextYankPost', {
  group = general,
  pattern = '*',
  callback = function()
    vim.highlight.on_yank({ higroup = 'IncSearch', timeout = 200 })
  end,
  desc = 'Highlight yanked text',
})

-- Remove trailing whitespace on save
autocmd('BufWritePre', {
  group = general,
  pattern = '*',
  command = [[%s/\s\+$//e]],
  desc = 'Remove trailing whitespace',
})

-- Auto-create parent directories on save
autocmd('BufWritePre', {
  group = general,
  pattern = '*',
  callback = function(args)
    local dir = vim.fn.fnamemodify(args.file, ':h')
    if vim.fn.isdirectory(dir) == 0 then
      vim.fn.mkdir(dir, 'p')
    end
  end,
  desc = 'Auto-create parent directories',
})

-- Close some filetypes with 'q'
autocmd('FileType', {
  group = general,
  pattern = {
    'qf',
    'help',
    'man',
    'lspinfo',
    'checkhealth',
  },
  callback = function(event)
    vim.bo[event.buf].buflisted = false
    vim.keymap.set('n', 'q', '<cmd>close<cr>', { buffer = event.buf, silent = true })
  end,
  desc = 'Close with q',
})

-- Go to last location when opening a buffer
autocmd('BufReadPost', {
  group = general,
  callback = function(event)
    local exclude = { 'gitcommit' }
    local buf = event.buf
    if vim.tbl_contains(exclude, vim.bo[buf].filetype) or vim.b[buf].last_pos then
      return
    end
    vim.b[buf].last_pos = true
    local mark = vim.api.nvim_buf_get_mark(buf, '"')
    local lcount = vim.api.nvim_buf_line_count(buf)
    if mark[1] > 0 and mark[1] <= lcount then
      pcall(vim.api.nvim_win_set_cursor, 0, mark)
    end
  end,
  desc = 'Go to last location',
})

-- File type specific settings
local filetype = augroup('FileType', { clear = true })

autocmd('FileType', {
  group = filetype,
  pattern = 'lua',
  callback = function()
    vim.opt_local.shiftwidth = 2
    vim.opt_local.tabstop = 2
  end,
})

autocmd('FileType', {
  group = filetype,
  pattern = {'python'},
  callback = function()
    vim.opt_local.shiftwidth = 4
    vim.opt_local.tabstop = 4
  end,
})

autocmd('FileType', {
  group = filetype,
  pattern = {'markdown', 'text'},
  callback = function()
    vim.opt_local.wrap = true
    vim.opt_local.spell = true
  end,
})

-- LSP specific
local lsp_group = augroup('LspFormat', { clear = true })

autocmd('BufWritePre', {
  group = lsp_group,
  pattern = {'*.lua', '*.py', '*.js', '*.ts', '*.tsx', '*.jsx'},
  callback = function()
    vim.lsp.buf.format({ async = false })
  end,
  desc = 'Format on save',
})

-- Terminal settings
local terminal = augroup('Terminal', { clear = true })

autocmd('TermOpen', {
  group = terminal,
  callback = function()
    vim.opt_local.number = false
    vim.opt_local.relativenumber = false
    vim.opt_local.signcolumn = 'no'
    vim.cmd('startinsert')
  end,
  desc = 'Terminal settings',
})
```

## Plugin Configuration Examples

### TreeSitter Configuration

```lua
-- lua/plugins/treesitter.lua

return {
  'nvim-treesitter/nvim-treesitter',
  build = ':TSUpdate',
  event = { 'BufReadPost', 'BufNewFile' },
  dependencies = {
    'nvim-treesitter/nvim-treesitter-textobjects',
  },
  config = function()
    require('nvim-treesitter.configs').setup({
      ensure_installed = {
        'lua', 'vim', 'vimdoc',
        'python', 'javascript', 'typescript',
        'html', 'css', 'json', 'yaml',
        'bash', 'markdown', 'markdown_inline',
      },
      auto_install = true,
      highlight = {
        enable = true,
        additional_vim_regex_highlighting = false,
      },
      indent = {
        enable = true,
      },
      incremental_selection = {
        enable = true,
        keymaps = {
          init_selection = '<CR>',
          node_incremental = '<CR>',
          scope_incremental = '<S-CR>',
          node_decremental = '<BS>',
        },
      },
      textobjects = {
        select = {
          enable = true,
          lookahead = true,
          keymaps = {
            ['af'] = '@function.outer',
            ['if'] = '@function.inner',
            ['ac'] = '@class.outer',
            ['ic'] = '@class.inner',
          },
        },
        move = {
          enable = true,
          set_jumps = true,
          goto_next_start = {
            [']m'] = '@function.outer',
            [']]'] = '@class.outer',
          },
          goto_next_end = {
            [']M'] = '@function.outer',
            [']['] = '@class.outer',
          },
          goto_previous_start = {
            ['[m'] = '@function.outer',
            ['[['] = '@class.outer',
          },
          goto_previous_end = {
            ['[M'] = '@function.outer',
            ['[]'] = '@class.outer',
          },
        },
      },
    })
  end,
}
```

### LSP Configuration

```lua
-- lua/plugins/lsp.lua

return {
  'neovim/nvim-lspconfig',
  event = { 'BufReadPre', 'BufNewFile' },
  dependencies = {
    'williamboman/mason.nvim',
    'williamboman/mason-lspconfig.nvim',
    'hrsh7th/cmp-nvim-lsp',
  },
  config = function()
    -- Setup mason
    require('mason').setup()
    require('mason-lspconfig').setup({
      ensure_installed = {
        'lua_ls',
        'pyright',
        'tsserver',
      },
    })

    -- LSP keymaps
    local on_attach = function(client, bufnr)
      local opts = { buffer = bufnr, silent = true }

      vim.keymap.set('n', 'gd', vim.lsp.buf.definition,
        vim.tbl_extend('force', opts, { desc = 'Go to definition' }))
      vim.keymap.set('n', 'gD', vim.lsp.buf.declaration,
        vim.tbl_extend('force', opts, { desc = 'Go to declaration' }))
      vim.keymap.set('n', 'gi', vim.lsp.buf.implementation,
        vim.tbl_extend('force', opts, { desc = 'Go to implementation' }))
      vim.keymap.set('n', 'gr', vim.lsp.buf.references,
        vim.tbl_extend('force', opts, { desc = 'Show references' }))
      vim.keymap.set('n', 'K', vim.lsp.buf.hover,
        vim.tbl_extend('force', opts, { desc = 'Hover documentation' }))
      vim.keymap.set('n', '<leader>rn', vim.lsp.buf.rename,
        vim.tbl_extend('force', opts, { desc = 'Rename' }))
      vim.keymap.set('n', '<leader>ca', vim.lsp.buf.code_action,
        vim.tbl_extend('force', opts, { desc = 'Code action' }))
      vim.keymap.set('n', '<leader>f', function()
        vim.lsp.buf.format({ async = true })
      end, vim.tbl_extend('force', opts, { desc = 'Format' }))
    end

    -- Capabilities
    local capabilities = require('cmp_nvim_lsp').default_capabilities()

    -- Configure language servers
    local lspconfig = require('lspconfig')

    -- Lua
    lspconfig.lua_ls.setup({
      on_attach = on_attach,
      capabilities = capabilities,
      settings = {
        Lua = {
          diagnostics = {
            globals = { 'vim' },
          },
          workspace = {
            library = vim.api.nvim_get_runtime_file('', true),
            checkThirdParty = false,
          },
          telemetry = { enable = false },
        },
      },
    })

    -- Python
    lspconfig.pyright.setup({
      on_attach = on_attach,
      capabilities = capabilities,
    })

    -- TypeScript
    lspconfig.tsserver.setup({
      on_attach = on_attach,
      capabilities = capabilities,
    })

    -- Diagnostic configuration
    vim.diagnostic.config({
      virtual_text = true,
      signs = true,
      update_in_insert = false,
      underline = true,
      severity_sort = true,
      float = {
        border = 'rounded',
        source = 'always',
      },
    })

    -- Diagnostic signs
    local signs = { Error = ' ', Warn = ' ', Hint = ' ', Info = ' ' }
    for type, icon in pairs(signs) do
      local hl = 'DiagnosticSign' .. type
      vim.fn.sign_define(hl, { text = icon, texthl = hl, numhl = hl })
    end
  end,
}
```

### Custom Utility Functions

```lua
-- lua/utils/init.lua

local M = {}

-- Check if buffer is empty
function M.is_buffer_empty()
  return vim.fn.empty(vim.fn.expand('%:t')) == 1
end

-- Check if buffer has words
function M.has_words_before()
  local line, col = unpack(vim.api.nvim_win_get_cursor(0))
  return col ~= 0 and vim.api.nvim_buf_get_lines(0, line - 1, line, true)[1]:sub(col, col):match('%s') == nil
end

-- Get visual selection
function M.get_visual_selection()
  local _, start_row, start_col, _ = unpack(vim.fn.getpos("'<"))
  local _, end_row, end_col, _ = unpack(vim.fn.getpos("'>"))

  local lines = vim.api.nvim_buf_get_lines(0, start_row - 1, end_row, false)

  if #lines == 0 then
    return ''
  end

  lines[1] = string.sub(lines[1], start_col)
  lines[#lines] = string.sub(lines[#lines], 1, end_col)

  return table.concat(lines, '\n')
end

-- Toggle quickfix
function M.toggle_quickfix()
  local qf_exists = false
  for _, win in pairs(vim.fn.getwininfo()) do
    if win.quickfix == 1 then
      qf_exists = true
      break
    end
  end

  if qf_exists then
    vim.cmd('cclose')
  else
    vim.cmd('copen')
  end
end

-- Create floating window
function M.create_float(opts)
  opts = opts or {}
  local width = opts.width or math.floor(vim.o.columns * 0.8)
  local height = opts.height or math.floor(vim.o.lines * 0.8)

  local row = math.floor((vim.o.lines - height) / 2)
  local col = math.floor((vim.o.columns - width) / 2)

  local buf = vim.api.nvim_create_buf(false, true)
  local win = vim.api.nvim_open_win(buf, true, {
    relative = 'editor',
    width = width,
    height = height,
    row = row,
    col = col,
    style = 'minimal',
    border = 'rounded',
  })

  return { buf = buf, win = win }
end

return M
```

## Complete init.lua Example

```lua
-- ~/.config/nvim/init.lua

-- Set leader keys
vim.g.mapleader = ' '
vim.g.maplocalleader = ' '

-- Disable some built-in plugins
local disabled_built_ins = {
  'netrw',
  'netrwPlugin',
  'netrwSettings',
  'netrwFileHandlers',
  'gzip',
  'zip',
  'zipPlugin',
  'tar',
  'tarPlugin',
  'getscript',
  'getscriptPlugin',
  'vimball',
  'vimballPlugin',
  '2html_plugin',
  'logipat',
  'rrhelper',
  'spellfile_plugin',
  'matchit',
}

for _, plugin in pairs(disabled_built_ins) do
  vim.g['loaded_' .. plugin] = 1
end

-- Load configuration
require('config')
```

This structure provides a solid foundation for a well-organized Neovim configuration in Lua!

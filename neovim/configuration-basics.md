# Neovim Configuration Basics

## Overview

Neovim is configured through a main configuration file that can be written in either Vimscript or Lua.

## Configuration File Locations

### Standard Paths
- **Linux/macOS**: `~/.config/nvim/init.vim` or `~/.config/nvim/init.lua`
- **Windows**: `~/AppData/Local/nvim/init.vim` or `~/AppData/Local/nvim/init.lua`

### File Priority
- If both `init.vim` and `init.lua` exist, Neovim will load `init.lua`
- You can use either Vimscript or Lua, or mix both

## Vimscript Basics

Vimscript is the traditional Vim configuration language.

### Comments
```vim
" This is a comment in Vimscript
```

### Setting Options

#### Boolean Options
```vim
set number          " Enable line numbers
set nonumber        " Disable line numbers
set number!         " Toggle line numbers
```

#### String/Number Options
```vim
set tabstop=4       " Set tab width to 4 spaces
set encoding=utf-8  " Set encoding
```

#### Multiple Options
```vim
set number relativenumber signcolumn=yes
```

### Common Option Operators
- `set option` - Enable boolean option
- `set nooption` - Disable boolean option
- `set option!` - Toggle boolean option
- `set option?` - Query current value
- `set option=value` - Set value
- `set option+=value` - Append to value
- `set option-=value` - Remove from value
- `set option^=value` - Prepend to value

### Key Mappings

#### Mapping Syntax
```vim
{mode}{attribute}map {lhs} {rhs}
```

#### Modes
- `n` - Normal mode
- `i` - Insert mode
- `v` - Visual mode
- `x` - Visual block mode
- `t` - Terminal mode
- `` (empty) - Normal, Visual, Select, Operator-pending

#### Attributes
- `nore` - Non-recursive (prevents infinite loops)
- `silent` - Don't echo command

#### Examples
```vim
" Normal mode: Press <A-p> to open file finder
nnoremap <A-p> :GFiles<CR>

" Insert mode: Map jj to Escape
inoremap jj <Esc>

" Terminal mode: Use Esc to exit terminal mode
tnoremap <Esc> <C-\><C-n>

" Silent mapping (doesn't show in command line)
nnoremap <silent> K :call ShowDocumentation()<CR>

" Leader key mappings
let mapleader = " "
nnoremap <leader>w :w<CR>
```

#### Special Keys
- `<CR>` - Enter/Return
- `<Esc>` - Escape
- `<Space>` - Spacebar
- `<Tab>` - Tab
- `<C-x>` - Ctrl+x
- `<A-x>` or `<M-x>` - Alt+x
- `<S-x>` - Shift+x
- `<leader>` - Leader key (customizable)

### Variables

```vim
" String variable
let g:python3_host_prog = '~/.config/nvim/python/venv/bin/python'

" Number variable
let g:my_var = 42

" List
let g:my_list = [1, 2, 3]

" Dictionary
let g:my_dict = {'key': 'value'}
```

#### Variable Scopes
- `g:` - Global
- `l:` - Local to function
- `b:` - Local to buffer
- `w:` - Local to window
- `t:` - Local to tab
- `s:` - Local to script
- `v:` - Vim predefined

### Functions

```vim
function! ShowDocumentation()
  if CocAction('hasProvider', 'hover')
    call CocActionAsync('doHover')
  else
    call feedkeys('K', 'in')
  endif
endfunction
```

- `function!` - The `!` allows overwriting existing functions
- `endfunction` - Marks the end of function definition

### Autocommands

Autocommands execute commands automatically on events.

```vim
" Create autocommand group (prevents duplicates on reload)
augroup mygroup
  autocmd!  " Clear all autocommands in this group
  autocmd FileType typescript,json setl formatexpr=CocAction('formatSelected')
  autocmd CursorHold * silent call CocActionAsync('highlight')
augroup end
```

#### Common Events
- `BufRead`, `BufNewFile` - When reading/creating buffer
- `FileType` - When filetype is set
- `CursorHold` - When cursor hasn't moved for 'updatetime' ms
- `InsertEnter`, `InsertLeave` - Enter/leave insert mode
- `VimEnter` - After Vim starts

### Conditional Statements

```vim
if condition
  " code
elseif another_condition
  " code
else
  " code
endif
```

### Syntax Highlighting

```vim
syntax on           " Enable syntax highlighting
syntax off          " Disable syntax highlighting
set syntax=python   " Set syntax for current buffer
```

## Essential Settings Explained

### Search Settings
```vim
set hlsearch       " Highlight search results
set incsearch      " Incremental search (search as you type)
set smartcase      " Case-sensitive if uppercase present, otherwise case-insensitive
set ignorecase     " Ignore case in searches (works with smartcase)
```

### Display Settings
```vim
set number         " Show line numbers
set relativenumber " Show relative line numbers
set signcolumn=yes " Always show sign column (prevents shift)
set signcolumn=no  " Never show sign column
```

### Special Settings
```vim
set ve+=onemore    " Virtual edit: allow cursor one char past line end
set re=0           " Use new regex engine (for better performance)
```

## Sourcing Other Files

```vim
" Source another Vimscript file
source ~/.config/nvim/plugins.vim

" Source a Lua file
lua require('config')
```

## Lua Blocks in Vimscript

You can embed Lua code in Vimscript files:

```vim
lua <<EOF
-- Lua code here
vim.opt.number = true
print("Hello from Lua")
EOF
```

## Checking Your Configuration

```vim
:echo $MYVIMRC           " Show path to your config file
:scriptnames             " List all sourced scripts
:verbose set option?     " Show where option was last set
:checkhealth             " Run health checks
```

## Best Practices

1. **Use comments** - Document your configuration
2. **Group related settings** - Organize by functionality
3. **Use `noremap` variants** - Prevent recursive mappings
4. **Create autocommand groups** - Prevent duplicate autocommands
5. **Check help** - Use `:help command` to learn more
6. **Keep backups** - Version control your config with git

## Next Steps

- Learn about [Plugin Management](./plugin-management.md)
- Understand [Lua in Neovim](./lua/README.md)
- Analyze [your current setup](./current-setup.md)

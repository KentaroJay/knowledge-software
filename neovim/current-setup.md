# Your Current Neovim Setup Analysis

This document breaks down your `~/.config/nvim/init.vim` configuration.

## Overview

**Configuration Style**: Vimscript with embedded Lua blocks
**Plugin Manager**: vim-plug
**Location**: `~/.config/nvim/init.vim`

## Standard Settings

```vim
" Exit Terminal Mode with Esc
tnoremap <Esc> <C-\><C-n>
```
- Maps Escape key in terminal mode to exit to normal mode
- `<C-\><C-n>` is the default terminal exit sequence

```vim
syntax on
```
- Enables syntax highlighting (though TreeSitter will override this)

```vim
set re=0
```
- Uses the new regex engine (better performance with modern syntax highlighting)
- `re=0` automatically selects the best engine

```vim
set hlsearch
set incsearch
set smartcase
```
- `hlsearch`: Highlights all search matches
- `incsearch`: Shows matches while typing search
- `smartcase`: Case-insensitive unless search contains uppercase

```vim
set signcolumn=no
```
- Hides the sign column (used by LSP for diagnostics, git signs, etc.)
- **Note**: Setting to `yes` or `auto` can prevent screen shifting

```vim
set number
```
- Shows absolute line numbers

```vim
set ve+=onemore
```
- Allows cursor to move one character past the end of the line
- `ve` = virtualedit

## Installed Plugins

### 1. FZF - Fuzzy Finder

```vim
Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }
Plug 'junegunn/fzf.vim'
```

**Purpose**: Fast fuzzy file and content searching

**Your Keybindings**:
- `Alt+p`: Open Git file finder (`:GFiles`)
- `Alt+g`: Open Silver Searcher (`:Ag`)

**Custom Ag Configuration**:
```vim
command! -bang -nargs=* Ag call fzf#vim#ag(<q-args>, {'options': '--delimiter : --nth 4..'}, <bang>0)
```
- Customizes Ag to only search in file content (skips filename, line number in search)

### 2. CoC.nvim - Conquer of Completion

```vim
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```

**Purpose**: LSP client providing code completion, diagnostics, and navigation

**Your Keybindings**:
- `gd`: Go to definition
- `gy`: Go to type definition
- `gi`: Go to implementation
- `gr`: Go to references
- `K`: Show documentation
- `<space>rn`: Rename symbol
- `<space>f`: Format selected code

**Auto-commands**:
```vim
autocmd CursorHold * silent call CocActionAsync('highlight')
```
- Highlights symbol under cursor and its references automatically

```vim
autocmd FileType typescript,json setl formatexpr=CocAction('formatSelected')
```
- Sets up format expression for TypeScript and JSON files

### 3. TreeSitter - Modern Syntax Highlighting

```vim
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}
```

**Purpose**: AST-based syntax highlighting (more accurate than regex)

**Your Configuration**:
```lua
require'nvim-treesitter.configs'.setup {
  ensure_installed = {
    "python", "javascript", "typescript", "lua", "vim", "vimdoc",
    "json", "yaml", "html", "css", "markdown", "markdown_inline", "bash"
  },
  auto_install = true,
  highlight = {
    enable = true,
    additional_vim_regex_highlighting = false,
  },
  indent = {
    enable = true,
  }
}
```
- Automatically installs and updates parsers for listed languages
- Enables highlighting and smart indentation
- Disables Vim's regex highlighting (TreeSitter is better)

### 4. Monokai Nightasty - Color Scheme

```vim
Plug 'polirritmico/monokai-nightasty.nvim'
colorscheme monokai-nightasty
```

**Purpose**: Modern dark color scheme with good contrast

### 5. Nvim-tree - File Explorer

```vim
Plug 'nvim-tree/nvim-web-devicons'  " Icons for file tree
Plug 'nvim-tree/nvim-tree.lua'      " File explorer
```

**Purpose**: File system explorer sidebar

**Your Configuration**:
```lua
-- Disable default file explorer
vim.g.loaded_netrw = 1
vim.g.loaded_netrwPlugin = 1

-- Enable 24-bit colors
vim.opt.termguicolors = true

-- Use default setup
require("nvim-tree").setup()
```

**Your Keybinding**:
- `Alt+e`: Toggle file explorer (`:NvimTreeToggle`)

## Python Provider

```vim
let g:python3_host_prog = '~/.config/nvim/python/venv/bin/python'
```
- Points Neovim to a Python virtual environment
- Used by Python-based plugins and features
- Isolates plugin dependencies from system Python

## Configuration Architecture

### Strengths
1. **Clean organization** - Settings grouped logically
2. **Good documentation** - Comments in Japanese and English
3. **Modern tooling** - Uses TreeSitter and LSP
4. **Efficient navigation** - FZF for quick file/content search
5. **Hybrid approach** - Vimscript + Lua where appropriate

### Potential Improvements

1. **Sign Column**:
   ```vim
   set signcolumn=yes  " or auto:1-2
   ```
   - Prevents screen shifting when diagnostics appear

2. **Relative Numbers**:
   ```vim
   set relativenumber
   ```
   - Easier to count lines for motions (e.g., `5j`)

3. **CoC Extensions**:
   ```vim
   let g:coc_global_extensions = [
     \ 'coc-json',
     \ 'coc-tsserver',
     \ 'coc-python',
     \ ]
   ```
   - Auto-install CoC language servers

4. **Split Navigation**:
   ```vim
   nnoremap <C-h> <C-w>h
   nnoremap <C-j> <C-w>j
   nnoremap <C-k> <C-w>k
   nnoremap <C-l> <C-w>l
   ```
   - Easier window navigation

5. **Better Search Clearing**:
   ```vim
   nnoremap <Esc><Esc> :nohlsearch<CR>
   ```
   - Clear search highlighting easily

## Plugin Workflow

### File Navigation
1. `Alt+e` - Open file tree
2. `Alt+p` - Fuzzy find Git files
3. `Alt+g` - Search content with Ag

### Code Navigation
1. Hover over symbol → `K` for documentation
2. `gd` - Jump to definition
3. `gr` - Find all references
4. `gi` - Go to implementation

### Editing
1. `<space>rn` - Rename symbol across project
2. `<space>f` - Format selected code
3. Auto-highlight references on cursor hold

## Dependencies

### Required System Tools
- **git** - For plugin installation
- **fzf** - Fuzzy finder
- **ag (silver searcher)** - Fast code search
- **Node.js** - Required by CoC.nvim
- **Python 3** - For Python provider
- **ripgrep** - Optional but recommended for searching

### Installation Check
```vim
:checkhealth
```
Run this command to verify all dependencies are installed.

## File Structure

Your current config is monolithic (`init.vim`). For better organization, consider:

```
~/.config/nvim/
├── init.vim              # Main entry point
├── lua/
│   ├── plugins.lua       # Plugin specifications
│   ├── settings.lua      # Core settings
│   └── keymaps.lua       # All keybindings
└── after/
    └── plugin/           # Plugin-specific configs
```

## Next Steps

1. **Explore plugins** - Learn each plugin's capabilities
2. **Customize keybindings** - Adjust to your workflow
3. **Add language support** - Install CoC extensions for your languages
4. **Learn Lua** - Gradually migrate to Lua configuration
5. **Experiment** - Try new plugins and features

## Useful Commands

```vim
:PlugStatus              " Check plugin status
:checkhealth             " Verify Neovim health
:CocInfo                 " CoC.nvim information
:TSUpdate                " Update TreeSitter parsers
:TSInstallInfo           " Show installed parsers
:NvimTreeFindFile        " Find current file in tree
:Maps                    " List all mappings
```

## Resources

- [CoC.nvim Documentation](https://github.com/neoclide/coc.nvim)
- [TreeSitter Documentation](https://github.com/nvim-treesitter/nvim-treesitter)
- [FZF.vim Documentation](https://github.com/junegunn/fzf.vim)
- [Nvim-tree Documentation](https://github.com/nvim-tree/nvim-tree.lua)

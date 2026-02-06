# Plugin Management in Neovim

## Overview

Neovim supports multiple plugin managers. Each has different features and approaches to installing and managing plugins.

## Popular Plugin Managers

### 1. vim-plug (Your Current Setup)

**Simple, fast, and widely used plugin manager.**

#### Installation
```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

#### Basic Usage in init.vim
```vim
call plug#begin()
  " Plugin declarations go here
  Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }
  Plug 'neoclide/coc.nvim', {'branch': 'release'}
call plug#end()
```

#### Syntax
```vim
Plug 'username/repository'                           " Basic plugin
Plug 'username/repository', { 'branch': 'release' }  " Specific branch
Plug 'username/repository', { 'tag': 'v1.0' }        " Specific tag
Plug 'username/repository', { 'do': 'command' }      " Run post-install command
Plug 'username/repository', { 'for': 'python' }      " Load only for specific filetype
Plug 'username/repository', { 'on': 'CommandName' }  " Lazy load on command
```

#### Commands
- `:PlugInstall` - Install plugins
- `:PlugUpdate` - Update plugins
- `:PlugClean` - Remove unused plugins
- `:PlugUpgrade` - Upgrade vim-plug itself
- `:PlugStatus` - Check plugin status
- `:PlugDiff` - See changes after update

#### Advanced Options
```vim
" Lazy loading on command
Plug 'scrooloose/nerdtree', { 'on': 'NERDTreeToggle' }

" Lazy loading on filetype
Plug 'tpope/vim-fireplace', { 'for': 'clojure' }

" Run command after installation/update
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}

" Install from local directory
Plug '~/my-local-plugin'

" Disable plugin temporarily
Plug 'username/repository', { 'frozen': 1 }
```

### 2. lazy.nvim (Modern, Lua-based)

**Fast, feature-rich plugin manager designed for Neovim.**

#### Installation (in init.lua)
```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable",
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)
```

#### Basic Usage
```lua
require("lazy").setup({
  "folke/tokyonight.nvim",
  {
    "nvim-treesitter/nvim-treesitter",
    build = ":TSUpdate",
  },
  {
    "neovim/nvim-lspconfig",
    dependencies = { "williamboman/mason.nvim" },
  },
})
```

#### Features
- **Lazy loading by default** - Plugins load when needed
- **Startup profiling** - See what slows down your startup
- **UI for plugin management** - Beautiful interface with `:Lazy`
- **Lockfile** - Ensures reproducible plugin versions
- **Automatic lazy loading** - Based on events, commands, keys, filetypes

#### Lazy Loading Examples
```lua
{
  "nvim-telescope/telescope.nvim",
  cmd = "Telescope",  -- Load when :Telescope command is used
}

{
  "folke/which-key.nvim",
  keys = { "<leader>" },  -- Load when leader key is pressed
}

{
  "lewis6991/gitsigns.nvim",
  event = "BufRead",  -- Load when reading a buffer
}

{
  "simrat39/rust-tools.nvim",
  ft = "rust",  -- Load only for Rust files
}
```

#### Commands
- `:Lazy` - Open plugin manager UI
- `:Lazy install` - Install missing plugins
- `:Lazy update` - Update plugins
- `:Lazy clean` - Remove unused plugins
- `:Lazy sync` - Install, update, and clean
- `:Lazy profile` - Profile startup time

### 3. packer.nvim (Lua-based, predecessor to lazy.nvim)

**Use-package inspired plugin manager for Neovim.**

#### Installation
```lua
local fn = vim.fn
local install_path = fn.stdpath('data')..'/site/pack/packer/start/packer.nvim'
if fn.empty(fn.glob(install_path)) > 0 then
  fn.system({'git', 'clone', '--depth', '1', 'https://github.com/wbthomason/packer.nvim', install_path})
end
```

#### Basic Usage
```lua
require('packer').startup(function(use)
  use 'wbthomason/packer.nvim'  -- Packer manages itself
  use 'folke/tokyonight.nvim'
  use {
    'nvim-treesitter/nvim-treesitter',
    run = ':TSUpdate'
  }
end)
```

#### Commands
- `:PackerInstall` - Install plugins
- `:PackerUpdate` - Update plugins
- `:PackerSync` - Install, update, and clean
- `:PackerClean` - Remove unused plugins
- `:PackerCompile` - Compile lazy-loader code

### 4. Native Package Management

Neovim has built-in package support (no plugin manager needed).

#### Directory Structure
```
~/.config/nvim/pack/mypackages/start/    # Auto-loaded plugins
~/.config/nvim/pack/mypackages/opt/      # Manually loaded plugins
```

#### Usage
```bash
# Install plugin (start - auto load)
cd ~/.config/nvim/pack/mypackages/start/
git clone https://github.com/username/plugin.git

# Install plugin (opt - manual load)
cd ~/.config/nvim/pack/mypackages/opt/
git clone https://github.com/username/plugin.git
```

#### Load Optional Plugins
```vim
packadd plugin-name
```

## Common Plugin Types

### 1. LSP (Language Server Protocol)
```vim
" Built-in LSP client configuration
Plug 'neovim/nvim-lspconfig'

" LSP enhancements
Plug 'williamboman/mason.nvim'
Plug 'williamboman/mason-lspconfig.nvim'

" Alternative: CoC (Language Server Client)
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```

### 2. Completion Engines
```vim
" nvim-cmp (native)
Plug 'hrsh7th/nvim-cmp'
Plug 'hrsh7th/cmp-nvim-lsp'

" CoC.nvim (your current setup)
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```

### 3. Syntax Highlighting
```vim
" TreeSitter (modern, AST-based)
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}

" Traditional syntax files
" (built-in, no plugin needed)
```

### 4. File Navigation
```vim
" File tree
Plug 'nvim-tree/nvim-tree.lua'
Plug 'kyazdani42/nvim-web-devicons'  " Icons

" Fuzzy finder
Plug 'junegunn/fzf'
Plug 'junegunn/fzf.vim'
Plug 'nvim-telescope/telescope.nvim'  " Lua alternative
```

### 5. Git Integration
```vim
Plug 'tpope/vim-fugitive'      " Git commands
Plug 'lewis6991/gitsigns.nvim' " Git decorations
```

### 6. Status Line
```vim
Plug 'nvim-lualine/lualine.nvim'
Plug 'vim-airline/vim-airline'
```

### 7. Color Schemes
```vim
Plug 'folke/tokyonight.nvim'
Plug 'polirritmico/monokai-nightasty.nvim'
Plug 'catppuccin/nvim'
```

## Plugin Configuration

### Lua Configuration (Preferred for Lua plugins)
```vim
lua <<EOF
require('nvim-tree').setup({
  view = {
    width = 30,
  },
})
EOF
```

### Vimscript Configuration
```vim
let g:plugin_option = 'value'
```

### Separate Config Files
```
~/.config/nvim/
├── init.vim
├── lua/
│   ├── plugins.lua          " Plugin specifications
│   └── config/
│       ├── treesitter.lua
│       └── lsp.lua
```

## Migration Guide: vim-plug to lazy.nvim

### vim-plug (init.vim)
```vim
call plug#begin()
  Plug 'folke/tokyonight.nvim'
  Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}
call plug#end()
```

### lazy.nvim (init.lua)
```lua
require("lazy").setup({
  "folke/tokyonight.nvim",
  { "nvim-treesitter/nvim-treesitter", build = ":TSUpdate" },
})
```

## Performance Tips

1. **Lazy load plugins** - Only load when needed
2. **Use profiling** - Identify slow plugins
3. **Minimal plugin count** - Only install what you use
4. **Async operations** - Use plugins that load asynchronously
5. **TreeSitter over regex** - Modern syntax highlighting

## Troubleshooting

```vim
" Check plugin status
:PlugStatus        " (vim-plug)
:Lazy              " (lazy.nvim)

" Check health
:checkhealth

" View loaded plugins
:scriptnames

" Check runtime path
:echo &runtimepath
```

## Best Practices

1. **Version control your config** - Use git
2. **Pin important plugins** - Use specific tags/commits
3. **Read plugin documentation** - Check the GitHub README
4. **Test before updating** - Update plugins carefully
5. **Use lockfiles** - Ensure reproducibility (lazy.nvim)
6. **Comment your plugins** - Explain why you use each plugin

## Next Steps

- Learn about [Lua in Neovim](./lua/README.md)
- Understand [your current configuration](./current-setup.md)
- Explore [configuration basics](./configuration-basics.md)

# Neovim Cheat Sheet

## Installation and Setup

### Installation
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install neovim

# macOS
brew install neovim

# Windows
winget install Neovim.Neovim

# Build from source
git clone https://github.com/neovim/neovim.git
cd neovim && make CMAKE_BUILD_TYPE=RelWithDebInfo
sudo make install
```

### Initial Configuration
```bash
# Create config directory
mkdir -p ~/.config/nvim

# Basic init.vim
echo 'set number' > ~/.config/nvim/init.vim

# Or init.lua for Lua configuration
echo 'vim.opt.number = true' > ~/.config/nvim/init.lua
```

## Essential Vim/Neovim Commands

### Basic Movement
| Command | Action |
|---------|---------|
| `h j k l` | Left, Down, Up, Right |
| `w` | Next word |
| `b` | Previous word |
| `e` | End of word |
| `0` | Beginning of line |
| `$` | End of line |
| `gg` | Top of file |
| `G` | Bottom of file |
| `Ctrl+f` | Page down |
| `Ctrl+b` | Page up |

### Editing Commands
| Command | Action |
|---------|---------|
| `i` | Insert before cursor |
| `a` | Insert after cursor |
| `I` | Insert at beginning of line |
| `A` | Insert at end of line |
| `o` | Open line below |
| `O` | Open line above |
| `x` | Delete character |
| `dd` | Delete line |
| `dw` | Delete word |
| `d$` | Delete to end of line |
| `yy` | Yank (copy) line |
| `yw` | Yank word |
| `p` | Paste after cursor |
| `P` | Paste before cursor |

### Search and Replace
| Command | Action |
|---------|---------|
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Next search result |
| `N` | Previous search result |
| `:%s/old/new/g` | Replace all |
| `:%s/old/new/gc` | Replace all with confirmation |
| `:s/old/new` | Replace in current line |

## Advanced Configuration

### Modern Neovim init.lua
```lua
-- ~/.config/nvim/init.lua
vim.g.mapleader = ' '

-- Basic settings
vim.opt.number = true
vim.opt.relativenumber = true
vim.opt.tabstop = 2
vim.opt.shiftwidth = 2
vim.opt.expandtab = true
vim.opt.smartindent = true
vim.opt.wrap = false
vim.opt.swapfile = false
vim.opt.backup = false
vim.opt.undodir = os.getenv("HOME") .. "/.vim/undodir"
vim.opt.undofile = true
vim.opt.hlsearch = false
vim.opt.incsearch = true
vim.opt.termguicolors = true
vim.opt.scrolloff = 8
vim.opt.signcolumn = "yes"
vim.opt.isfname:append("@-@")
vim.opt.updatetime = 50
vim.opt.colorcolumn = "80"

-- Key mappings
vim.keymap.set("v", "J", ":m '>+1<CR>gv=gv")
vim.keymap.set("v", "K", ":m '<-2<CR>gv=gv")
vim.keymap.set("n", "J", "mzJ`z")
vim.keymap.set("n", "<C-d>", "<C-d>zz")
vim.keymap.set("n", "<C-u>", "<C-u>zz")
vim.keymap.set("n", "n", "nzzzv")
vim.keymap.set("n", "N", "Nzzzv")

-- Leader key mappings
vim.keymap.set("n", "<leader>pv", vim.cmd.Ex)
vim.keymap.set("x", "<leader>p", [["_dP]])
vim.keymap.set({"n", "v"}, "<leader>y", [["+y]])
vim.keymap.set("n", "<leader>Y", [["+Y]])
vim.keymap.set({"n", "v"}, "<leader>d", [["_d]])
```

### Plugin Management with lazy.nvim
```lua
-- ~/.config/nvim/lua/plugins/init.lua
return {
  -- Telescope (fuzzy finder)
  {
    'nvim-telescope/telescope.nvim',
    tag = '0.1.4',
    dependencies = { 'nvim-lua/plenary.nvim' },
    config = function()
      require('telescope').setup{}
      local builtin = require('telescope.builtin')
      vim.keymap.set('n', '<leader>ff', builtin.find_files, {})
      vim.keymap.set('n', '<leader>fg', builtin.live_grep, {})
      vim.keymap.set('n', '<leader>fb', builtin.buffers, {})
      vim.keymap.set('n', '<leader>fh', builtin.help_tags, {})
    end
  },

  -- Treesitter (syntax highlighting)
  {
    'nvim-treesitter/nvim-treesitter',
    build = ':TSUpdate',
    config = function()
      require('nvim-treesitter.configs').setup {
        ensure_installed = { "lua", "vim", "javascript", "python", "rust" },
        highlight = { enable = true },
        indent = { enable = true },
      }
    end
  },

  -- LSP Zero (language server)
  {
    'VonHeikemen/lsp-zero.nvim',
    branch = 'v3.x',
    dependencies = {
      'williamboman/mason.nvim',
      'williamboman/mason-lspconfig.nvim',
      'neovim/nvim-lspconfig',
      'hrsh7th/nvim-cmp',
      'hrsh7th/cmp-nvim-lsp',
      'L3MON4D3/LuaSnip',
    },
    config = function()
      local lsp_zero = require('lsp-zero')
      lsp_zero.on_attach(function(client, bufnr)
        local opts = {buffer = bufnr, remap = false}
        vim.keymap.set("n", "gd", function() vim.lsp.buf.definition() end, opts)
        vim.keymap.set("n", "K", function() vim.lsp.buf.hover() end, opts)
        vim.keymap.set("n", "<leader>vws", function() vim.lsp.buf.workspace_symbol() end, opts)
        vim.keymap.set("n", "<leader>vd", function() vim.diagnostic.open_float() end, opts)
        vim.keymap.set("n", "[d", function() vim.diagnostic.goto_next() end, opts)
        vim.keymap.set("n", "]d", function() vim.diagnostic.goto_prev() end, opts)
        vim.keymap.set("n", "<leader>vca", function() vim.lsp.buf.code_action() end, opts)
        vim.keymap.set("n", "<leader>vrr", function() vim.lsp.buf.references() end, opts)
        vim.keymap.set("n", "<leader>vrn", function() vim.lsp.buf.rename() end, opts)
        vim.keymap.set("i", "<C-h>", function() vim.lsp.buf.signature_help() end, opts)
      end)

      require('mason').setup({})
      require('mason-lspconfig').setup({
        ensure_installed = {'tsserver', 'rust_analyzer', 'pyright'},
        handlers = {
          lsp_zero.default_setup,
        },
      })
    end
  },

  -- Git integration
  {
    'tpope/vim-fugitive',
    cmd = 'Git'
  },

  -- File explorer
  {
    'nvim-tree/nvim-tree.lua',
    dependencies = { 'nvim-tree/nvim-web-devicons' },
    config = function()
      require('nvim-tree').setup()
      vim.keymap.set('n', '<leader>e', ':NvimTreeToggle<CR>')
    end
  },

  -- Status line
  {
    'nvim-lualine/lualine.nvim',
    dependencies = { 'nvim-tree/nvim-web-devicons' },
    config = function()
      require('lualine').setup()
    end
  },

  -- Color scheme
  {
    'folke/tokyonight.nvim',
    lazy = false,
    priority = 1000,
    config = function()
      vim.cmd[[colorscheme tokyonight]]
    end,
  }
}
```

### Bootstrap lazy.nvim
```lua
-- ~/.config/nvim/init.lua (add this before plugin configuration)
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup("plugins")
```

## Essential Plugins

### Plugin Categories

#### Language Server Protocol (LSP)
```lua
-- LSP Configuration
{
  'neovim/nvim-lspconfig',
  'williamboman/mason.nvim',
  'williamboman/mason-lspconfig.nvim',
  'VonHeikemen/lsp-zero.nvim',
}

-- Completion
{
  'hrsh7th/nvim-cmp',
  'hrsh7th/cmp-nvim-lsp',
  'hrsh7th/cmp-buffer',
  'hrsh7th/cmp-path',
  'hrsh7th/cmp-cmdline',
}

-- Snippets
{
  'L3MON4D3/LuaSnip',
  'saadparwaiz1/cmp_luasnip',
}
```

#### Navigation and Search
```lua
{
  'nvim-telescope/telescope.nvim',          -- Fuzzy finder
  'nvim-telescope/telescope-fzf-native.nvim', -- Native FZF
  'ThePrimeagen/harpoon',                   -- File bookmarking
  'christoomey/vim-tmux-navigator',         -- Tmux integration
}
```

#### Editing Enhancement
```lua
{
  'nvim-treesitter/nvim-treesitter',        -- Syntax highlighting
  'nvim-treesitter/nvim-treesitter-textobjects', -- Text objects
  'numToStr/Comment.nvim',                  -- Smart commenting
  'windwp/nvim-autopairs',                  -- Auto pairs
  'tpope/vim-surround',                     -- Surround objects
  'wellle/targets.vim',                     -- Additional text objects
}
```

#### Git Integration
```lua
{
  'tpope/vim-fugitive',      -- Git commands
  'lewis6991/gitsigns.nvim', -- Git signs in gutter
  'sindrets/diffview.nvim',  -- Git diff viewer
}
```

#### UI Enhancement
```lua
{
  'nvim-lualine/lualine.nvim',    -- Status line
  'nvim-tree/nvim-tree.lua',      -- File explorer
  'akinsho/bufferline.nvim',      -- Buffer tabs
  'folke/which-key.nvim',         -- Key binding helper
  'folke/trouble.nvim',           -- Diagnostics list
}
```

## Lua Scripting

### Basic Lua API
```lua
-- Vim API access
vim.api.nvim_set_keymap('n', '<leader>w', ':w<CR>', {noremap = true})
vim.api.nvim_buf_set_lines(0, 0, -1, false, {'Hello', 'World'})
vim.api.nvim_win_set_cursor(0, {1, 0})

-- Options
vim.opt.number = true
vim.opt.relativenumber = true
vim.g.mapleader = ' '

-- Key mappings
vim.keymap.set('n', '<leader>ff', function()
  require('telescope.builtin').find_files()
end)

-- Autocommands
vim.api.nvim_create_autocmd("BufWritePre", {
  pattern = "*",
  callback = function()
    vim.lsp.buf.format()
  end,
})

-- User commands
vim.api.nvim_create_user_command('Hello', function()
  print('Hello World!')
end, {})
```

### Module Structure
```lua
-- ~/.config/nvim/lua/config/keymaps.lua
local M = {}

function M.setup()
  -- General keymaps
  vim.keymap.set('n', '<leader>w', ':w<CR>')
  vim.keymap.set('n', '<leader>q', ':q<CR>')
  
  -- Window navigation
  vim.keymap.set('n', '<C-h>', '<C-w>h')
  vim.keymap.set('n', '<C-j>', '<C-w>j')
  vim.keymap.set('n', '<C-k>', '<C-w>k')
  vim.keymap.set('n', '<C-l>', '<C-w>l')
end

return M
```

## Modern Features

### Built-in LSP
```lua
-- Language server setup
local lspconfig = require('lspconfig')

-- TypeScript
lspconfig.tsserver.setup{}

-- Python
lspconfig.pyright.setup{}

-- Rust
lspconfig.rust_analyzer.setup{
  settings = {
    ["rust-analyzer"] = {
      cargo = {
        allFeatures = true,
      },
    },
  },
}

-- Lua
lspconfig.lua_ls.setup {
  settings = {
    Lua = {
      runtime = {
        version = 'LuaJIT',
      },
      diagnostics = {
        globals = {'vim'},
      },
      workspace = {
        library = vim.api.nvim_get_runtime_file("", true),
      },
      telemetry = {
        enable = false,
      },
    },
  },
}
```

### Tree-sitter Configuration
```lua
require('nvim-treesitter.configs').setup {
  ensure_installed = {
    "lua", "vim", "vimdoc", "query",
    "javascript", "typescript", "tsx",
    "python", "rust", "go", "java",
    "html", "css", "json", "yaml"
  },
  
  highlight = {
    enable = true,
    additional_vim_regex_highlighting = false,
  },
  
  indent = {
    enable = true
  },
  
  textobjects = {
    select = {
      enable = true,
      lookahead = true,
      keymaps = {
        ["af"] = "@function.outer",
        ["if"] = "@function.inner",
        ["ac"] = "@class.outer",
        ["ic"] = "@class.inner",
      },
    },
  },
}
```

### Telescope Configuration
```lua
require('telescope').setup{
  defaults = {
    mappings = {
      i = {
        ["<C-n>"] = require('telescope.actions').cycle_history_next,
        ["<C-p>"] = require('telescope.actions').cycle_history_prev,
        ["<C-j>"] = require('telescope.actions').move_selection_next,
        ["<C-k>"] = require('telescope.actions').move_selection_previous,
      },
    },
  },
  extensions = {
    fzf = {
      fuzzy = true,
      override_generic_sorter = true,
      override_file_sorter = true,
      case_mode = "smart_case",
    }
  }
}

-- Load extensions
require('telescope').load_extension('fzf')

-- Keymaps
local builtin = require('telescope.builtin')
vim.keymap.set('n', '<leader>ff', builtin.find_files, {})
vim.keymap.set('n', '<leader>fg', builtin.live_grep, {})
vim.keymap.set('n', '<leader>fb', builtin.buffers, {})
vim.keymap.set('n', '<leader>fh', builtin.help_tags, {})
vim.keymap.set('n', '<leader>fr', builtin.oldfiles, {})
vim.keymap.set('n', '<leader>fc', builtin.commands, {})
```

## Advanced Customization

### Custom Functions
```lua
-- ~/.config/nvim/lua/utils/init.lua
local M = {}

function M.toggle_number()
  if vim.opt.number:get() then
    vim.opt.number = false
    vim.opt.relativenumber = false
  else
    vim.opt.number = true
    vim.opt.relativenumber = true
  end
end

function M.format_buffer()
  local save_cursor = vim.api.nvim_win_get_cursor(0)
  vim.cmd([[%s/\s\+$//e]]) -- Remove trailing whitespace
  vim.api.nvim_win_set_cursor(0, save_cursor)
end

function M.source_vimrc()
  vim.cmd('source ~/.config/nvim/init.lua')
  print('Vimrc reloaded!')
end

return M
```

### Statusline Configuration
```lua
-- Custom lualine configuration
require('lualine').setup {
  options = {
    icons_enabled = true,
    theme = 'auto',
    component_separators = { left = '', right = ''},
    section_separators = { left = '', right = ''},
    disabled_filetypes = {
      statusline = {},
      winbar = {},
    },
    always_divide_middle = true,
    globalstatus = false,
  },
  sections = {
    lualine_a = {'mode'},
    lualine_b = {'branch', 'diff', 'diagnostics'},
    lualine_c = {'filename'},
    lualine_x = {'encoding', 'fileformat', 'filetype'},
    lualine_y = {'progress'},
    lualine_z = {'location'}
  },
  inactive_sections = {
    lualine_a = {},
    lualine_b = {},
    lualine_c = {'filename'},
    lualine_x = {'location'},
    lualine_y = {},
    lualine_z = {}
  },
  tabline = {},
  winbar = {},
  inactive_winbar = {},
  extensions = {}
}
```

## Best Practices and Tips

### Performance Optimization
```lua
-- Lazy loading plugins
{
  'nvim-telescope/telescope.nvim',
  cmd = 'Telescope',
  keys = {
    { '<leader>ff', '<cmd>Telescope find_files<cr>' },
    { '<leader>fg', '<cmd>Telescope live_grep<cr>' },
  },
}

-- Disable unused providers
vim.g.loaded_python_provider = 0
vim.g.loaded_python3_provider = 0
vim.g.loaded_node_provider = 0
vim.g.loaded_ruby_provider = 0
vim.g.loaded_perl_provider = 0

-- Optimize startup
vim.opt.shadafile = "NONE"
vim.opt.swapfile = false
vim.opt.backup = false
```

### Code Organization
```text
~/.config/nvim/
├── init.lua
├── lua/
│   ├── config/
│   │   ├── keymaps.lua
│   │   ├── options.lua
│   │   └── autocmds.lua
│   ├── plugins/
│   │   ├── init.lua
│   │   ├── lsp.lua
│   │   ├── telescope.lua
│   │   └── treesitter.lua
│   └── utils/
│       └── init.lua
└── after/
    └── plugin/
```

### Workflow Tips
1. **Use leader key effectively**: Map common actions to `<leader>` combinations
2. **Master text objects**: Learn `ci"`, `da{`, `yi(` patterns
3. **Use marks**: Set marks with `ma`, jump with `'a`
4. **Buffer management**: Use `:bd` to delete buffers, `:ls` to list
5. **Window splits**: `<C-w>v` vertical, `<C-w>s` horizontal
6. **Registers**: Use `"ay` to yank to register a, `"ap` to paste
7. **Macros**: Record with `qa`, replay with `@a`

## Troubleshooting Common Issues

### Plugin Issues
```lua
-- Check plugin status
:Lazy
:checkhealth

-- Clear plugin cache
:Lazy clear

-- Update plugins
:Lazy update
```

### LSP Issues
```bash
# Check LSP status
:LspInfo

# Restart LSP
:LspRestart

# Check Mason installations
:Mason
```

### Performance Issues
```lua
-- Profile startup time
nvim --startuptime startup.log

-- Check slow plugins
:Lazy profile
```

### Configuration Debugging
```lua
-- Reload configuration
:luafile %

-- Check lua errors
:messages

-- Debug key mappings
:nmap <key>
:verbose nmap <key>
```

## Official Documentation Links

- [Neovim Documentation](https://neovim.io/doc/)
- [Neovim GitHub](https://github.com/neovim/neovim)
- [Lua API Reference](https://neovim.io/doc/user/lua.html)
- [Plugin Development](https://neovim.io/doc/user/lua-guide.html)
- [LSP Configuration](https://neovim.io/doc/user/lsp.html)
- [Tree-sitter Guide](https://neovim.io/doc/user/treesitter.html)
- [Neovim Wiki](https://github.com/neovim/neovim/wiki)
- [Awesome Neovim](https://github.com/rockerBOO/awesome-neovim)
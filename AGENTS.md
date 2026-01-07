# Agent Guidelines for Kickstart.nvim

This document provides coding standards and operational commands for agentic development in this Neovim configuration repository.

## Project Overview

This is a Neovim configuration based on the [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim) template. It uses `lazy.nvim` for plugin management and `mason.nvim` to manage external tools like language servers and formatters.

## Build/Lint/Test Commands

### Lua Formatting
- **Check formatting**: `stylua --check .`
- **Format code**: `stylua .`
- CI validates formatting on PRs via `.github/workflows/stylua.yml`

### Health Checks
- Run health check in Neovim: `:checkhealth`
- Run kickstart-specific health: `:checkhealth kickstart`
- Health checks verify: Neovim version (>= 0.10), git, make, unzip, ripgrep

### Plugin Management
- Sync plugins: `:Lazy sync`
- Check plugin status: `:Lazy`
- Clean unused plugins: `:Lazy clean`

### Testing
- No unit tests exist (configuration only)
- Validate by loading Neovim and running `:checkhealth`
- Test specific features interactively in Neovim

## Code Style Guidelines

### Formatting (Stylua)
- Indent width: 2 spaces
- Column width: 160 characters
- Line endings: Unix
- Quote style: AutoPreferSingle
- Call parentheses: None (avoid redundant parentheses in function calls)

### Imports and Requires
- Use `require 'module'` for Lua modules
- Plugin requires in config functions: `local lint = require 'lint'`
- Import at point of use within config blocks, not file top

### Naming Conventions
- Variables/functions: `snake_case` (e.g., `lint_augroup`, `check_version`)
- Global Neovim settings: `camelCase` via `vim.g` (e.g., `vim.g.have_nerd_font`)
- File/module names: `snake_case` (e.g., `neo-tree.lua`, `debug.lua`)
- Augroup names: lowercase strings (e.g., `'lint'`, `'user_events'`)

### Neovim API Usage
- Options: `vim.o.option_name = value` (e.g., `vim.o.mouse = 'a'`)
- Commands: `vim.cmd 'command_name'` (e.g., `vim.cmd 'filetype plugin indent on'`)
- Keymaps: `vim.keymap.set(mode, lhs, rhs, { desc = '...' })`
- Autocommands: `vim.api.nvim_create_autocmd(events, { group, callback, ... })`
- Augroups: `vim.api.nvim_create_augroup(name, { clear = true })`

### Plugin Configuration (lazy.nvim)
Plugin specs use table format with common fields:
```lua
{
  'author/plugin-name',
  version = '*',          -- optional: pin version
  dependencies = { ... }, -- optional: plugin deps
  event = { ... },        -- optional: lazy loading
  keys = { ... },         -- optional: keybindings
  opts = { ... },         -- optional: auto-configuration
  config = function()     -- optional: custom config
    -- setup code
  end,
}
```

### Keybindings
- Define in `keys` field for plugins or via `vim.keymap.set` in init.lua
- Include `desc` field for all keymaps (required by which-key)
- Format: `{ '<leader>gb', function() ... end, desc = '[G]it [B]lame' }`
- Use descriptive bracket notation: `[C]ode`, `[D]iff`, `[G]it`
- Mode prefixes: `n` (normal), `i` (insert), `x` (visual), `v` (visual+select)

### Autocommands
- Always create augroups with `{ clear = true }` to avoid duplicates
- Group related autocmds in dedicated augroups
- Use `vim.bo` for buffer options in callbacks

### Comments
- Use `--` prefix for comments (Lua style)
- Section headers: `-- [[ Section Name ]]`
- Help references: `-- See \` :help topic \``
- NOTE comments: `-- NOTE:` for important warnings
- Avoid block comments `--[[ ... ]]` except for file headers

### File Organization
- `init.lua`: Main configuration file (single entry point)
- `lua/kickstart/plugins/*.lua`: Modular plugin configurations
- `lua/custom/plugins/*.lua`: Custom plugin additions (merge-safe)
- `lua/kickstart/health.lua`: Health check definitions

### Error Handling
- Use `pcall` for plugin requires when optional: `local ok, plugin = pcall(require, 'plugin')`
- Check plugin availability before use in keymaps/autocommands
- Lazy loading reduces startup errors from missing tools

### General Principles
- Keep `init.lua` readable with clear section headers
- Use lazy loading for non-essential plugins
- Document user-configurable settings with comments
- Prefer plugin's `opts` field over `config` when possible
- Keep keymaps discoverable via which-key descriptions

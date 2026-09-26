# Agent Guidelines for Kickstart.nvim

This document provides coding standards and operational commands for agentic development in this Neovim configuration repository.

## Project Overview

This is a Neovim configuration based on the [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim) template. It uses Neovim's built-in `vim.pack` for plugin management and `mason.nvim` to manage external tools like language servers and formatters.

## Build/Lint/Test Commands

### Lua Formatting
- **Check formatting**: `stylua --check .`
- **Format code**: `stylua .`
- `.github/workflows/stylua.yml` is inherited from upstream and only runs on `nvim-lua/kickstart.nvim`, so run `stylua --check .` yourself

### Health Checks
- Run health check in Neovim: `:checkhealth`
- Run kickstart-specific health: `:checkhealth kickstart`
- Health checks verify: Neovim version (>= 0.12), git, make, unzip, ripgrep

### Plugin Management
- Update plugins: `:PackUpdate` (all) or `:PackUpdate <name>` (defined in `init.lua`, wraps `vim.pack.update()`)
- Plugin revisions are pinned in `nvim-pack-lock.json`; Mason tool versions in `mason-lock.json`

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
- Require a plugin right after its `vim.pack.add` call, not at file top: `local lint = require 'lint'`

### Naming Conventions
- Variables/functions: `snake_case` (e.g., `lint_augroup`, `check_version`)
- Global Neovim settings: `camelCase` via `vim.g` (e.g., `vim.g.have_nerd_font`)
- File/module names: match the plugin or feature name (e.g., `neo-tree.lua`, `indent_line.lua`)
- Augroup names: lowercase strings (e.g., `'lint'`, `'user_events'`)

### Neovim API Usage
- Options: `vim.o.option_name = value` (e.g., `vim.o.mouse = 'a'`)
- Commands: `vim.cmd 'command_name'` (e.g., `vim.cmd 'filetype plugin indent on'`)
- Keymaps: `vim.keymap.set(mode, lhs, rhs, { desc = '...' })`
- Autocommands: `vim.api.nvim_create_autocmd(events, { group, callback, ... })`
- Augroups: `vim.api.nvim_create_augroup(name, { clear = true })`

### Plugin Configuration (vim.pack)
Each plugin is added, set up, and given its keymaps in one block of `init.lua`. `gh` is a local helper in `init.lua`; files under `lua/` pass the full URL instead:
```lua
vim.pack.add { gh 'f-person/git-blame.nvim' }
require('gitblame').setup { enabled = false }
vim.keymap.set('n', '<leader>gb', function() vim.cmd.GitBlameToggle() end, { desc = '[G]it [B]lame' })
```

### Keybindings
- Define with `vim.keymap.set` next to the plugin's setup call
- Include `desc` field for all keymaps (required by which-key)
- Use descriptive bracket notation: `[C]ode`, `[D]iff`, `[G]it`

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
- `lua/kickstart/plugins/*.lua`: Optional plugin configurations; `init.lua` does not load them, so add `require 'kickstart.plugins.<name>'` to use one
- `lua/custom/plugins/*.lua`: Custom plugin additions (merge-safe); `init.lua` does not load them, so add `require 'custom.plugins'` to use them
- `lua/kickstart/health.lua`: Health check definitions

### Error Handling
- Use `pcall` for plugin requires when optional: `local ok, plugin = pcall(require, 'plugin')`
- Check plugin availability before use in keymaps/autocommands

### General Principles
- Keep `init.lua` readable with clear section headers
- Document user-configurable settings with comments
- Keep keymaps discoverable via which-key descriptions

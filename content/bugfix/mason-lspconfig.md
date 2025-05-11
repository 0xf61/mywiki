---
title: Failed to run config for nvim-lspconfig after mason upgrade
draft: false
date: 2025-05-10
tags:
  - bugfix
  - neovim
  - lazyvim
---

Hello! I got this error today (2025-05-10) which I seems like related to this [PR](https://github.com/mason-org/mason.nvim/pull/1925).

```
Failed to run `config` for nvim-lspconfig

...share/nvim/lazy/LazyVim/lua/lazyvim/plugins/lsp/init.lua:215: module 'mason-lspconfig.mappings.server' not found:
        no field package.preload['mason-lspconfig.mappings.server']
        cache_loader: module 'mason-lspconfig.mappings.server' not found
        cache_loader_lib: module 'mason-lspconfig.mappings.server' not found
        no file './mason-lspconfig/mappings/server.lua'
        no file '/usr/local/share/lua/5.1/mason-lspconfig/mappings/server.lua'
        no file '/usr/local/share/lua/5.1/mason-lspconfig/mappings/server/init.lua'
        no file './mason-lspconfig/mappings/server.so'
        no file '/usr/local/lib/lua/5.1/mason-lspconfig/mappings/server.so'
        no file '/usr/local/lib/lua/5.1/loadall.so'
        no file './mason-lspconfig.so'
        no file '/usr/local/lib/lua/5.1/mason-lspconfig.so'
        no file '/usr/local/lib/lua/5.1/loadall.so'

# stacktrace:
  - /LazyVim/lua/lazyvim/plugins/lsp/init.lua:215 _in_ **config**
```

To fix this issue I set the version for mason to v1 as suggested [here](https://github.com/LazyVim/LazyVim/issues/6039).

```lua
return {
  { "mason-org/mason.nvim", version = "^1.0.0" },
  { "mason-org/mason-lspconfig.nvim", version = "^1.0.0" },
}
```

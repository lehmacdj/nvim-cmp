# nvim-cmp

A completion plugin for neovim written in Lua.

Status
====================

not yet stable but ok to use (for testing).


Configuration
====================

First, You should install core and sources by your favorite plugin manager.

The `nvim-cmp` sources can be found in [here](https://github.com/topics/nvim-cmp).

```viml
Plug 'hrsh7th/nvim-cmp'
Plug 'hrsh7th/cmp-buffer'
```

Then setup configuration.

```viml
" Setup global configuration
lua <<EOF
  require'cmp'.setup {
    -- You should change this example to your chosen snippet engine.
    snippet = {
      expand = function(args)
        -- You must install `vim-vsnip` if you set up as same as the following.
        vim.fn['vsnip#anonymous'](args.body)
      end
    },
  
    -- You should specify your *installed* sources.
    sources = {
      { name = 'buffer' }
    },
  }
EOF

" Setup buffer configuration
autocmd FileType markdown lua require'cmp'.setup.buffer {
\   sources = {
\     { name = 'buffer' },
\   },
\ }
```

Configuration
====================

The default configuration can be found in [here](./lua/cmp/config/default.lua)

You can use your own configuration like this:
```lua
require'cmp'.setup {
	completion = {
		autocomplete = { .. },
		completeopt = 'menu,menuone,noselect',
		keyword_pattern = [[\%(-\?\d\+\%(\.\d\+\)\?\|\h\w*\%(-\w*\)*\)]],
		keyword_length = 1,
	},
	sorting = {
		priority_weight = 2.,
		comparators = { ... },
	},
	...
}
```

### completion.autocomplete (type: cmp.TriggerEvent[])

Which events should trigger `autocompletion`.

If you leave this empty or `nil`, `nvim-cmp` does not perform completion automatically. 
You can still use manual completion though (like omni-completion).

Default: `{types.cmp.TriggerEvent.InsertEnter, types.cmp.TriggerEvent.TextChanged}`

### completion.keyword_pattern (type: string)

The default keyword pattern.  This value will be used if a source does not set a source specific pattern.

Default: `[[\%(-\?\d\+\%(\.\d\+\)\?\|\h\w*\%(-\w*\)*\)]]`

### completion.keyword_length (type: number)

The minimal length of a word to complete; e.g., do not try to complete when the
length of the word to the left of the cursor is less than `keyword_length`.

Default: `1`


### completion.completeopt (type: string)

vim's `completeopt` setting. Warning: Be careful when changing this value.

Default: `menu,menuone,noselect`


### sorting.priority_weight (type: number)

When sorting completion items before displaying them, boost each item's score
based on the originating source. Each source gets a base priority of `#sources -
(source_index - 1)`, and we then multiply this by `priority_weight`:

`score = score + ((#sources - (source_index - 1)) * sorting.priority_weight)`

Default: `2`

### sorting.comparators (type: function[])

When sorting completion items, the sort logic tries each function in
`sorting.comparators` consecutively when comparing two items. The first function
to return something other than `nil` takes precedence.

Each function must return `boolean|nil`.

Default: 
```lua
{
        compare.offset,
        compare.exact,
        compare.score,
        compare.kind,
        compare.sort_text,
        compare.length,
        compare.order,
}
```

Source creation
====================

If you publish `nvim-cmp` source to GitHub, please add `nvim-cmp` topic for the repo.

You should read [cmp types](/lua/cmp/types) and [LSP spec](https://microsoft.github.io/language-server-protocol/specifications/specification-current/) to create sources.

- The `complete` function is required but others can be omitted.
- The `callback` argument must always be called.
- The custom source only can use `require('cmp')`.
- The custom source can specify `word` property to CompletionItem. (It isn't an LSP specification but supported as a special case.)

```lua
local source = {}

---Source constructor.
source.new = function()
  local self = setmetatable({}, { __index = source })
  self.your_awesome_variable = 1
  return self
end

---Return keyword pattern which will be used...
---  1. Trigger keyword completion
---  2. Detect menu start offset
---  3. Reset completion state
---@return string
function source:get_keyword_pattern()
  return '???'
end

---Return trigger characters.
---@return string[]
function source:get_trigger_characters()
  return { ??? }
end

---Invoke completion (required).
---  If you want to abort completion, just call the callback without arguments.
---@param request  cmp.CompletionRequest
---@param callback fun(response: lsp.CompletionResponse|nil)
function source:complete(request, callback)
  callback({
    { label = 'January' },
    { label = 'February' },
    { label = 'March' },
    { label = 'April' },
    { label = 'May' },
    { label = 'June' },
    { label = 'July' },
    { label = 'August' },
    { label = 'September' },
    { label = 'October' },
    { label = 'November' },
    { label = 'December' },
  })
end

---Resolve completion item that will be called when the item selected or before the item confirmation.
---@param completion_item lsp.CompletionItem
---@param callback fun(completion_item: lsp.CompletionItem|nil)
function source:resolve(completion_item, callback)
  callback(completion_item)
end

---Execute command that will be called when after the item confirmation.
---@param completion_item lsp.CompletionItem
---@param callback fun(completion_item: lsp.CompletionItem|nil)
function source:execute(completion_item, callback)
  callback(completion_item)
end

return source
```


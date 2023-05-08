# vim-lsp [![Gitter](https://badges.gitter.im/vimlsp/community.svg)](https://gitter.im/vimlsp/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

Async [Language Server Protocol](https://github.com/Microsoft/language-server-protocol) plugin for vim8 and neovim.

# Installing

Install [vim-plug](https://github.com/junegunn/vim-plug) and then:

```viml
Plug 'OurCodeBase/vim-lsp'
```

__Performance__

Certain bottlenecks in Vim script have been implemented in lua. If you would like to take advantage of these performance gains
use vim compiled with lua or neovim v0.4.0+

## LanguageServers

```vim
if executable('pylsp')
    " pip install python-lsp-server
    au User lsp_setup call lsp#register_server({
        \ 'name': 'pylsp',
        \ 'cmd': {server_info->['pylsp']},
        \ 'allowlist': ['python'],
        \ })
endif
```

```vim
function! s:on_lsp_buffer_enabled() abort
    setlocal omnifunc=lsp#complete
    setlocal signcolumn=yes
    if exists('+tagfunc') | setlocal tagfunc=lsp#tagfunc | endif
    nmap <buffer> <cr> <plug>(lsp-definition)
    nmap <buffer> gs <plug>(lsp-document-symbol-search)
    nmap <buffer> gS <plug>(lsp-workspace-symbol-search)
    nmap <buffer> gr <plug>(lsp-references)
    nmap <buffer> gi <plug>(lsp-implementation)
    nmap <buffer> gt <plug>(lsp-type-definition)
    nmap <buffer> <leader>rn <plug>(lsp-rename)
    nmap <buffer> [g <plug>(lsp-previous-diagnostic)
    nmap <buffer> ]g <plug>(lsp-next-diagnostic)
    nmap <buffer> <cr> <plug>(lsp-hover)
    nnoremap <buffer> <expr><c-f> lsp#scroll(+4)
    nnoremap <buffer> <expr><c-d> lsp#scroll(-4)

    let g:lsp_format_sync_timeout = 1000
    autocmd! BufWritePre *.rs,*.go call execute('LspDocumentFormatSync')
    
    " refer to doc to add more commands
endfunction

augroup lsp_install
    au!
    " call s:on_lsp_buffer_enabled only for languages that has the server registered.
    autocmd User lsp_buffer_enabled call s:on_lsp_buffer_enabled()
augroup END
```

Refer to [vim-lsp-settings](https://github.com/mattn/vim-lsp-settings) on how to easily setup language servers using vim-lsp automatically.

```viml
Plug 'OurCodeBase/vim-lsp'
Plug 'mattn/vim-lsp-settings'
```

## auto-complete

Refer to docs on configuring omnifunc or [asyncomplete.vim](https://github.com/prabirshrestha/asyncomplete.vim).

## Snippets
vim-lsp does not support snippets by default. If you want snippet integration, you will first have to install a third-party snippet plugin and a plugin that integrates it in vim-lsp.
At the moment, you have following options:
1. [vim-vsnip](https://github.com/hrsh7th/vim-vsnip) together with [vim-vsnip-integ](https://github.com/hrsh7th/vim-vsnip-integ)
```vim
Plug 'hrsh7th/vim-vsnip'
Plug 'hrsh7th/vim-vsnip-integ'
Plug 'OurCodeBase/friendly-snippets'
```

```vim
" Snippets Settings
imap <expr> <Tab>   vsnip#jumpable(1)   ? '<Plug>(vsnip-jump-next)'      : '<Tab>'
smap <expr> <Tab>   vsnip#jumpable(1)   ? '<Plug>(vsnip-jump-next)'      : '<Tab>'
```

For more information, refer to the readme and documentation of the respective plugins.

## Semantic highlighting
vim-lsp supports the unofficial extension to the LSP protocol for semantic highlighting (https://github.com/microsoft/vscode-languageserver-node/pull/367).
This feature requires Neovim highlights, or Vim with the `textprop` feature enabled.
You will also need to link language server semantic scopes to Vim highlight groups.
Refer to `:h vim-lsp-semantic` for more info.

## Supported commands

**Note:**
* Some servers may only support partial commands.
* While it is possible to register multiple servers for the same filetype, some commands will pick only the first server that supports it. For example, it doesn't make sense for rename and format commands to be sent to multiple servers.

| Command | Description|
|--|--|
|`:LspAddTreeCallHierarchyIncoming`| Find incoming call hierarchy for the symbol under cursor, but add the result to the current list |
|`:LspCallHierarchyIncoming`| Find incoming call hierarchy for the symbol under cursor |
|`:LspCallHierarchyOutgoing`| Find outgoing call hierarchy for the symbol under cursor |
|`:LspCodeAction`| Gets a list of possible commands that can be applied to a file so it can be fixed (quick fix) |
|`:LspCodeLens`| Gets a list of possible commands that can be executed on the current document |
|`:LspDeclaration`| Go to the declaration of the word under the cursor, and open in the current window |
|`:LspDefinition`| Go to the definition of the word under the cursor, and open in the current window |
|`:LspDocumentDiagnostics`| Get current document diagnostics information |
|`:LspDocumentFormat`| Format entire document |
|`:LspDocumentRangeFormat`| Format document selection |
|`:LspDocumentSymbol`| Show document symbols |
|`:LspHover`| Show hover information |
|`:LspImplementation` | Show implementation of interface in the current window |
|`:LspNextDiagnostic`| jump to next diagnostic (all of error, warning, information, hint) |
|`:LspNextError`| jump to next error |
|`:LspNextReference`| jump to next reference to the symbol under cursor |
|`:LspNextWarning`| jump to next warning |
|`:LspPeekDeclaration`| Go to the declaration of the word under the cursor, but open in preview window |
|`:LspPeekDefinition`| Go to the definition of the word under the cursor, but open in preview window |
|`:LspPeekImplementation`| Go to the implementation of an interface, but open in preview window |
|`:LspPeekTypeDefinition`| Go to the type definition of the word under the cursor, but open in preview window |
|`:LspPreviousDiagnostic`| jump to previous diagnostic (all of error, warning, information, hint) |
|`:LspPreviousError`| jump to previous error |
|`:LspPreviousReference`| jump to previous reference to the symbol under cursor |
|`:LspPreviousWarning`| jump to previous warning |
|`:LspReferences`| Find references |
|`:LspRename`| Rename symbol |
|`:LspStatus` | Show the status of the language server |
|`:LspTypeDefinition`| Go to the type definition of the word under the cursor, and open in the current window |
|`:LspTypeHierarchy`| View type hierarchy of the symbol under the cursor |
|`:LspWorkspaceSymbol`| Search/Show workspace symbol |

### Diagnostics

Document diagnostics (e.g. warnings, errors) are enabled by default, but if you
preferred to turn them off and use other plugins instead (like
[Neomake](https://github.com/neomake/neomake) or
[ALE](https://github.com/w0rp/ale), set `g:lsp_diagnostics_enabled` to
`0`:

```viml
let g:lsp_diagnostics_enabled = 0         " disable diagnostics support
```

### Highlight references

Highlight references to the symbol under the cursor (enabled by default).
You can disable it by adding

```viml
let g:lsp_document_highlight_enabled = 0
```

to your configuration.

To change the style of the highlighting, you can set or link the `lspReference`
highlight group, e.g.:

```viml
highlight lspReference ctermfg=red guifg=red ctermbg=green guibg=green
```

## Debugging

In order to enable file logging set `g:lsp_log_file`.

```vim
let g:lsp_log_verbose = 1
let g:lsp_log_file = expand('~/vim-lsp.log')

" for asyncomplete.vim log
let g:asyncomplete_log_file = expand('~/asyncomplete.log')
```

You can get detailed status on your servers using `:CheckHealth` -- built into neovim or in a plugin on vim:

```vim
if !has('nvim') | Plug 'rhysd/vim-healthcheck' | endif
CheckHealth
```

## Tests

[vim-themis](https://github.com/thinca/vim-themis) is used for testing. To run
integration tests [gopls](https://github.com/golang/tools/tree/master/gopls)
executable must be in path.

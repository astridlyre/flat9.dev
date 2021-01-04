---
layout: post
title: 'Setting up Neovim for Go & JavaScript development'
date: 2021-01-03
categories: linux neovim vim config
---

![My Neovim Setup](/assets/img/neovim.jpg)

I've been using [SpaceVim](https://www.spacevim.org/) for a little while, after getting
into modal editing through [the VS Code Neovim
extension](https://marketplace.visualstudio.com/items?itemName=asvetliakov.vscode-neovim).
This week I finally decided to try to customize my own configuration. My needs were as
follows:

- I needed LSP (Language Server Protocol) integration
- I needed a solid set up for working with Go and JavaScript projects
- I wanted a fairly minimal setup, that would use FZF (Fuzzy finder) for navigation around
  files and code.

So I started with [Vim Plug](https://github.com/junegunn/vim-plug), a _Minimalist Vim
Plugin Manager_. I set up my config to automatically install if I need to copy it to a
remote server.

```vim
" ================= auto-install vim-plug ================== "

if empty(glob('~/.config/nvim/autoload/plug.vim'))
  silent !curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs
    \ https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
  autocmd VimEnter * PlugInstall | source $MYVIMRC
endif

call plug#begin(expand('~/.config/nvim/plugged'))
```

## Plugins

After the plugin manager is loaded, I install the various plugins I am using. I chose
to go with [vim-airline](https://github.com/vim-airline/vim-airline) for the easy to
configure status line, along with the treesitter-compatible (for syntax highlighting)
theme [gruvbox-material](https://github.com/sainnhe/gruvbox-material).

For more
functionality, I chose [coc](https://github.com/neoclide/coc.nvim). COC, or Conqueror of
Completion, is not just a plugin, but rather a whole ecosystem of tools. While it offers a
ton of functionality, it does come with a rather large footprint. I find the trade-off to
be worth it. A few other plugins that are noteworthy are
[vim-sandwich](https://github.com/machakann/vim-sandwich), for working with text objects
and surrounding them with brackets or quotation marks,
[nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter), for smarter syntax
highlighting, and [vim-go](https://github.com/fatih/vim-go), for the excellent IDE-like
features which make Golang development fun and easy!

```vim
" ================= looks and GUI stuff ================== "

Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'ryanoasis/vim-devicons'
Plug 'sainnhe/gruvbox-material'

" ================= Functionalities ================= "

Plug 'neoclide/coc.nvim', {'branch': 'release'}
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'
Plug 'SirVer/ultisnips'
Plug 'honza/vim-snippets'
Plug 'Yggdroot/indentLine'
Plug 'tpope/vim-liquid'
Plug 'tpope/vim-commentary'
Plug 'mhinz/vim-startify'
Plug 'tpope/vim-fugitive'
Plug 'machakann/vim-sandwich'
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}
Plug 'jiangmiao/auto-pairs'
Plug 'fatih/vim-go', { 'do': ':GoUpdateBinaries' }
call plug#end()
```

## Treesitter

The stated goal of [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) is to provide a simple and easy way to use the
tree-sitter interface in Neovim. The highlighting provided is a huge step up from
traditional RegExp-based highlighting. This plugin used the new Lua API in Neovim for
configuration.

```lua
lua <<EOF
require'nvim-treesitter.configs'.setup {
  ensure_installed = "maintained",
  highlight = {
    enable = true,
  },
}
EOF
```

## General configuration

Neovim differentiates itself from Vim by offering a few more sensible defaults out of the
box, but I did a few more tweaks for my purposes, including some standard performance
tweaks which help when using macros, such as `set lazyredraw`.

```vim
" ==================== general config ======================== "

set termguicolors                                       " Opaque Background
set mouse=a                                             " enable mouse scrolling
set clipboard+=unnamedplus                              " use system clipboard by default
set tabstop=2 softtabstop=2 shiftwidth=2 autoindent     " tab width
set expandtab smarttab                                  " tab key actions
set incsearch ignorecase smartcase hlsearch             " highlight text while searching
set list listchars=trail:»,tab:»-                       " use tab to navigate in list mode
set fillchars+=vert:\▏                                  " requires a patched nerd font (try JetBrains Mono Nerd)
set wrap breakindent                                    " wrap long lines to the width set by tw
set number                                              " enable numbers on the left
set relativenumber                                      " current line is 0
set title                                               " tab title as file name
set noshowmode                                          " dont show current mode below statusline
set noshowcmd                                           " to get rid of display of last command
set conceallevel=2                                      " set this so we wont break indentation plugin
set splitright                                          " open vertical split to the right
set splitbelow                                          " open horizontal split to the bottom
set tw=90                                               " auto wrap lines that are longer than that
set emoji                                               " enable emojis
set history=1000                                        " history limit
set undofile                                            " enable persistent undo
set undodir=/tmp                                        " undo temp file directory
set foldlevel=0                                         " open all folds by default
set inccommand=nosplit                                  " visual feedback while substituting
set showtabline=2                                       " always show tabline
set grepprg=rg\ --vimgrep                               " use rg as default grepper

" performance tweaks
set nocursorline
set nocursorcolumn
set scrolljump=5
set lazyredraw
set redrawtime=10000
set synmaxcol=180
set re=1

" required by coc
set hidden
set nobackup
set nowritebackup
set cmdheight=1
set updatetime=300
set shortmess+=c
set signcolumn=yes
```

## Plugin-specific settings

One of the things I didn't add for my configuration was a file-tree, such as
[nerdtree](https://github.com/preservim/nerdtree). For me, I find it isn't necessary. I
use fuzzy finder for quickly finding files on my Linux system, and it also works well in
Neovim. Make sure to set the `$FZF_DEFAULT_COMMAND` to ignore any folders you don't want
to search, otherwise your list will be dominated by `node_modules`, or other such folders.

I also changed the default save action for vim-go to be `goimports` because I like the
feature of automatically importing any missing packages.

```vim
" FZF
let g:fzf_action = {
  \ 'ctrl-t': 'tab split',
  \ 'ctrl-x': 'split',
  \ 'ctrl-v': 'vsplit' }

let g:fzf_layout = {'up':'~90%', 'window': { 'width': 0.8, 'height': 0.8,'yoffset':0.5,'xoffset': 0.5, 'border': 'sharp' } }
let g:fzf_tags_command = 'ctags -R'

let $FZF_DEFAULT_OPTS = '--layout=reverse --inline-info'
let $FZF_DEFAULT_COMMAND = "rg --files --hidden --glob '!.git/**' --glob '!build/**' --glob '!node_modules/**' --glob '!vendor/bundle/**'"

" vim-go
let g:go_fmt_autosave = 1
let g:go_fmt_command = "goimports"
```

## Custom Mappings

Obviously key bindings are quite personal and there is not right or wrong way to do it,
but maybe you'll find some use seeing what I have come up with. I prefer to change my
`<leader>` key to `<space>` because it is under-utilized and in such a prime location.

```vim
" ======================== Custom Mappings ====================== "

"" the essentials
let mapleader=' '
nmap \ <leader>q
map <F3> :Startify <CR>
nmap <leader>r :so ~/.config/nvim/init.vim<CR>
nmap <leader>q :bd<CR>    " quickly exit a buffer
nmap <leader>\ :qa!<CR>   " quickly exit neovim without saving
nmap <leader>w :w<CR>
nmap <leader>z :Format<CR>
nmap <Tab> :bnext<CR>
nmap <S-Tab> :bprevious<CR>
noremap <leader>e :PlugInstall<CR>
noremap <C-q> :q<CR>
map Y y$                  " make Y consistent with C and D
inoremap jk <ESC>
cnoremap jk <ESC>
nnoremap <leader>y +y
nnoremap <leader>Y +Y
nnoremap <leader>p +p

" mapping to move lines up or down with alt+j / alt+k
nmap <M-j> mz:m+<cr>`z
nmap <M-k> mz:m-2<cr>`z
vmap <M-j> :m'>+<cr>`<my`>mzgv`yo`z
vmap <M-k> :m'<-2<cr>`>my`<mzgv`yo`z

" insert a new line in normal mode and return to cursor position
noremap <leader>[ myO<ESC>`y
noremap <leader>] myo<ESC>`y

" switch between splits using ctrl + {h,j,k,l}
inoremap <C-h> <C-\><C-N><C-w>h
inoremap <C-j> <C-\><C-N><C-w>j
inoremap <C-k> <C-\><C-N><C-w>k
inoremap <C-l> <C-\><C-N><C-w>l
nnoremap <C-h> <C-w>h
noremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l

" disable search highlighting with 2 esc
noremap <silent><esc> <esc>:noh<CR><esc>

" trim white spaces on keypress
nnoremap <F2> :let _s=@/<Bar>:%s/\s\+$//e<Bar>:let @/=_s<Bar><CR>

"" FZF
nnoremap <silent> <leader>f :Files<CR>
nmap <leader>b :Buffers<CR>
nmap <leader>c :Commands<CR>
nmap <leader>t :BTags<CR>
nmap <leader>/ :Rg<CR>
nmap <leader>gc :Commits<CR>
nmap <leader>gs :GFiles?<CR>
nmap <leader>sh :History/<CR>

" show key mapping on all modes with F1
nmap <F1> <plug>(fzf-maps-n)
imap <F1> <plug>(fzf-maps-i)
vmap <F1> <plug>(fzf-maps-x)

"" coc

" use tab to navigate snippet placeholders
inoremap <silent><expr> <TAB>
      \ pumvisible() ? "\<C-n>" :
      \ <SID>check_back_space() ? "\<TAB>" :
      \ coc#refresh()
inoremap <expr><S-TAB> pumvisible() ? "\<C-p>" : "\<C-h>"

" Use enter to accept snippet expansion
inoremap <expr> <cr> pumvisible() ? "\<C-y>" : "\<CR>"

" Use `[g` and `]g` to navigate diagnostics
nmap <silent> [g <Plug>(coc-diagnostic-prev)
nmap <silent> ]g <Plug>(coc-diagnostic-next)

" GoTo code navigation
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)

" Use CTRL-S for selections ranges.
nmap <silent> <C-s> <Plug>(coc-range-select)
xmap <silent> <C-s> <Plug>(coc-range-select)

" Use K to show documentation in preview window.
nnoremap <silent> K :call <SID>show_documentation()<CR>

" symbol rename
nmap <leader>lrn <Plug>(coc-rename)
nmap <leader>o :OR <CR>

" jump stuff
nmap <leader>jd <Plug>(coc-definition)
nmap <leader>jy <Plug>(coc-type-definition)
nmap <leader>ji <Plug>(coc-implementation)
nmap <leader>jr <Plug>(coc-references)

" vim-go mappings
nmap <leader>lt :GoTest<CR>
nmap <leader>ll :GoMetaLint<CR>
nmap <leader>lc :GoCoverageToggle<CR>
nmap <leader>la :GoAlternate<CR>
nmap <leader>lr :GoRun<CR>
nmap <leader>li :GoImports<CR>
nmap <leader>lfs :GoFillStruct<CR>
nmap <leader>lie :GoIfErr<CR>
```

## Full Configuration

Hopefully that provided some inspiration! I did not include the entire file in these
snippets, but you can find the complete `init.vim` [here](https://gitlab.com/ebflat9/dotfiles/-/blob/master/.config/nvim/init.vim),
in addition to some of my other config files.

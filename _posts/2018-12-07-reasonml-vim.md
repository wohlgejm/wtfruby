---
layout: post
comments: true
title: ReasonML and vim
date: 2018-12-07 8:20:30
categories: postgres
short_description: Vim config for ReasonML
image_preview: /images/bikes.gif
---

Here's how to get started with vim and [ReasonML](https://reasonml.github.io/).

Start by installing `bs-platofrm`.
```bash
npm install -g bs-platform
```

For syntax highlighting, install [vim-reason-plus](https://github.com/reasonml-editor/vim-reason-plus).
```bash
Plugin 'reasonml-editor/vim-reason-plus'
```

For type hints and autocompletion, you can use [vim-lsp](https://github.com/prabirshrestha/vim-lsp).
```bash
Plugin 'prabirshrestha/async.vim'
Plugin 'prabirshrestha/vim-lsp'
Plugin 'prabirshrestha/asyncomplete.vim'
Plugin 'prabirshrestha/asyncomplete-lsp.vim'
```

Next, install the [ocaml-language-server](https://github.com/freebroccolo/ocaml-language-server). This requires [Merlin](https://github.com/ocaml/merlin), which you can install using another set of tools, [reason-cli](https://github.com/reasonml/reason-cli).
```bash
npm install -g reason-cli@latest-linux
npm install -g ocaml-language-server
```

Lastly, add the following to your `.vimrc`.
```bash
if executable('ocaml-language-server')
    au User lsp_setup call lsp#register_server({
      \ 'name': 'ocaml-language-server',
      \ 'cmd': {server_info->[&shell, &shellcmdflag, 'ocaml-language-server --stdio']},
      \ 'whitelist': ['reason', 'ocaml'],
      \ })
endif
```

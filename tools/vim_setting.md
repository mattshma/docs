# vim优化

## vundle插件管理
[vundle](https://github.com/VundleVim/Vundle.vim.git)是vim的一个插件管理器。

## powerline

安装见[文档](https://powerline.readthedocs.io/en/latest/installation/linux.html#font-installation)。

配置如下:
```
Bundle 'Lokaltog/vim-powerline'
" set guifont=PowerlineSymbols
" let g:Powerline_symbols = 'fancy'
let g:Powerline_symbols = 'unicode'
set laststatus=2
set noshowmode
set t_Co=256
```

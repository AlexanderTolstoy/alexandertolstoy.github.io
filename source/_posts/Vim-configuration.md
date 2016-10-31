---

title: 设置一个强大好用的Vim使用环境

categories: 
- OS
- Programer 

tags: 
- Vim  
- configuration  

---

## 参考 ##

### Vim basic###

- [Vim.org](http://www.vim.org/)
- [Vimer](http://www.vimer.cn/)
- [use vim as a IDE](https://github.com/yangyangwithgnu/use_vim_as_ide)
- [VundleVim](https://github.com/VundleVim/Vundle.Vim)
- [Vim-pathogen](https://github.com/tpope/vim-pathogen)

### Vim for languages ###

- [plasticboy/vim-markdown](https://github.com/plasticboy/vim-markdown)

    
<!-- more -->

### Installation of Vundle ###

```bash

mkdir -p ~/.vim/bundle

cp /etc/vimrc  ~/.vimrc

cd ~/.vim/bundle

git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim


```

Put this at the top of your .vimrc to use Vundle. Remove plugins you don't need, they are for illustration purposes.

```bash

set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle"begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle"begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
" Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
" Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Install L9 and avoid a Naming conflict if you've already installed a
" different version somewhere else.
Plugin 'ascenator/L9', {'name': 'newL9'}

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just
" :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line

```


### Markdown's ENV  ###

if you use Vundle, add the following line to your ~/.vimrc:

```bash

Plugin 'godlygeek/tabular'
Plugin 'plasticboy/vim-markdown'

```

Then run inside Vim:  

> :so ~/.vimrc
> :PluginInstall



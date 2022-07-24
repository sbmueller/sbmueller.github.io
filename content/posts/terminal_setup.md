---
title: "My Terminal Setup"
date: 2019-05-19T16:20:37+01:00
tags: ["hacking"]
draft: false
---

## On the Terminal
Why is the terminal still something important in the year 2019 you might ask.
There are GUI applications for literally any purpose, so why bother setting up
a terminal environment where you have to enter complicated commands?

It surely depends on the kind of work you are doing, but for most engineers and
developers, a neat terminal setup provides some crucial benefits. First of all,
this kind of workflow has to be embraced and is not for everyone. I moved most
of my workflow (except Email and Web-Applications) to the terminal while many
of my colleagues prefer GUI applications for each part of their workflow.
However, I love my setup for one reason: I'm much faster.

One reason for that is that I hardly need to move my hands during work. I
rarely have to reach for the mouse, move the mouse to a certain location and
perform a single action. My hands remain mostly in the same position while I'm
working, eliminating unnecessary movement. I'm using __vim key bindings__ as
well in my setup, which for instance maps the arrow keys to h, j, k, l. This is
where the fingers of my right hand are positioned most of the time.

A smart way to bind keys to certain actions allows to have multiple command
prompts opened for different things and quickly be able to switch between them.
I use [tmux](https://github.com/tmux/tmux) to enable this. So for instance in
one split, I'm editing a source code file in [vim](https://github.com/vim/vim)
while in the other, I'm executing [git](https://github.com/git/git) commands. I
found this to be much quicker than to have two GUI applications running and
using the mouse to operate them. Since I'm using the same keystrokes every day,
I have memorized them, and they are performed completely automatically, without
me putting any effort into it.

A second aspect is that while GUI applications might seem easier to operate,
I don't believe they are generally faster to use. I have tried to use a
graphical git client recently and often where stuck on where to find a certain
button to perform a basic git action ("I just want to do a `git rebase master`,
why is this so complicated?"). Surely you can train and become used to it, but
then again you can just learn how to use the CLI application. Also, keep in mind
that the GUI application adds one layer of abstraction, potentially making it
harder to find the source of a problem.

## My Terminal
So here is my setup as I'm writing this post:<br>
![iTerm2](/images/iterm.png)
I have chosen a vertical split since the preview of this article is on the
second half of the screen, in a web browser. Normally, I work with several tmux
windows and up to two splits.

As you can see, I use vim in the upper split as my text editor, writing these
lines. In the lower half, I'm in the directory of this website project, ready to run
some [hugo](https://github.com/gohugoio/hugo) (the software to create this website)
or git commands.

Let's dive into some detail. The application I'm running here is
[iTerm2](https://github.com/gnachman/iTerm2), a great terminal emulator for
MacOS. Within there, I'm running a tmux session, cognizable by the status bar
at the bottom. I have two tmux windows open, and currenlty I'm working on my
blog in parallel to my secret project. I can create
several windows and quickly switch between them. Note that the **splits are
part of one window** so It's possible to build up different windows for
different projects.

I have split the tmux window in two vertical splits. I can quickly switch focus
beteen them. In each split, a
[zsh](https://github.com/zsh-users/zsh) shell is running. I'm using many
extensions, mainly [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) along with
[Spaceship Prompt](https://github.com/denysdovhan/spaceship-prompt). As you can
see, I can directly see the status of my git repository without running git
itself. The colors are [material-dark](https://github.com/MartinSeeler/iterm2-material-design)
and the font I use is called [Fura Code NF](https://github.com/ryanoasis/nerd-fonts).
This font enables me to use glyphs as well, though I'm not missing it
currently.

Some handy CLI tools I don't want to miss here are
[fasd](https://github.com/clvv/fasd), which allows me to `cd` to certain
directories instantly, and [ranger](https://github.com/ranger/ranger), a file
explorer for the terminal.

## My Vim
I *really* love [vim](https://github.com/vim/vim). In early 2018 I finally
swiched to vim as my one and only editor/IDE and never looked back. I use it
for anything at work and privately. There are a lot of resources on why vim is
great, but I want to summarize it like this:

> Vim focuses on text editing instead of text writing. In normal mode, the
> keyboard becomes a control panel for altering the content of a file rather
> than adding letters. So instead of letter keys producing letters in the file,
> the keys are mapped to various navigation and modifying functions. This
> enables an incredibly fast workflow, if you're willing to climb the steep
> learning curve.

### Vim Plugins

Vim is extremely customizable with plugins. Plugins I use include:

- [YouCompleteMe](https://github.com/Valloric/YouCompleteMe) Code autocompletion with great Python and C/C++ support.
- [nerdtree](https://github.com/scrooloose/nerdtree) File explorer in vim.
- [fzf.vim](https://github.com/junegunn/fzf.vim) Fuzzy file search.
- [vim-fugitive](https://github.com/tpope/vim-fugitive) Trigger git commands from within vim.
- [vim-airline](https://github.com/vim-airline/vim-airline) Fancy status bar.
- [nerdcommenter](https://github.com/scrooloose/nerdcommenter) Quickly comment/uncomment code parts.
- [vim-grammarous](https://github.com/rhysd/vim-grammarous) Grammar checker.
- [gundo.vim](https://github.com/sjl/gundo.vim) Visual history of changes to a file.

### Useful Vim Settings
There are countless settings for vim. I want to share some handpicked ones:

- `set colorcolumn=85` shows a vertical ruler in column 85
- `set mouse=a` activate mouse support (sometimes handy)
- `set hidden` allow hidden buffers (switch to another buffer even if changes
  are unsaved)
- `set foldmethod=indent` automatically fold code (I can't live without this
  one)
- `set undofile` enable persistent undo history
- `set undodir=~/.vim/undodir` location for persistent history files
- Also, I use `Tab` and `Shift-Tab` to switch buffers and `Space` to jump
  between splits:

```
nnoremap <space> <C-W><C-W>
nnoremap <tab> :bn<CR>
nnoremap <S-tab> :bp<CR>
```

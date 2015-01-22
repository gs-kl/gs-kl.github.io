---
layout: post
title: "Smart Tmux split creation from within Vim"
---
While running Vim inside of Tmux, I often want to create a new Tmux split that opens in Vim's working directory or the directory of the current Vim file. Here's how I made keymappings for that in Vim.


## Background
The initial `tmux new-session` or `tmux attach-session` command determines a _session working directory_, which if not specified with the `-c` flag defaults to the directory in which that command was run. All future windows and splits created in that session open in the session working directory unless a _start directory_ is specified, again with the `-c` flag.

So, for example, if I ran `tmux` in `/Users/kevin`, `cd`ed to `/Users/kevin/Documents`, and created a new horizontal split with `Prefix-"`, it would open in `/Users/kevin`. If I wanted to open the split in `/Users/kevin/Documents`, I could enter the shell command `tmux sp -c ${PWD}`. To always open horizontal splits in the current directory, I could create a shell alias for the aforementioned command, or I could add a line in my `.tmux.conf` rebinding `Prefix-"` to a custom command; [this Stack Overflow thread](http://unix.stackexchange.com/questions/12032/create-new-window-with-current-directory-in-tmux) provides ideas for that sort of thing. 

But I need the new split to get its `-c` from Vim, not Tmux or the shell. Happily, this turns out to be very simple -- we just have to give the directory name to Vimscript's `system()` function, which runs `tmux sp -c [path]` in the background to accomplish the desired effect.


## Code
Here are the Vimscript functions the keybindings will use. The code is pretty self-explanatory; just note that `split-window` is the complete version of `sp`, written out here for clarity, and that in Tmux `-v` means "horizontal" and `-h` means "vertical."

{% highlight bash %}
function! OpenTmuxSplitHorizontal(directory)
  call system("tmux split-window -v -c " . a:directory)
endfunction

function! OpenTmuxSplitVertical(directory)
  call system("tmux split-window -h -c " . a:directory . " -p 20")
endfunction
{% endhighlight %}

And the keymappings themselves. `expand("%:p:h")` returns the head (`h`; that is, minus the filename) of the absolute path (`p`) of the current file (`%`). `getcwd()` returns the absolute path of the working directory.


{% highlight text %}
map <leader>" :call OpenTmuxSplitHorizontal(expand("%:p:h"))<CR>
map <leader>' :call OpenTmuxSplitHorizontal(getcwd())<CR>
map <leader>% :call OpenTmuxSplitVertical(expand("%:p:h"))<CR>
map <leader>5 :call OpenTmuxSplitVertical(getcwd())<CR>
{% endhighlight %}


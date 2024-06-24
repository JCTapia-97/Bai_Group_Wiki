When working on the group server or HPCs, we generally don't use
graphical user interfaces (GUIs), rather we primarily interact with the
machines through the command-line interface (CLI). Various shell
environments exist but the most common are Bourne-Again Shell (Bash),
the default for Ubuntu/WSL; Z shell (Zsh), the default for MacOS; and
PowerShell, only used on Windows machines. Zsh is built on top of Bash
so the commands are relatively interchangeable but if you'd like more
information about the differences,
[this](https://apple.stackexchange.com/questions/361870/what-are-the-practical-differences-between-bash-and-zsh)
stack exchange post describes the difference comprehensively. The
primary difference to note is the name of the default configuration
environment for each with `.bashrc` and `.bash_profile` being for Bash
and `.zshrc` and `.zprofile` for Zsh. Powershell isn't used often in our
group workflow as most machines use a Linux-based environment. However,
if you need to do anything on your Windows machine outside of WSL, it
may be useful to know a little about it. The syntax and commands are
very different between PowerShell and Bash/Zsh.

## Tutorials

### Bash/Zsh

- The
  [Ubuntu](https://ubuntu.com/tutorials/command-line-for-beginners#1-overview)
  website provides a good tutorial on how to start with the command line
  and some basics with plumbing. If you're using WSL and are not in the
  Ubuntu operating system (OS), WSL is already the Linux terminal, you
  can skim over the first part in section 3.
- For Bash scripting
  [geeksforgeeks](https://www.geeksforgeeks.org/bash-scripting-introduction-to-bash-and-bash-scripting/)
  provides good basics for scripting and program control methods
- For Zsh scripting [Austin Traver's site](https://helpful.wiki/zsh/)
  provides a more comprehensive guide on scripting along with various
  other tutorials for MacOS

### PowerShell

- Microsoft provides [tutorials for
  PowerShell](https://learn.microsoft.com/en-us/training/modules/introduction-to-powershell/)
  on their website

## Tips/Tricks/Tools

### Terminal Emulators

While the stock terminal emulator for most OS's are good enough, added
functionality and customizability can be gleaned by using other
programs. The most popular ones are listed below

- MacOS: [iTerm2](https://iterm2.com/index.html) has a lot of
  customizability and extra features compared to Terminal and a large
  following
- Windows: [Windows
  Terminal](https://apps.microsoft.com/detail/9n0dx20hk701?hl=en-US&gl=US)
  is the replacement for Windows Console and can run any command-line
  app in separate tabs

### Programs

Many of these programs are optional as they fix some niche issues. I
would recommend not using/installing these programs at first until
you're comfortable with the default methods they replace as they won't
be available on all computers.

- [Neovim](https://neovim.io/) or `nvim` is a modernized version of
  `vim` which is extremely modifiable and uses `Lua` for plugins.
  [Kickstart](https://github.com/nvim-lua/kickstart.nvim) is a good
  starting point for plugins which you can choose to edit later
- [tmux](https://github.com/tmux/tmux/wiki) is a terminal multiplexer,
  allowing you to switch between programs and/or run them in background.
  This [youtube video](https://www.youtube.com/watch?v=DzNmUNvnB04)
  provides a good starting point for `.tmux.conf`.
- [fzf](https://github.com/junegunn/fzf) is an interactive filter which
  uses a fuzzy matching algorithm. Very useful for reverse searching
  commands and works well with `tmux` and `nvim`.
- [Zoxide](https://github.com/ajeetdsouza/zoxide) is a replacement for
  cd which uses `fzf` to more easily switch between commonly used
  directories.

### Tips

- For Window Terminal, unbind copy and paste from `ctrl + c` and
  `ctrl + v` and replace it with `ctrl + shift + c` and
  `ctrl + shift + v` respectively. This will allow you to do the
  following shortcuts
- Pressing up on the arrow keys allows you to pull previously input
  commands
- `esc` then the left or right arrow key can move you by word (For
  MacOS, the `option` key also works)
- Hidden files start with `.` like `.bashrc` above. These files won't
  normally be shown when using `ls`. Use `ls -a` to see all files,
  including hidden files or better yet, use `ls -halF`. Even better yet,
  the group server's default `.bashrc` file has `ll` aliased to
  `ls -alF`.

### Terminal Shortcuts

Shortcuts are applicable for both Linux/WSL and MacOS terminals. For
MacOS, shortcuts are as is (i.e. do not replace `ctrl` with `cmd`)

- `ctrl + a`: Move the curser to the start of the line
- `ctrl + e`: Move the curser to the end of the line
- `ctrl + c`: clear current line/kill current process
- `ctrl + l`: clear entire terminal
- `ctrl + z`: Pause the current process (can be restarted with the
  command `fg`)
- `ctrl + r`: Reverse search previously input command

### Dotfile Management

The configuration files for various programs are hidden files (e.g.
`.bashrc`, `.zshrc`, `.tmux.conf`) and are usually in your `$HOME`
directory. This is fine at first but when you configure your environment
more, it quickly becomes difficult to manage them, especially when you
have multiple computers you work with. The following
[tutorial](https://www.atlassian.com/git/tutorials/dotfiles) will help
with managing dotfiles through a bare git directory. This method is
slightly more difficult as you need some basic understanding of git but
is good as git is installed on pretty much every HPC we use. Another
method would be to [GNU Stow](https://www.gnu.org/software/stow/) but
this would require downloading the program on the HPC first.

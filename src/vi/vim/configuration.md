# Configuration

## Clipboard

This is one of the first steps to work with Vim, for example, to copy and paste between different Vim instances.

### Command

Checking for X11-clipboard support in terminalEdit.

```bash
vim --version | grep clipboard
```

If you see -clipboard or -xterm_clipboard, use te following commands.

```bash
sudo apt-get install vim-gtk
```

### Resources

<https://vim.fandom.com/wiki/Accessing_the_system_clipboard>

## Config file

### Modify config file

```bash
vim ~/.vimrc
```

Note. If the previous file does not exist, create it in order to avoid edit /etc/vim/vimrc as a super user.

### Configuration example

See <https://github.com/CarlosAMolina/dotfiles/blob/main/dotfiles/config/vim/.vimrc>.


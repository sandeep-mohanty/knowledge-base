# 8 Linux commands so good, they feel like cheating

While the Linux command set is great on its own, it seems that programmers can't stop reinventing it. Here are some of the best modern takes on classic Linux tools that fix some common annoyances with Unix-like systems. You might wonder how you ever got by before you knew about them like I did.

## Search text quickly with fzf

   ![fzy find showing list of files with "py" inside them.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/fzf-py-search.png?q=49&fit=crop&w=825&dpr=2)

To find files on Linux, you might use the ls or even the find command. While you can locate exact filenames and use wildcards to to generate lists of files to pass to a command, [fzf](https://github.com/junegunn/fzf), for "fuzzy find," lets you search output in real. The "fuzzy" part comes from how it will show the search string highlighted in the list of files. fzf is a full-screen terminal program. With [the find command's](https://www.howtogeek.com/771399/how-to-use-the-find-command-in-linux/) notoriously abstruse syntax, fzf is a welcome replacement.

It's also useful as a replacement for grep. fzf is great for piping things into for searching.

To install it from Debian or Ubuntu:

```
sudo apt install fzf
```

To install it on Arch:

```
sudo pacman -Syu fzf
```

On Fedora and other Red Hat family distros:

```
sudo dnf install fzf
```

## Find info on commands quickly with tldr

    ![The tldr command showing help on itself.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/the-tldr-command-showing-help-on-itself.jpg?q=49&fit=crop&w=825&dpr=2)Credit: Bobby Jack / How To Geek

If you ever read [Linux manpages](https://www.howtogeek.com/663440/how-to-use-linuxs-man-command-hidden-secrets-and-basics/), it seems like you're drinking from a firehose of technical information. Linux programs have a lot of options, but it can be frustrating to read through these walls of text to find the one youneed. Fortunately, [tldr](https://github.com/tldr-pages/tldr) can help.

For example:

```
tldr ls
```

[tldr displays a summary](https://www.howtogeek.com/323020/tldr-converts-man-pages-into-concise-plain-english-explanations/) of a command-line program's function with some common options. You'll likely find the one you want to use.

You can install it via pip, regardless of distro:

```
pip install tldr
```

## ack: Another grep replacement

   ![achk higlighting of search for "root" with the shell code, "ps aux | ack" redircting the output of ps into ack.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/ack-ps-aux.png?q=49&fit=crop&w=825&dpr=2)

[ack](https://beyondgrep.com/) is a tool that's meant to replace [grep, the tool for searching through text](https://www.howtogeek.com/496056/how-to-use-the-grep-command-on-linux/). Like grep, it also has a funny name, but it's a serious tool. It's designed with programmers in mind and knows how to avoid irrelevant files in output. It takes Perl regular expressions and will highlight the matches in the search input, although most modern implementations of grep do the same thing. It can also use "proximate searches" to find matches that are close but not exact.

You can install ack via Debian or Ubuntu:

```
sudo apt install ack
```

And in Arch:

```
sudo pacman -Syu ack
```

And in Fedora:

```
sudo dnf install ack
```

## mtr: It's both a ping and traceroute!

   ![MTR for google.com in the WSL terminal.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/01/mtr-terminal.png?q=49&fit=crop&w=825&dpr=2)

If you do any networking, you're likely familiar with running [the ping command](https://www.howtogeek.com/355664/how-to-use-ping-to-test-your-network/) to see if your connection is up. You may also be familiar with [traceroute or tracepath](https://www.howtogeek.com/134132/how-to-use-traceroute-to-identify-network-problems/), a utility that lets you see the paths your packets are taking across the internet to the their destinations. [mtr](https://www.bitwizard.nl/mtr/) ("My Tracroute," formerly "Matt's Traceroute") is a command that lets you combine both into one program. Like ping, it sends out packets and listens for an answer from a remote server. Like traceroute, it traces the paths in real time.

You can install it on Debian and Ubuntu:

```
sudo apt install mtr
```

And on Arch:

```
sudo pacman -Syu mtr
```

And on Fedora

```
sudo dnf install mtr
```

## eza: Stylish file listing

   ![Listing of /bin directory using eza in the Linux terminal.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/eza-bin-listing.png?q=70&fit=crop&w=825&dpr=1)

[eza](https://github.com/eza-community/eza/tree/main) is a program that aims to replace ls as the tool to list files on Linux. It seems superficially similar to ls but it has some features to make it more readable. It has color schemes that work with both light and dark terminals. The long mode, or -l, looks similar to ls, but all of the columns are colored.

eza is evailable on Debian and Ubuntu:

```
sudo apt install eza
```

And in Arch:

```
sudo pacman -Syu eza
```

It's not in the repositories for Fedora, but you can download a binary version from the website.

## bat/batcat: A slick take on a basic Linux utility

   ![Fortune command redirected into batcat with line numbers.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/fortune-batcat.png?q=70&fit=crop&w=825&dpr=1)

bat is a utility that aims to replace [the classic cat command](https://www.howtogeek.com/ways-to-use-the-linux-cat-command/). It works the same way: it concatenates [standard input](https://www.howtogeek.com/435903/what-are-stdin-stdout-and-stderr-on-linux/) (keyboard input or from files) to the standard input (such as a terminal or a [teletype](https://www.howtogeek.com/727213/what-are-teletypes-and-why-were-they-used-with-computers/) in the olden days).

You can use it the same way that you would use cat, but on Ubuntu, the name of the utility has been renamed to "batcat" for some reason. Fortunately, you can use a [shell alias](https://www.howtogeek.com/why-you-should-be-using-aliases-in-the-linux-terminal/) if you want. This one should work in most Bourne-type shells, such as Bash or zsh:

```
alias bat="batcat"
```

You can drop this in your shell's startup files, such as the .bashrc or .zshrc.

You can obviously install it on Debian or Ubuntu:

```
sudo apt install bat
```

And on Arch:

```
sudo pacman -Syu bat
```

And on Fedora:

```
sudo dnf install bat
```

## ripgrep: Easy searching through your directories

   ![ripgrep search for "python" in the terminal.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/ripgrep-python.png?q=49&fit=crop&w=825&dpr=2)

The name [ripgrep](https://github.com/BurntSushi/ripgrep) should give you a clue as to [what utility it's intended to replace](https://www.howtogeek.com/496056/how-to-use-the-grep-command-on-linux/). One key difference of ripgrep from grep is that it recursively searches subdirectories. I don't think it would actually replace grep for standard input and output, since it's designed to work with files and directories, but it's a more than welcome replacement for find.

Like grep, it searches for files using regular expressions, which let you specify matches down to the character. To search recursively for the term python in the current directory:

```
rg python
```

This will also search the subdirectories relative to the current directory.

To install it on Debian or Ubuntu:

```
sudo apt install ripgrep
```

And on Arch:

```
sudo pacman -Syu
```

And on Fedora:

```
sudo dnf install ripgrep
```

gping

   ![gping showing graphical ping times from Google.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/gping-google.png?q=49&fit=crop&w=825&dpr=2)

While the ping tool is useful, [gping](https://github.com/orf/gping) offers a visual display of ping times in the terminal. It's pretty neat to watch. It's interesting how the time path of ping times shows a sinusoidal pattern.

You can use it the same way as you would the regular [ping command](https://www.howtogeek.com/355664/how-to-use-ping-to-test-your-network/)

```
gping google.com
```

To install it on Debian/Ubuntu:

```
sudo apt install gping
```

And on Arch:

```
sudo pacman -Syu gping
```

And on Fedora

```
sudo dnf install gping
```

* * *

It seems that as great as the standard Linux command set is, things can always be improved. As a Linux user, it pays to be open to new utilities. You never know when you might find one that makes working with files and networks easier.
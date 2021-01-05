---
layout: post
title: 'Learning Bash Scripting'
date: 2021-01-04
categories: linux bash scripting
---

One of the most important places to start when learning Linux is understanding the command
line, or terminal. The default terminal emulator installed on most Linux distributions is
the Bourne Again Shell or `bash`. Bash is a scripting language and also an interactive
environment where a user can interact with their computer using text commands.

## The basic grammar of a bash command

An example command would be the basic `ls` command used to list the content of a directory.
Simple commands are structured as follows: the values in square brackets
are optional and are variable in number.

```bash
# [ var#value ... ] name [ arg ... ] [ redirection ... ]
ls -a
# "ls" is the name, and "-a" is the arg
```

## The power of redirection

The great thing about Linux (or rather, the _Unix Philosophy_) is that each simple command
is capable of being used within a greater system. Like in Vim/Neovim, where you learn to
speak a kind of "language", the true power of the command line is when you start
redirecting the output of one command into another.

```bash
# Redirects the output of echo into a file hello.tx
echo "Hello World!" > hello.txt

# Tries to remove hello.txt or displays an error
rm hello.txt || echo "Unable to delete file" >&2
```

In addition to redirecting, as seen above it is also possible to use `||` to perform a
second command conditionally, if the first command were to fail. The output of the second
command is then redirected to the `stderr` or `FD2`, the error stream. There is so much
more to `bash` than simply running commands in the terminal, and a full tutorial is not my
goal here, but you can also create functions and use if/then statements, among other idiomatic
programming devices.

## Baby's first script

One of the first scripts I have written is a simple script to keep my installed version of
[Neovim](https://neovim.io) up-to-date. It is important to start a `bash` script with a
[shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>), in order to ensure the proper
interpreter is used to run the script. The _shebang_ for a Bash script is `#!/bin/bash`,
which points to the executable file in the system binaries directory.

The first thing to do is write a short little error handler, in case something goes
horribly awry! This little snippet will echo the error message, along with the name of the program
that has suffered the error. `PROGNAME` is a variable which is assigned a value with `=`, and
then it can be used when prefixed with a `$`.

```bash
#!/bin/bash
PROGNAME="${basename $0}"

error_exit()
{
	echo "{$PROGNAME}: ${1:- "Unknown Error"}" 1>&2
	exit 1
}
```

Next, the script uses `wget` to download the latest AppImage from Neovim's GitHub releases
page. What's the difference between `wget` and `curl` you might ask? Well, in this case,
they satisfy completely different needs. Wget is excellent at retrieving files from remote
locations recursively, using intelligent routines. It excels at downloading files. Curl,
on the other hand is much more versatile. It supports over 20 different protocols, and
while it is arguably easier to integrate with other commands, it did not fit my use case
here.

```bash
echo "Updating neovim"

if wget "https://github.com/neovim/neovim/releases/download/nightly/nvim.appimage"; then
	mv ./nvim.appimage $HOME/Applications/nvim || error_exit "Unable to move file"
	chmod +x $HOME/Applications/nvim || error_exit "Unable to change file permissions"
	echo "Update complete"
else
	error_exit "Update failed, unable to download file"
fi
# Hopefully everything went well!
```

## More resources

If you want to learn more about `bash` scripting, the Linux command line, here are a few
links you might find interesting.

- [Bash Academy](https://guide.bash.academy)
- [The Linux Command Line](https://www.linuxcommand.org/tlcl.php)
- A few useful tools to explore with `curl`, such as [cht.sh](https://cht.sh/),
  [wttr.in](https://wttr.in/), or [rate.sx](https://rate.sx/).

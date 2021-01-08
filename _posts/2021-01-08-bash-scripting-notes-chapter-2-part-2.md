---
layout: post
title: 'Bash Academy, Chapter Two, Part Two Notes'
date: 2021-01-08
categories: linux bash scripting
---

Simple commands are the foundation of bash. Everything else is contingent upon a solid
understand of how to structure and use this syntax.

## Command names

The first thing that happens when bash is given a command is that it searches, in order to
find out what to do. This search goes in order:

1. _Function_, a previously declared block of code.
1. _Built-in_, mini-functions that come with bash.
1. _External command_, a program installed on the system that is able to be found in the
   `PATH` environment variable. If such a program is not found, then bash will return
   an error like `<program name>: command not found`.

## The `PATH` variable

Linux programs are installed in various standard places, such as `/bin`, `/usr/bin`, etc.
The `PATH` variable contains a colon-separated list of where to search for programs given
to bash as a command to execute.

```bash
# To see where bash finds a program use "type"
type ping
# => ping is /usr/bin/ping

type echo
# => echo is a builtin
```

If the program is not in the system `PATH`, its location must be specified exactly, for
example: `/usr/bin/ping -c 1 bitcointicker.co`. It is also possible to add further
locations to the `PATH` variable.

```bash
# Updating the variable to include a new program
# This change only lasts through the current session
PATH=$PATH:/home/user/cat-trivia
```

## Command arguments

To the bash shell, blank space is syntax. It means to separate one thing from the next.
This is called _word splitting_. If spaces are part of an argument, they must either be
_quoted_ or _escaped_. One of the most important skills to master is quoting, as it is the
easiest way to avoid many common mistakes with command arguments.

```bash
# Quoting
ffplay "Baby's first movie.mp4"
# Escaping

mv My\ Cool\ Directory my-cool-directory
```

> "If there is whitespace or a symbol in your argument, you must quote it. If there isn't,
> quotes are usually optional, but you can still quote it to be safe." [Bash Academy](https://guide.bash.academy/commands/?=Command_arguments_and_quoting_literals#a3.3.0_3)

In order to tell a command what to do, one must pass it _arguments_. In bash, these
arguments are separated by spaces. So, it is good practice to quote arguments because
there is not harm in doing it even if it was unnecessary.

## Redirection

In some cases, it is useful to change where the output of a command ends up. For instance,
it is possible to redirect output into a file with the `>` right angle bracket.

```bash
# Redirecting a list of files in /bin to a text file
ls -la /bin > programs.txt

# Redirecting an error message to a file
ls -l /notarealfolder 2> oupsies.txt
```

A common use case for redirection is to _copy_ file descriptors, or in other words, to
connect a file descriptor's stream to another file descriptor. Often it is useful to
redirect FD2 to FD1 `2>&1`. This makes standard error write to where standard output is
writing.

### Some more examples of redirection

```bash
# File redirection
echo "option=true" > ~/.config/buybitcoin.conf

# Redirecting a file's output to a variable
read price <cryptocoins

# File descriptor copying
ping 127.0.0.1 >results 2>&1

# Appending file redirection
echo "balance=0" >> ~/.config/buybitcoin.conf

# Redirecting both stdin and sterr
ping 127.0.0.1 &>results

# Here documents
cat <<.
Hello.
This reads until the delimer appears on a line alone.
In this case it is a period.
.

# Here strings
cat <<<"Hello world.
The land is Bash seems ever-growing!"

# Closing file descriptors >&- defaults to closing
# stout, and <&- defaults to closing stin
exec 3>&1 >mylog; echo moo; exec 1>&3 3>&-

# Moving file descriptors, second descriptor is copied
# to the first, and then the second is closed
exec 3>&1- >mylog; echo moo; exec >&3-

# Reading and writing with a file descriptor
exec 5<>/dev/tcp/ifconfig.me/80
echo "GET /ip HTTP/1.1
Host: ifconfig.m
" >&5
cat <&5
```

While redirections can be placed anywhere within a command, it is best to place them at
the end for consistency and to avoid missing them. This rule ought only to be broken in interests
of readability.

## Summary

- The first things bash looks for is the name, and whether it is a function, a built-in
  command, or a program.
- Bash checks the `PATH` environment variable to find programs.
- Blank space is used to delimit command arguments and must be quoted or escaped if it is
  part of the argument.
- There are many ways to use redirection, some more obscure than others. The most common
  ones being `>`, `<`, `>>` and `&>`.

---
layout: post
title: 'Bash Academy, Chapter One Notes'
date: 2021-01-07
categories: linux bash scripting
---

What is [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell))?

- Bash is a program designed to be easy to interact with, in a way such as giving it
  commands and performing certain tasks.
- Bash was designed to be general purpose, rather than to perform any specific task.
- A _shell_ program is one that provides users an ability to interact with and combine the
  output and inputs of other programs.

There are a few different popular shells, such as C shell (csh), Z shell (zsh),
Korn shell (ksh), Bourne shell, Debian's Almquist shell (dash), etc. Bash is currently the
most popular and widely installed shell on Linux systems. **Bash is a _shell program_
designed to listen to commands.** While shells have their similarities, they are not the
same and one must be aware of the language one is writing in.

## Modes

Bash runs in two different modes:

- _Interactive mode_, where it waits for commands and then executes them. Once the command
  has been performed, the shell awaits the next command.
- _Non-interactive mode_, where the shell executes scripts. A script is a series of commands
  which can be executed without requiring user input. Usually, the script is stored in a
  file.

Bash runs in a terminal, which is a _text-based interface_. Once an actual hardware
interface, most terminals are now "emulated". Popular terminal emulators include _rxvt_,
_xterm_, _gnome-terminal_, _konsole_, _iTerm2_, _cmd.exe_, or my favourite _kitty_.

## Inputs and outputs

A process has certain _hooks_ which connect it to the outside world and relay instructions
to the _kernel_. There are called **file descriptors**.

- _Standard Input_, or file descriptor 0, is where most shell processes receive input. By
  default, terminal processes take input from a keyboard.
- _Standard output_, or file descriptor 1, is where most processes send their output. This
  ends up on the display of the terminal emulator, or can be redirected to another
  processes' standard input.
- _Standard error_, or file descriptor 2, is where processes send error or informational
  messages. It is not just for errors, as most bash processes use it for information to be
  shown to the user.

A process can also create new file descriptors as it sees fit. When information flows
between programs rather than simply being displayed on the terminal screen, it is called a
_stream_. This is an example of uni-directional data flow.

**File descriptors are process-specific** and describe the specific "plugs" that processes
can use to communicate with one another.

## Summary

- Bash is a programming language used to give a Linux machine commands and execute
  scripts.
- It runs interactively or non-interactively.
- The three main files descriptors are standard input, standard output and standard error.

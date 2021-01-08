---
layout: post
title: 'Bash Academy, Chapter Two, Part One Notes'
date: 2021-01-07
categories: linux bash scripting
---

The basics of bash commands are important to avoid creating programs that can inflict extensive damage to
unsuspecting users or systems.

## What are bash commands?

The most basic form is _synchronous command execution_. Bash takes a command, executes
it and then awaits another command. A bash command is the smallest unit of code that
bash can independently execute.

```bash
# This command happens so fast it seems instant
echo "This is a synchronous command"

# When starting a program from the command line,
# bash waits for the program to end. This can take
# quite some time.
firefox
```

## How to give bash commands

Bash is typically a line-based language. Most commands are only one line, but sometimes
the syntax indicates that the command is not yet finished. Bash then waits until the end
is reached before the command is executed.

```bash
# Bash has to wait until "fi" before it has enough information
# to execute the following command.
read -p "Your name? " name
if [[ $name = "Erin" ]]; then
  echo "Hi to myself!"
else
  echo "Hello, $name"
fi
```

## Converting a command to a script

It is easy to convert a command such as the one above to a script. The only difference is
adding a _hashbang_ line to the top and saving it to a file `script.bash`.

```bash
#!/usr/bin/env bash
read -p "Your name? " name
if [[ $name = "Erin" ]]; then
  echo "Hi to myself!"
else
  echo "Hello, $name"
fi
```

The program `env` knows how to find bash, so rather than specify the direct path to the
bash executable, this enables the script to be more flexible and not rely on the system to
have bash in the usual location of `/usr/bin/bash`. The final step is to make the script
file executable by changing the file permissions `chmod +x script.bash`. Then it can be
run with `./script.bash` and the effect is the same as the previous interactive example.

## Learning to speak "bash"

Bash is not a strict language. The responsibility for writing good bash code falls solely
on the shoulders of the developer. It requires _discipline_, _consistency_, and the
ability to recognize potential pitfalls. There are several types of commands, and what
follows is a rough outline of various common syntaxes.

### Simple commands

The most common kind of command, it consists of the name of the command to execute, along
with a few optional components: _arguments, environment variables,_ and _file descriptor
redirections._

```bash
# [ variable=value ... ] name [ arguments ... ] [ redirection ... ]
PRICE=40000 echo "The price of Bitcoin is almost $PRICE USD!" > omg.txt
```

Environment variables are either initialized when the user logs in, or they are declared
before executing the command. Arguments consist of "flags", usually two mid-dashes
followed by a word, or one mid-dash followed by the first letter of the word, such as
`--move` or `-m`.

### Pipelines

Pipelines are a simple way to connect the logic of two commands. An example would be
piping the output of `cat` to `less`, in order to read a long file, without it all just
scrolling quickly past in the terminal window.

```bash
cat syslog.txt | less
```

### Lists

Bash also provides a way to execute multiple commands on one line, either one command
after another, or one command conditionally dependant on the result of the previous. This
can be useful for only running something if the previous command failed, such as `rm hello.txt || echo "couldn't remove"`.

```bash
# make a directory, then go into it
mkdir secrets; cd secrets
# create a file, then open it in vim
touch MY_BITCOINS.txt && vim MY_BITCOINS.txt
```

As a side note, many scripting languages use semi-colons to denote the end of a command.
In addition, new lines also break up command logic in a similar way, with some languages
automatically inserting semi-colons when parsing the script, such as JavaScript or Go.

### Compound commands

A compound command features a bit more logic inside, while still functioning as a single
command. An example of this would be an if-else statement. While the block itself has many
sub-commands, the result is a single command that gets executed.

```bash
if [[ $PRICE = 40000 ]]; then
  echo "Wow, Bitcoin is mooning!"
else
  echo "Maybe it's time to buy..."
fi
```

### Functions

One step up from a compound command is a function. It is another way to encapsulate logic
and make it possible to invoke it later on a script. While some languages allow parameters
to be specified within a function's parentheses, bash does not.

```bash
sayHello() {
  echo "Hello world!";
}
# sayHello => "Hello world!"
```

## Summary of Part One

- A Command is a small piece of logic that bash can execute.
- Bash can be given commands in interactive mode or with scripts.
- Bash syntax supports a variety of different types of commands, such as simple commands,
  pipelines, lists, compound commands and functions.

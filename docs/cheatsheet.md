---
layout: default
title: "Cheatsheet"
header_type: base
permalink: /docs/cheatsheet/
---

This page is currently still a work in progress, but will contain some information on some frequently used commands.

## Logging into a server

`ssh`

`sudo`

`chmod` / `chown`

`passwd`

## Basic moving around the command line and moving files

`cd`

`ls -lh` / `ls -la`

`less`

`wc -l`

`head` / `tail`

`pwd`

`rm`

`mkdir`

`mv`

`cp`

`ln -s`

`ctrl` + `c`

`tar` / `gzip` / `gunzip`

## Downloading files

`scp`

`wget`

`rsync`

`curl`

## Seeing processing running and disk usage

`top` / `htop`

`du -sh `/ `df`

## Editing or creating files and manipulating strings

`vi` / `nano` / `touch`

`cat` (`>` or `>>`)

`echo`

`export`

`grep`

`awk`

`sed`

`sort`

`cut`

`find`

## Installing environments and getting help with commands

`conda` (`activate` / `create` / `deactivate` / `install`)

`command --help` / `command --version` / `which command`

`man command`

`time`

## Keeping things running even if you get disconnected from your server

For programs that may take a while, there are several tools that are pre-installed on most Linux systems that we can use to make sure that our program carries on running even if we get disconnected from the server. One of the most frequently used ones is called `tmux` (another common one is `screen`). To activate it, just type in `tmux` and press enter. It should take a second to start up, and then load up with a similar looking command prompt to previously, but with a coloured bar at the bottom of the screen.

To get out of this window again, press `ctrl`+`b` at the same time, let go of the keys completely, and then immediately press `d`. You should see your original command prompt and something like
```bash
[detached (from session 0)]
```

We can actually use tmux to have multiple sessions, so to see a list of the active sessions, use:
```bash
tmux ls
```

We can rename the tmux session that we just created with this:
```bash
tmux rename-session -t 0 metagenome
```
Note that we know it was session 0 because it said that we detached from session 0 when we exited it.

If we want to re-enter this window, we use:

```bash
tmux attach-session -t metagenome
```

Or if we want to go back to the last tmux session that we had open, we can just use:
```bash
tmux a
```

Now, we can run all of our analysis inside this tmux session, and if we get disconnected from the server we simply use the attach-session command above to get back into our analysis.

We need these because things like metagenomic assembly can take weeks to run, even on a server, and it's not realistic to stay connected to the server with no interruptions at all for that long.


## Running the same command on multiple files - a crash course in GNU Parallel

Sometimes in bioinformatics, the number of tasks you have to complete can get VERY large (e.g. when we have thousands of samples). Fortunately, there are several tools that can help us with this. One such tool is [GNU Parallel](https://www.gnu.org/software/parallel/parallel_tutorial.html). This tool can simplify the way in which we approach large tasks, and as the name suggests, it can iterate though many tasks in parallel, i.e. at the same time. 

First, we'll activate the environment that we'll be using:
```bash
conda activate kneaddata-0.12.4
```


We can use a simple command to demonstrate how to use parallel:
```bash
parallel 'echo {}' ::: a b c
```

With the command above, the program contained within the quotation marks `' '` is `echo`. This program is run 3 times, as there are 3 inputs listed after the `:::` characters. What happens if there are multiple lists of inputs? Try the following:
```bash
parallel 'echo {}' ::: a b c ::: 1 2 3
```

Here, we have demonstrated how `parallel` treats multiple inputs. It uses all combinations of one of each from `a b c` and `1 2 3`. But, what if we wanted to use 2 inputs that were sorted in a specific order? This is where the `--link` flag becomes particularly useful. Try the following:
```bash
parallel --link 'echo {}' ::: a b c ::: 1 2 3
```

In this case, the inputs are “linked”, such that only one of each is used. If the lists are different lengths, `parallel` will go back to the beginning of the shortest list and continue to use it until the longest list is completed.
```bash
parallel --link 'echo {}' ::: light dark ::: red blue green
```

Notice how `light` appears a second time (on the third line of the output) to satisfy the length of the second list.

Another useful feature is specifying *which* inputs we give `parallel` are to go *where*. This can be done intuitively by using multiple brackets `{ }` containing numbers corresponding to the list we are interested in.
```bash
parallel --link 'echo {1} {3}; echo {2} {3}' ::: one red ::: two blue ::: fish
```

Finally, a handy feature is that `parallel` accepts files as inputs. This is done slightly differently than before, as we need to use four colon characters `::::` instead of three. Parallel will then read each line of the file and treat its contents as a list. You can also mix this with the three-colon character lists `:::` you are already familiar with. Using the following code, create a test file and use `parallel` to run the `echo` program:
```bash
echo -e "A\nB\nC" > test.txt
parallel --link 'echo {2} {1}' :::: test.txt ::: 1 2 3
```

Take a look inside `test.txt` with the `less` command if you like. Remember that you can use `q` to exit the file again.

And with that, you’re ready to use `parallel` for all of your bioinformatic needs! We will continue to use it throughout this tutorial and show some additional features along the way. There is also a cheat-sheet [here](https://www.gnu.org/software/parallel/parallel_cheat.pdf) for quick reference.


## Authors

**Author:** Robyn Wright<br>
**Modifications by:** NA<br>
**Based on initial versions by:** NA

<img src="/assets/images/MicrobiomeHelperLogo.png" alt="Microbiome Helper logo" style="width: 50%; height: auto;">

---
layout: default
title: "Cheatsheet"
header_type: base
permalink: /docs/cheatsheet/
---

Here are a few other cheatsheets that we've used:
- [Linux training academy](https://www.linuxtrainingacademy.com/linux-commands-cheat-sheet/)
- [Conda cheatsheet](https://kapeli.com/cheat_sheets/Conda.docset/Contents/Resources/Documents/index)
- [One-liners](https://github.com/ECBSU/oneliners)
- [More one-liners](https://github.com/stephenturner/oneliners)
- [`vi` cheatsheet](https://linuxsimply.com/cheat-sheets/vi/)
- [`nano` cheatsheet](https://linuxize.com/cheatsheet/nano/)

### Moving around

| Command | Description |
| --- | --- |
| `pwd` | show where you are |
| `ls -lh` | list files with human-readable sizes |
| `ls -a` | list all files including hidden ones |
| `cd dir_name` | change into a directory |
| `cd ..` / `cd ~` | go up one level/go home |
| `mkdir -p results/qc ` | make directories, including parents |
| `tab` key | autocomplete file/foler names |
| `up` arrow, history | scroll up through last used commands |
| `ctrl`+`c` | stop the command you are running immediately |

### Managing/moving files

| Command | Description |
| --- | --- |
| `cp a.txt b.txt` | make a copy of `a.txt` to `b.txt` |
| `mv a.txt data/` | move `a.txt` to the `data` folder |
| `mv a.txt c.txt` | rename `a.txt` to `c.txt` |
| `rm file` / `rm -r dir_name` | delete a file/folder - careful, this is permanent! |
| `ln -s /path/to/raw_reads .` | make a shortcut to files instead of copying |
| `du -sh dir_name` | size of folder |
| `df -h` | check free disk space |

### Looking at files

| Command | Description |
| --- | --- |
| `head -n 8 file` | first 8 lines of file (or `tail` for last) |
| `cat file` | view the contents of a file | 
| `less -S file.tsv` | scroll through file, `-S` means without wrapping |
| `wc -l file` | count the number of lines in a file |
| `cut -f 1,3 table.tsv` | get columns 1 and 3 from a tab separated file |

### Searching, sorting or counting

| Command | Description |
| --- | --- |
| `grep "Bacteroides" taxa.tsv` | lines containing a pattern/string |
| `grep -c "^>" seqs.fasta` | count occurrences of `>` as first character in a line |
| `sort file | uniq -c | sort -nr` | count and sort unique values in a file |
| `awk -F'\t' '$3 > 1000' table.tsv` | get rows where column 3 in a tab separated file is > 1000 |
| `sed 's/old/new/g' file` | find and replace all instances of `old` with `new` in `file` |

### Pipes, redirection, and combining files

| Command | Description |
| --- | --- |
| `cmd1 | cmd2` | send output of one command into the next using `|` |
| `cmd > out.txt ` | save the output from `cmd` to `out.txt` (writes over anything in `out.txt`) |
| `cmd >> out.txt ` | append the output from `cmd` to `out.txt` |
| `cmd 2> errors.log` | save error messages from `cmd` to `errors.log` | 
| `cat file1 file2 > file3` | print out `file1` and `file2` to `file3` (i.e. combine files 1 & 2) |

### Compression and archives

| Command | Description |
| --- | --- |
| `gzip file` / `gunzip file.gz` | compress/decompress `file` |
| `tar -czvf folder.tar.gz folder` | compress `folder` and name it `folder.tar.gz` |
| `tar -xvf folder.tar.gz` | decompress `folder.tar.gz` |
| `zless`, `zcat`, `zgrep` | `less`, `cat`, `grep` on `.gz` files |

### Editing or creating files

| Command | Description |
| --- | --- |
| `touch file` | create an empty file called `file` |
| `vi file.txt` | create/open `file.txt` with `vi` text editor |
| `nano file.txt` | create/open `file.txt` with `nano` text editor |

**`vi` commands:**

| Command | Description |
| --- | --- |
| `i` | enter insert mode |
| `esc` | exit insert mode |
| `:x!`+`enter` | save and exit file |
| `:q!`+`enter` | exit file without saving changes |

While in insert mode, you can use the arrow keys to move your cursor around and your other keys to make changes as normal. There are lots of other shortcuts that you can see in the `vi` cheatsheet we have linked above!

**`nano` commands:**

| Command | Description |
| --- | --- |
| `i` | enter insert mode |
| `ctrl`+`S` | save file (no prompt) |
| `ctrl`+`X` then `Y` | save and exit file |
| `ctrl`+`X` then `N` | exit file without saving changes

### Using a server

| Command | Description |
| --- | --- |
| `ssh user@server` | log into `server` as `user` |
| `passwd` | change password on server |
| `scp file user@server:path/` | copy `file` to `path` on `server`, e.g. `scp test.txt user@amazon.com:/home/user/microbiome_files/` |
| `rsync -P file user@server:path/` | like `scp` but resumable if the transfer gets interrupted |
| `wget URL -o new_file.txt` | download `file.txt` and save it as `new_file.txt` | 
| `curl -L -O URL` | alternative to `wget` that follows redirects (`-L`) & keeps the filename from the URL (`-O`) |
| `top` / `htop` | see what's running (see below) |
| `chmod +x script.sh` | make script runnable |
| `sudo` | add before command to run with admin security privileges (if available) |
| `sudo chown -R user folder` | change the owner of all files in `folder` to `user` |
| `md5sum file` | verify MD5 checksums* |

\* it is often useful to check that your files transferred intact. Each file has a unique file has a unique MD5 hash (a 32-character hexadecimal string), which is like a fingerprint (e.g. `2a417713736e980f7400e6c09560ab0f`). You can run this on your files before transferring and then again at their destination to ensure that they are the same. For example, `md5sum file.txt > file.txt.md5` run on `file.txt` on your laptop should give you an identical to when you run it on `file.txt` after you've transferred it to your server.

### Conda environments

| Command | Description |
| --- | --- |
| `conda info --envs` | get list of all available `conda` environments |
| `conda create --name env_name` | create a `conda` environment called `env_name` |
| `conda activate env_name` | activate the `env_name` environment |
| `conda deactivate` | deactivate the current environment |
| `conda remove -n env_name --all` | delete the `env_name` environment |
| `conda install package_name` | install `package_name` in the current environment |
| `conda remove package_name` | remove `package_name` from the current environment |

### Sequence file tricks

| Command | Description |
| --- | --- |
| `echo $(( $(zcat sample.fastq.gz | wc -l) / 4 ))` | get number of reads in a gzipped fastq file |
| `echo $(( $(zcat sample.fasta.gz | wc -l) / 2 ))` | get number of reads in a gzipped fasta file |
| `zcat sample.fastq.gz | head -n 4` | look at the first read (& quality information) in a gzipped fastq file |

### GNU Parallel

| Command | Description |
| --- | --- |
| `parallel 'command {1} {2}' ::: input_1 input_2 ::: input_a input_b` | run `command` on all combinations of input_1/2 and input_a/b |
| `parallel 'command' :::: file.txt` | run `command` on all lines in `file.txt` (especially useful for lists that are too long!) |
| `--link` | links inputs together so one is taken from each input sequentially |
| `--dry-run` | prints the commands that will be run to terminal instead of running them |
| `--eta` | displays estimated time of completion for all inputs |
| `--progress` | displays how many inputs are running, have been run, and still to run along with average time per task |

| Replacement string | Value if input is `mydir/mysubdir/myfile.myext` |
| --- | --- |
| `{}` | `mydir/mysubdir/myfile.myext` |
| `{.}` | `mydir/mysubdir/myfile` |
| `{/}`, `{//}`, `{/.}` | `myfile.myext`, `mydir/mysubdir`, `myfile` |
| `{2}` | Value from the second input source |
| `{2.}`, `{2/}`, `{2//}`, `{2/.}` | Combination of `{2}` and `{.}`, `{/}`, `{//}`, `{/.}` |

### Getting help and other useful things

| Command | Description |
| --- | --- |
| `command --help` | give a summary of the options available & how to run `command` |
| `man command` | get the full manual for `command` (`q` to quit) |
| `time command` | measure how long `command` takes to run |

### Keeping things running even if you get disconnected from your server

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


### Running the same command on multiple files - a crash course in GNU Parallel

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

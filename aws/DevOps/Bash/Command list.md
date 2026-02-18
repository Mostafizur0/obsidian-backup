SSH uses a client-server architecture, where the client initiates a connection to the server and requests a secure communication channel.
https://www.freecodecamp.org/news/ssh-meaning-in-linux/
```shell
ssh -i <secret-key-file-path> <username>:<ip-address>
```

| flags | description            |
| ----- | ---------------------- |
| -i    | identity file location |
|       |                        |

```shell
ls -ltr
```
https://www.digitalocean.com/community/tutorials/ls-command-in-linux-unix

| flags | description                                                                                                                                              |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -l    | list the permissions of the files and directories as well as other attributes such as folder names, file and directory sizes, and modified date and time |
| -t    | sort by time, newest first                                                                                                                               |
| -r    | To list files in reverse order                                                                                                                           |
| -h    | human readable format                                                                                                                                    |
| -a    | view hidden files                                                                                                                                        |
https://medium.com/@kikuchidaisuke.zr/linux-free-command-explained-a6f2e2d311e7
```shell
free -g
```
![[Pasted image 20260214193430.png]]
<span style="color: #1DB954;">Shared</span>: It refers to the memory used for tmpfs, POSIX shared memory and etc.
<span style="color: #1DB954;">Buffer</span>: cache for file system metadata in block level fully managed by OS
<span style="color: #1DB954;">Cache</span>: Cache, on the other hand, is the cache for an actual file content not metadata, and it will be cached as an entire file content, not like spread block data.  More specifically this cache is called “OS Page cache”. They are both handled by OS
<span style="color: #1DB954;">Available</span>: free + Buffer + Cache (as buffer and cache are reclaimable)
<span style="color: #1DB954;">Swap</span>: It is the size of disk used as memory ss

https://www.youtube.com/watch?v=rz1JZA-l9nc
```shell
nproc
```
to count the number of available processing units available to the current processes.

```shell
df -h
```
==df (disk free)== displays the disk space usage of all mounted filesystems in a "human-readable" format(e.g., GB or MB instead of bytes). This command retrieves the information from `/proc/mounts` or `/etc/mtab`

https://www.howtogeek.com/668986/how-to-use-the-linux-top-command-and-understand-its-output/
```shell
top
```
mange and view cpu, memory, process details. combine above 3 commands

https://askubuntu.com/questions/141928/what-is-the-difference-between-bin-sh-and-bin-bash

https://www.computerhope.com/unix/uchmod.htm
```shell
chmod options permissions file name
chmod u=rwx,g=rx,o=r myfile
chmod 754 myfile
```
**chmod** stands for "change mode." It restricts the way a file can be accessed. _permissions_ defines the permissions for the owner of the file (the "user"), members of the group who owns the file (the "group"), and anyone else ("others"). There are two ways to represent these permissions: with symbols ([alphanumeric](https://www.computerhope.com/jargon/a/alphanum.htm) [characters](https://www.computerhope.com/jargon/c/charact.htm)), or with [octal](https://www.computerhope.com/jargon/o/octal.htm) numbers (the digits **0** through **7**).
**chmod** never changes the permissions of [symbolic links](https://www.computerhope.com/jargon/s/symblink.htm); the **chmod** system call cannot change their permissions. However, this is not a problem since the permissions of symbolic links are never used. However, for each symbolic link listed on the [command line](https://www.computerhope.com/jargon/c/commandi.htm), **chmod** changes the permissions of the pointed-to file. In contrast, **chmod** ignores symbolic links encountered during [recursive](https://www.computerhope.com/jargon/r/recursive.htm) directory traversals.

==The restricted deletion flag or sticky bit== is a single bit, whose interpretation depends on the file type. For directories, it prevents unprivileged users from removing or renaming a file in the directory unless they own the file or the directory; this is called the restricted deletion flag for the directory, and is commonly found on world-writable directories like **/tmp**. For regular files on some older systems, the bit saves the program's text image on the swap device so it loads quicker when run; this is called the sticky bit.

```shell
history
```
The `history` command in Linux displays a numbered list of previously executed commands. The command history for a user is typically stored in the hidden file `~/.bash_history`.

https://www.scaler.com/topics/ps-command-in-linux/
```shell
ps -ef
```
Viewing processes for all users

```shell
ps aux  // will work
ps -aux // will not work
```
Viewing detailed process information with cpu, memory etc
https://stackoverflow.com/questions/15102576/ps-aux-works-but-ps-aux-doesnt

```shell
grep [options] pattern [file...]
grep 'Linux' welcome.txt
```
The **`grep` command** is a powerful command-line utility used to search for specific text **patterns** (regular expressions) within files and print the lines that match those patterns. Its name is an acronym for "global regular expression print".
https://stackoverflow.com/questions/7727640/what-are-the-differences-among-grep-awk-sed
<span style="color: #1DB954;">Grep</span> is useful if you want to quickly search for lines that match in a file. It can also return some other simple information like matching line numbers, match count, and file name lists.

<span style="color: #1DB954;">Awk</span> is an entire programming language built around reading CSV-style files, processing the records, and optionally printing out a result data set. It can do many things but it is not the easiest tool to use for simple tasks.
https://www.theunixschool.com/2012/09/grep-vs-awk-examples-for-pattern-search.html

<span style="color: #1DB954;">Sed</span> is useful when you want to make changes to a file based on regular expressions. It allows you to easily match parts of lines, make modifications, and print out results. It's less expressive than awk but that lends it to somewhat easier use for simple tasks. It has many more complicated operators you can use (I think it's even turing complete), but in general you won't use those features.

https://unix.stackexchange.com/questions/63658/redirecting-the-content-of-a-file-to-the-command-echo
what will be the result of `date | echo "this"` ?
answer: this
date command send output to the stdin but 
`echo` <span style="color: #1DB954;">doesn't read its standard input</span>. All it does is <span style="color: #00BFFF;">write to standard output</span> its _arguments_ separated by a space character and terminated by a newline character (and with some `echo` implementations with some escape sequences in them expanded and/or arguments starting with `-` possibly treated as options).
If you want `echo` to display the content of a file, you have to pass that content as an argument to `echo`.

```shell
set -x # debug mode
set -e # exit when error; only catch error on the last command of a piped statement
set -o pipefail # catches any pipe (|) failure
```

<span style="color: #1DB954;">wget</span> downloads content locally, <span style="color: #00BFFF;">curl</span> is used for request-response scenarios.
https://daniel.haxx.se/docs/curl-vs-wget.html
https://unix.stackexchange.com/questions/47434/what-is-the-difference-between-curl-and-wget

https://serveracademy.com/blog/linux-find-command/
https://www.networkworld.com/article/2501224/how-to-find-files-on-linux.html
```shell
find ~ -type f,l -name "notebook*"
find [path] [options] [expression]
find ./gfg -name "sample.txt"
find ~/Documents -ls # find everything in this dir
find ~ -type f,l -maxdepth 1 -name "notebook*"
find ~/Documents/ -name "*txt" -exec grep -Hi penguin {} \;
find ~ -type f -empty # find empty files
find /var/log -iname "*~" -o -iname "*log*" -mtime -7 -ls
find / -type d -name 'img' -ipath "*public_html/example.com*" 2>/dev/null # search for the path
```

https://unix.stackexchange.com/questions/306111/what-is-the-difference-between-the-bash-operators-vs-vs-vs
`((...))` is the shell's **arithmetic** construct.
`(...)` is a _grouping_ construct that executes the contained commands in a subshell
`[...]` is the "legacy" conditional construct.
`[[...]]` does everything that `[...]` does. The difference is that word splitting and glob expansion are not performed for variables inside `[[...]]` so quoting the variables is not so crucial. Additionally, `[[` can do _pattern matching_ with the `==` operator and _regular expression matching_ with the `=~` operator.

The reason `[[ 10 > 9 ]]` gives you an unexpected result is that the `>` operator inside `[[...]]` is for _string comparison_ and the string "10" is "less than" the string "9".
https://dev.to/rpalo/bash-brackets-quick-reference-4eh6

https://www.baeldung.com/linux/trap-command
```shell
cat trap_preserving.sh
!/bin/bash

 Save existing trap for SIGINT
trap_saved_SIGINT=$(trap -p SIGINT)

 Set up trap to catch SIGINT (Ctrl+C) and execute cleanup function
trap "echo Trapped." SIGINT

sleep 10

eval "$trap_saved_SIGINT"
```

```shell
trap "" SIGINT SIGABRT # disable SIGINT SIGABRT signals
trap - SIGINT SIGABRT # set SIGINT SIGABRT to default signal action
trap -p # it prints the piece of code we wrote associated with each signal.
trap -p SIGINT # the commands/piece of code associated with that signal is displayed.
trap -l # list of all available signals that can be trapped
```
https://www.baeldung.com/linux/trap-command

```shell
grade=85
if [ "$grade" -ge 90 ]; then
  echo "Grade: A"
elif [ "$grade" -ge 80 ]; then
  echo "Grade: B"
elif [ "$grade" -ge 70 ]; then
  echo "Grade: C"
else
  echo "Grade: F"
fi
```

```bash
for i in 1 2 3 4 5; do 
	if [ $i -eq 3 ]; then 
		break 
	fi 
	echo $i 
done 
# Outputs: 1 2
```

```bash
for ((i=0; i<5; i++)); do
    echo "Count: $i"
done
Count: 0
Count: 1
Count: 2
Count: 3
Count: 4
```

```bash
COUNT=0
while [ $COUNT -lt 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done
```

```bash
COUNT=0
until [ $COUNT -ge 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done
```

```bash
case $1 in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    *)
        echo "Usage: $0 {start|stop}"
        ;;
esac
```
https://freeacademy.ai/lessons/script-conditionals

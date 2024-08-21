---
layout: post
title: overthewire bandit room guide
---
I’ve been doing OverTheWire challenges recently, just for fun. Here’s a solution guide I’ve cooked up for the Bandit room. Some of these answers might not be the most efficient, so feel free to contact me if you have any suggestions. 

This guide isn’t finished yet. I’ll be updating it as time goes on.
# Pregame (Level 0)
## Level Goal
The goal of this level is for you to log into the game using SSH. The host to which you need to connect is **bandit.labs.overthewire.org**, on port 2220. The username is **bandit0** and the password is **bandit0**. Once logged in, go to the [Level 1](https://overthewire.org/wargames/bandit/bandit1.html) page to find out how to beat Level 1.
## Solution
Run `ssh bandit0@bandit.labs.overthewire.org -p 2220`. Provide the password when prompted.
# Level 0 (0 → 1)
## Level Goal
The password for the next level is stored in a file called **readme** located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH (on port 2220) to log into that level and continue the game.
## Solution
We can use `ls` to list the files within the current directory. 
```
bandit0@bandit:~$ ls
readme
```
Ah, there’s the file we’re looking for! Use `cat` to retrieve the contents (our password) of the file:
```
bandit0@bandit:/home$ cat readme
ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If
```
# Level 1
## Level Goal
The password for the next level is stored in a file called **-** located in the home directory.
## Solution
You may notice that `cat -` does not produce an output. This is because using `-` as an argument refers to STDIN/STDOUT. To open this file, we must provide its full path:
```
bandit1@bandit:/home$ cat ./-
263JGJPfgU6LtdEvgfWU1XP5yac29mFx
```
# Level 2
## Level Goal
The password for the next level is stored in a file called **spaces in this filename** located in the home directory.
## Solution
Passing the file directly into `cat` does not work here, as it registers the name as several arguments. We must put the filename into quotations:
```
bandit2@bandit:/home$ cat "spaces in this filename"
MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx
```
# Level 3
## Level Goal
The password for the next level is stored in a hidden file in the **inhere** directory.
## Solution
We should first change our working directory to **inhere**. While it’s possible to solve the problem (or any problem, for that matter) from the home directory, it’s nice to be able to switch our working directory. We use the `cd` command for this:
```
bandit3@bandit:~$ cd inhere
bandit3@bandit:~/inhere$ 
```
Files that begin with a period are hidden from the regular `ls` command. We need to pass the argument `-a` to list all hidden and non-hidden files within the directory. 
```
bandit3@bandit:~/inhere$ ls -a
.  ..  ...Hiding-From-You
```
There’s the bastard.
```
bandit3@bandit:~/inhere$ cat ...Hiding-From-You
2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ
```
# Level 4
## Level Goal
The password for the next level is stored in the only human-readable file in the **inhere** directory. Tip: if your terminal is messed up, try the “reset” command.
## Solution
Let’s look at what we’re working with first.
```
bandit4@bandit:~$ cd inhere
bandit4@bandit:~/inhere$ ls -a
.  ..  -file00  -file01  -file02  -file03  -file04  -file05  -file06  -file07  -file08  -file09
```
We know that the password is stored in one of these files, which is “human-readable”. There are now two solutions:
1. Since there are only 10 files, we could `cat` all of the files and read the output. 
2. A better solution. See below.
The `file` command returns the type of data of the file. We can check all of the files by running `file ./*`. Note that we must include the full path, as each file starts with a dash.
```
bandit4@bandit:~/inhere$ file ./*
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
```
There we are.
```
bandit4@bandit:~/inhere$ cat ./-file07
4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw
```
# Level 5
## Level Goal
The password for the next level is stored in a file somewhere under the **inhere** directory and has all of the following properties:
- human-readable
- 1033 bytes in size
- not executable
## Solution
Let’s see what we’re working with first. 
```
bandit5@bandit:~$ cd inhere
bandit5@bandit:~/inhere$ ls
maybehere00  maybehere03  maybehere06  maybehere09  maybehere12  maybehere15  maybehere18
maybehere01  maybehere04  maybehere07  maybehere10  maybehere13  maybehere16  maybehere19
maybehere02  maybehere05  maybehere08  maybehere11  maybehere14  maybehere17
```
They seem to all be directories. We can check all the files in the directories with the `-R` flag, searching everything recursively:
```
bandit5@bandit:~/inhere$ ls -R
.:
maybehere00  maybehere03  maybehere06  maybehere09  maybehere12  maybehere15  maybehere18
maybehere01  maybehere04  maybehere07  maybehere10  maybehere13  maybehere16  maybehere19
maybehere02  maybehere05  maybehere08  maybehere11  maybehere14  maybehere17

./maybehere00:
-file1  -file2  -file3  spaces file1  spaces file2  spaces file3

...about 60 more lines...
```
That’s a lot of files! We can’t resort to using naive solutions anymore. Let’s take a look at the file properties.
1. The file is human readable.
We have experience with finding human-readable files. We simply run the `file` command, and look for files in ASCII format. However, we need to modify our command to look for *all* files in `inhere`. We run `file*/{.,}*`. `.` catches hidden files, and `,` catches everything else.
```
bandit5@bandit:~/inhere$ file */{.,}*
maybehere00/.file1:       ASCII text, with very long lines (550)
maybehere00/.file2:       ASCII text, with very long lines (7835)
maybehere00/.file3:       data
maybehere01/.file1:       Clarion Developer (v2 and above) memo data
maybehere01/.file2:       ASCII text, with very long lines (3069)
maybehere01/.file3:       data
maybehere02/.file1:       ASCII text, with very long lines (6350)
maybehere02/.file2:       ASCII text, with very long lines (2576)
maybehere02/.file3:       data
maybehere03/.file1:       ASCII text, with very long lines (9768)
...about 300 more lines...
```
That’s a lot of data! And a lot of them are human-readable. Not very helpful, but some of them aren’t human readable. Let’s keep that in mind for now. Let’s take a look at the second property.
2. The file is 1033 bytes in size.
This clue is rather specific, and seems to be rather helpful. All we have to do is check the size of every file, and find the one that is 1033 bytes. We can use the `du` command to achieve this, passing in the `-b` flag for bytes, and `-a` for all files. However, there are a lot of files, and it would take a long time to look through all the filesizes. We can use `grep` to give us only the files with a size of 1033 bytes. 
Our final command looks like `du -b -a | grep 1033`. The pipe (`|`) sends the command to `grep`.
```
bandit5@bandit:~$ du -b -a | grep 1033
1033	./inhere/maybehere07/.file2
bandit5@bandit:~$ cat ./inhere/maybehere07/.file2
HWasnPhtq9AVKe0dmk45nxy20cvUa6EG
bandit5@bandit:~$
```
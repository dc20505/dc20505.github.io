---
title: "[OTW] Bandit Wargame Walkthrough"
permalink: /writeups/otw/bandit/
excerpt: "Walkthrough for the Bandit wargame from Over the Wire."
---

---
{% include toc icon="cog" title="Bandit Wargame" %}
[Bandit](http://overthewire.org/wargames/bandit/) is an online wargame is an online game offered by  [Over the Wire](http://overthewire.org) that allows users to explore the Linux command-line interface (CLI) through progressive challenges. 
{: .text-justify}

This writeup is not all-inclusive - it is intended as a guide to assist users where they may be stuck, but ultimately it is the user's responsibility to actually seek out information about what the commands can do and how, in more advanced techniques, they can be chained. Short descriptions will be provided often, but you can use the command `man` (short for manual) followed by the command you are investigating, or append `--help` for more information.
{: .text-justify}


**Note:** The [official](http://overthewire.org/wargames/bandit/) website gives additional details on the goal of each challenges and some helpful material to read, so it is recommended to have it available as you progress through these challenges.
{: .notice--info}

## [Bandit 00](https://overthewire.org/wargames/bandit/bandit0.html)

The bandit games are accessed by SSHing into another system. If you aren't yet familiar with what SSH is, you should research how it until you understand that it allows us to remotely log in to a system as a user, complete with the intended restrictions and access capabilities that were placed upon that user. Once logged in, we are able to perform any actions via the command line just as if we were pyhsically at the system with a keyboard.

### Level Goal

The goal of this level is for you to log into the game using SSH. The host to which you need to connect is **bandit.labs.overthewire.org**, on **port 2220**. The username is **bandit0** and the password is **bandit0**. Once logged in, go to the Level 1 page to find out how to beat Level 1.

The password for the next level is stored in a file called **readme** located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, you may use `exit` to end your current session and then use SSH (on port 2220) to log into the next level and continue the game. The SSH code snippet will only be shown for bandit0 - all you need to do is iterate the user to bandit1 and so on for future levels.

### Commands you may need to solve this level

`ssh, ls, cd, cat, file, du, find`

```bash
$ ssh bandit0@bandit.labs.overthewire.org -p 2220
```

A basic function within the command line is to know where you are working and what you have access to. The `pwd` command prints our current working directory while the list command, `ls`, provides information about files in an alphabetic order. The addition of the flags `a` and `l` option flags command it to include all hidden files and be listed in the long format, respectively.
```bash
bandit0@bandit:~$ pwd
/home/bandit0

bandit0@bandit:~$ ls -la
total 24
drwxr-xr-x  2 root    root    4096 May  7  2020 .
drwxr-xr-x 41 root    root    4096 May  7  2020 ..
-rw-r--r--  1 root    root     220 May 15  2017 .bash_logout
-rw-r--r--  1 root    root    3526 May 15  2017 .bashrc
-rw-r--r--  1 root    root     675 May 15  2017 .profile
-rw-r-----  1 bandit1 bandit0   33 May  7  2020 readme
```
The only file that stands out is `readme`.
```bash
bandit0@bandit:~$ cat readme
boJ9jbbUNNfktd78OOpsqOltutMc3MY1
```

**Explanation:** We are able to view the contents of the file **readme** with the command `cat`, which simply concatenates (joins) text and prints it to our output. 
{: .notice--success}






## [Bandit 01](https://overthewire.org/wargames/bandit/bandit1.html)

### Level Goal

The password for the next level is stored in a file called **-** located in the `home` directory

### Commands you may need to solve this level

`ssh, ls, cd, cat, file, du, find`

We are able to see file **-** in the home directory, but if we try to `cat` it then we have an odd situation where the CLI will just print anything we put into the standard input, `stdin`.

```bash
bandit1@bandit:~$ cat -
test "<<typed>>"
test "<<returned>>"

bandit1@bandit:~$ cat ./-
CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9
```

**Explanation:** After we learn about how `stdin` works, we discover that to read **-** as a file rather than `stdin`, we must specify the full path. Appending `./` before the file allows us to read it. This `./` format just means to execute the command from our current location, but we also could have specified it verbosely as `cat /home/bandit1/-` for the same result.
{: .notice--success}






## [Bandit 02](https://overthewire.org/wargames/bandit/bandit2.html)

### Level Goal

The password for the next level is stored in a file called **spaces in this filename** located in the `home` directory.

### Commands you may need to solve this level

`ls, cd, cat, file, du, find`

```bash
bandit2@bandit:~$ ls
spaces in this filename
bandit2@bandit:~$ cat "spaces in this filename" 
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
```

**Explanation:** Spaces often indicate separation between different commands or flags in linux. To tell the system we mean this specific file with spaces in it, we enclose it in quotation marks. It's also worth noting that you can use `tab` on your keyboard to have the system autcomplete a unique file name - very helpful for long file names.
{: .notice--success}





## [Bandit 03](https://overthewire.org/wargames/bandit/bandit3.html)

### Level Goal

The password for the next level is stored in a hidden file in the `inhere` directory.

### Commands you may need to solve this level

`ls, cd, cat, file, du, find`

Our initial evaluation of the system shows us that there is a folder named `inhere`. We can use `cd` to change directories and navigate into the folder. Note that our CLI now indicates that we are working within the `inhere` folder.

```bash
bandit3@bandit:~$ ls
inhere
bandit3@bandit:~$ cd inhere
bandit3@bandit:~/inhere$ ls

```
Initially no files are returned with the use of `ls`, but if we again add the flags `-la` we are prestned with additional information.

```bash
bandit3@bandit:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 May  7  2020 .
drwxr-xr-x 3 root    root    4096 May  7  2020 ..
-rw-r----- 1 bandit4 bandit3   33 May  7  2020 .hidden
bandit3@bandit:~/inhere$ cat .hidden
pIwrPrtPN36QITSp3EQaw936yaFoFgAB

```

**Explanation:** Systems hide many files from basic users as they are typically not relevant to them, such as configuration and settings files. A period `.` in front of a file hides it from both Graphic User Interface (GUI) inclusion and as we saw here, the command line. After we tell the system to produce **all** files with the `-a` flag we are able to simply cat the file we are now aware is present.
{: .notice--success}





## [Bandit 04](https://overthewire.org/wargames/bandit/bandit4.html)

### Level Goal

The password for the next level is stored in the only human-readable file in the `inhere` directory. Tip: if your terminal is messed up, try the “reset” command.

### Commands you may need to solve this level

`ls, cd, cat, file, du, find`

We do our initial listing but quickly find no way to discriminate between the available files.

```bash
bandit4@bandit:~$ cd inhere
bandit4@bandit:~/inhere$ ls -la
total 48
drwxr-xr-x 2 root    root    4096 May  7  2020 .
drwxr-xr-x 3 root    root    4096 May  7  2020 ..
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file00
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file01
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file02
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file03
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file04
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file05
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file06
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file07
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file08
-rw-r----- 1 bandit5 bandit4   33 May  7  2020 -file09

```
We could try `cat` on each file, but it's much more efficient to use the power of a computer to sort things out. Here, we can use a different command, `file ./*`, to discover human-readable text.
```bash
bandit4@bandit:~/inhere$ file ./-file0*
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
bandit4@bandit:~/inhere$ cat ./-file07
koReBOKuIDDepwhWk7jZC0RTdopnAYKh

```

**Explanation:** The `file` command attempts to classify files and returns data based upon which of it's tests succeed first. We tell it to look within our current directory with `./` and issue a wildcard '*' to search everything. We also could have used `file ./-file0*`, which would only search the files that begin with `file0` in the current directory.
{: .notice--success}





## [Bandit 05](https://overthewire.org/wargames/bandit/bandit5.html)

### Level Goal

The password for the next level is stored in a file somewhere under the `inhere` directory and has all of the following properties:
- human-readable
- 1033 bytes in size
- not executable


### Commands you may need to solve this level

`ls, cd, cat, file, du, find`

In our initial listing we see that there are multiple files that give us no clues as to where the desired file is.

```bash
bandit5@bandit:~/inhere$ ls -la
total 88
drwxr-x--- 22 root bandit5 4096 May  7  2020 .
drwxr-xr-x  3 root root    4096 May  7  2020 ..
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere00
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere01
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere02
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere03
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere04
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere05
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere06
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere07
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere08
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere09
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere10
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere11
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere12
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere13
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere14
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere15
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere16
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere17
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere18
drwxr-x---  2 root bandit5 4096 May  7  2020 maybehere19

```
If we research the `find` command further via it's manual, we discover that we can toggle different option flags to isolate a specific resource.

```bash
bandit5@bandit:~/inhere$ find ./ -type f -readable ! -executable -size 1033c
./maybehere07/.file2
bandit5@bandit:~/inhere$ cat ./maybehere07/.file2
DXjZPULLxYr17uwoI01bNLQbtFemEgo7
```
**Explanation:** The addition of flags allow us to specify paramaters to identify a specific file. `-type f` tells the system we want a regular file, `readable` searches for human-readable files, `! executable` is superfluous but tells the system to return files that are NOT executable, and finallt `-size 1033c` returns files that size in bytes.  
{: .notice--success}





## [Bandit 06](https://overthewire.org/wargames/bandit/bandit6.html)

### Level Goal

The password for the next level is stored **somewhere on the server** and has all of the following properties:
- owned by user bandit7
- owned by group bandit6
- 33 bytes in size


### Commands you may need to solve this level

`ls, cd, cat, file, du, find, grep`

This challenge requires some deeper research than realizing "I need to do x, and then y." After some experimenting you might come to a formula such as:

```bash
bandit6@bandit:~$ find / -type f -size 33c -group bandit6 -user bandit7 2>&1 | grep -v "Permission denied"
find: ‘/proc/32731/task/32731/fdinfo/6’: No such file or directory
find: ‘/proc/32731/fdinfo/5’: No such file or directory
/var/lib/dpkg/info/bandit7.password

bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs

```

**Explanation:** Theres a bit to unpack here, and we'll ignore aspects already covered in previous exercises. The `-group` and `-user` flags should be self explanatory, but there are a few other components that are noteworthy. `2&>1` tells the system to print anything that was sent to the standard error, `stderr`/2 to the standard output, `stdout`/1/our terminal. Following that we see a new item, the pipe `|`. This causes the system to take the output generated from the command(s) before it and process it through new commands. Here, we use `grep` to find patterns in a file, namely text. We also have the flag `-v` which tells it to find the opposite of what we type - since we searched for "Permission denied," it only returned non-matching lines. A few files were returned, but the one we are looking for is obvious.
{: .notice--success}






## [Bandit 07](https://overthewire.org/wargames/bandit/bandit7.html)

### Level Goal

The password for the next level is stored in the file `data.txt` next to the word **millionth**.

### Commands you may need to solve this level

`grep, sort, uniq, strings, base64, tr, tar, gzip, bzip2, xxd`

```bash
bandit7@bandit:~$ find / -name "data.txt" -exec grep -H 'millionth' {} \; 2>&1 | grep -v "Permission denied"
/home/bandit7/data.txt:millionth	cvX2JJa4CFALtqS87jk27qwqGhBM9plV

```

**Explanation:** This exercise is similar to the last one, but we use some additional options to search file contents. The `-exec` argument is used to run a command, `grep -H millionth`, which is searching within the file for the text **millionth** and when it finds it will print out the file name as well. We then again sort out all the files returned with permission issues.
{: .notice--success}





## [Bandit 08](https://overthewire.org/wargames/bandit/bandit8.html)

### Level Goal

The password for the next level is stored in the file `data.txt` and is the only line of text that occurs only once.

### Commands you may need to solve this level

`grep, sort, uniq, strings, base64, tr, tar, gzip, bzip2, xxd`

```bash
bandit8@bandit:~$ sort data.txt | uniq -c | grep "1 "
      1 UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR

```

**Explanation:** The `sort` command will privide an ordered concatenation of the file contents. This output is piped into `uniq -c` who searches for uniques strings and counts each of them. This output is piped into `grep` as we are only looking for the output that occurs once. 
{: .notice--success}






## [Bandit 09](https://overthewire.org/wargames/bandit/bandit9.html)

### Level Goal

The password for the next level is stored in the file `data.txt` in one of the few human-readable strings, preceded by several ‘=’ characters.

### Commands you may need to solve this level

`grep, sort, uniq, strings, base64, tr, tar, gzip, bzip2, xxd`

```bash
bandit9@bandit:~$ strings data.txt | grep "^=="
========== password
========== isa
========== truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk

```

**Explanation:** The `strings` command prints character sequences that are at least four characters long. This output is then piped into grep, with the argument `^==`, where `^` indicates empty-space and `==` are the characters we are searching for at the start of a string.
{: .notice--success}





## [Bandit 10](https://overthewire.org/wargames/bandit/bandit10.html)

### Level Goal

The password for the next level is stored in the file `data.txt`, which contains **base64** encoded data.

### Commands you may need to solve this level

`grep, sort, uniq, strings, base64, tr, tar, gzip, bzip2, xxd`

```bash
bandit10@bandit:~$ ls
data.txt
bandit10@bandit:~$ cat data.txt 
VGhlIHBhc3N3b3JkIGlzIElGdWt3S0dzRlc4TU9xM0lSRnFyeEUxaHhUTkViVVBSCg==
bandit10@bandit:~$ cat data.txt | base64 -d
The password is IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR

```

**Explanation:** Base64 is a standard form of encoding. Here we print the contents of the file and pipe it through a program, `base64`, that reverses the endoding via the `-d` flag.
{: .notice--success}
Note that encoding and decoding are NOT the same as encryption and decryption. Encoding a file is NOT a secure way to hide it's contents.
{: .notice--warning}
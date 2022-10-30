---
title: Intermediate Linux Skills
date: 2022-10-29 17:30:00
author: Aaron
top: false
cover: true 
toc: true
summary: Linux Command
categories: 
  - Linux
tags: 
  - Linux
---


## Wildcards
```yaml
- ?: matches exactly one character
  - ls ??
- *: matches zero or more characters
  - *.txt
- []: a character class
- examples:
  - ca[nt]*: can, cat, candy, catch
  - [!aeiou]*: find patterns do not start with "aeiou"
  - [a-g]: a, b, c, d, e, f, g
  - [1-3]: 1, 2, 3
  - [[:alpha:]], [[:alnum:]], [[:digit:]], [[:lower:]], [[:upper:]], [[:space:]]
  - *\?: done?
```

## IO redirection
```yaml
- Input/Output Types:
  - standard input(stdin): 0 (file descriptor)
  - standard output(stdin): 1 (file descriptor)
  - standard error(stderr): 2 (file descriptor)
- redirection
  - >: redirects standard output to a file and overwirte existing contents
  - >>: redirects standard output to a file and append to existing contents
  - <: redirects input from file to command
  - &: used with redirection to signal that a file descriptor is being used
  - 2>&1: combine stderr and stndard output  (note: no space)
  - 2>file: redirect standard error to a file
  - >/dev/null: redirect output to nowhere
  - examples:
    - ls -l > files.txt = ls -l 1> files.txt
    - ls >> files.txt
    - sort < files.txt
    - sort < files.txt > sorted_files.txt
    - ls files.txt not-here > out: redirect stdout, stderr still show
    - ls files.txt not-here 2> out.err: redirect stderr, stdout still show
    - ls files.txt not-here 1>out 2>out.err
    - ls files.txt not-here > out.both 2>&1
    - ls files.txt not-here 2>/dev/null
```
- [2>&1 vs. 2>>&1](https://superuser.com/questions/1433540/why-it-is-21-and-not-21-to-append-to-a-log-file): When we redirect something by &number, we are not opening a new file at all; we're reusing an already open file along with whatever mode it was opened.

## Search in File
```yaml
- grep: "grep pattern file"
  - options
    - -i: perform a search, ignoring case
    - -c: count the number of occurrences in a file
    - -n: precede output with line numbers
    - -v: invert match. print lines that don't match
  - exmpales:
    - grep -i User file.txt
    - grep -ci User file.txt
    - grep -ni User file.txt
- strings
  - display printable strings
- |: pipe (take stdout from the proceeding command and pass it into the following command, "command-output | command-input")
- cut [file]: cut out selected portions of file. if file is omitted, use stdin
  - options
    - -d: delimiter
    - -f N: display the n-th field
- more, less: piping output to a pager
- tr: transform character
- column: display the contents of a file in columns
- examples:
  - cat file | grep pattern
  - strings file.mp3 | grep -i John
  - strings file.mp3 | grep -i John | head -1
  - strings file.mp3 | grep -i John | head -1 | cut -d' ' -f2
  - grep bob /etc/passwd | cut -d: -f1,5
  - grep bob /etc/passwd | cut -d: -f1,5 | sort | tr ":" " "
  - grep bob /etc/passwd | cut -d: -f1,5 | sort | tr ":" " " | column -t
```

## Environment Variable
```yaml
- pirntenv
  - pirntenv: show all envs
  - printenv NAME: show env value with NAME
- echo $NAME: print env value with NAME
- export NAME="value": set or update env with NAME:value
- unset NAME
```

## Process and Job
```yaml
- ps: display process status
  - -e: everything, all processes
  - -f: full format listing
  - -u: username
  - -p: pid
  - a: all users
  - u: display the process's user/owner
  - x: show processes not attached to a terminal
  - example
    - ps aux: all processes with status
    - ps -eH: display a process tree
    - ps -e --forest
    - ps -u username: display user's processes
- pstree: display processes in a tree format
- top, htop: interactive process viewer
  - ?: question for help
- command &: start command in background
- ctrl-c: kill the foreground process
- ctrl-z: suspend the foreground process
- bg [%num]: background a suspended process
- fg [%num]: foreground a background process
  - fg %1
- kill: kill a process by job number or PID
  - kill [-sig] pid: send a signal to a process
    - kill 123: terminate pid-123
    - kill -15 123: send signal 15 to pid-123
    - kill -TERM 123: send signal TERM to pid-123
  - kill -l: display a list of signals
- jobs [%num]: list jobs
  - jobs %%
  - jobs %+
  - jobs %-
  - jobs %2
  - jobs
```

## Scheduling Repeated Jobs with Cron
```yaml
- cron: a time based job scheduling service
- crontab: a program to create, read, update and delete your job schedules
  - file: install a new crontab from file
  - -l: list your cron jobs
  - -e: edit your cron jobs
  - -r: remove all of your cron jobs
  - format: min hour day month day command
  - shortcut
    - @yearly
    - @annually...
  - examples: should edit the job in a file then run "crontab file"
    - 0 7 * * 1 /opt/sales/bin/weekly-report: run command every monday at 07:00
    - 0 2 * * * /root/backupdb > /tmp/db.log 2>&1: everyday at 02:00
    - 0,15,30,45 * * * * command: every 15 minutes
    - */15 * * * * command: every 15 minutes
    - 0-4 * * * * command: run for the first 5 minutes of the hour
```


## Shell History
```yaml
- store location: ~/.bash_history, ~/.history, ~/.histfile
- history: display the shell history
- ctrl-r: reverse shell history search
  - enter: execute
  - arrows: change the command
  - ctrl-g: cancel the search
- HISTSIZE: controls the number of commands to retain in history (export HISTSIZE=1000)
- !: exclamation mark
  - !N: repeat command line number N
  - !!: repeat the previous command line
  - !string: repeat the most recent command starting with "string"
  - more ! syntax
    - !:N <Event> <Separator> <Word>
    - !: represents a command line
      - !: the most recent command line
      - !: !!
      - !^: represents the first argument (!:1)
      - !$: represents the last argument
    - :N: represents a word on the command line
      - N=0: command
      - N=1: first argument, etc
    - exmpale:
      - head files.txt sorted_files.txt notes.txt
      - !! -> head files.txt sorted_files.txt notes.txt
      - vi !:2 -> vi sorted_files.txt
      - !^ -> files.txt
      - !$ -> notes.txt
```

## Package Management
``` yaml
### atp
- apt-cache
  - apt-cache serach string: search for string
  - apt-cache show package: display info about the package
- apt-get
  - apt-get install -y package: install package
  - apt-get remove package: remove package, leaving config
  - apt-get purge package: remove both package and config
  - apt-get update: update the local list of remote packages
  - apt-get upgrade -y: upgrade all installed packages
- apt: combination of apt-cache and apt-get

### dpkg
```

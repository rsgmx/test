# Linux

## Shells

Advanced Bash Scripting Guide
https://tldp.org/LDP/abs/html/index.html

### 1. Concepts

Different Shells - history. cornshell.



### 2. bash

bash = borne again shell

Types of shells:

<em>(1) Interactive Shell:</em> <br>
>``$ bash``
A shell started without a ``-c command`` argument. IO-Streams will be connected to STDIN and STDOUT. ``$i`` will include ``i``

<code>
$ echo $i <br>
> HimB
</code>

<em>(2) Nontinteractive Shell</em><br>
Shell started with ``bash -c command``. 
>``$bash -c command``

STDIN is not connected(?)

<em>(3) Login Shell:</em><br> 
A shell started with ``--login`` or first argument first character is ``-``
>``$ bash --login``

<em>(4) Restricted Shell:</em><br> 
> ``$ bash -r | --restricted``

#### Questions:
1. How do I find out what kind of shell do I have?
A: ???


### 2. bash-Startup - Invocation
Invocation of the bash shell.
> ``bash -i -r --login --noprofile``

>`` -i == interactive shell``<br>
`` -r | --restricted == restricted shell``<br>
`` --login == login shell``<br>
`` --noprofile == do not load any of the profile files``<br>
``bash -c "command" [arguments]``


### 3. bash configuration

Files: .profile

Aliases

History

Commandline editor

Prompt


### 4. process - job - coprocess

execution context - environment - process group - signals - background vs foreground - signals - pipelines - subshell - process and child process

External Tools:
>`` ps, top, pstree, kill, pidof``

List process for process id PID 8805:
>``$ ps -p 8805 -o pid,pgid,sid,tty,bsdstart,comm``


List process for process group id PGID 22631
>``$ ps -g 22631 -o pid,pgid,sid,tty,bsdstart,comm``

List process for session id SID 31270
>``$ ps -s 22631 -o pid,pgid,sid,tty,bsdstart,comm``


bash-Internals:
>``jobs bg fg disown ``

coprocesses (internal)

> ``coproc [NAME] command [redir]``

Creates a process with name NAME ( default COPROC). Creates an ARRAY NAME[]. Creates Pipes with NAME[0] connected with STDOUT of command, NAME[1] with STDIN of command. Processes redirections. 


#### FAQ:

Q: Can a background process access STDOUT / STDERR ? A: Yes. 

Q: What happens if a background process waits for input from STDIN? A: it gets suspended

Q: Get process ID for the running shell: ``$ echo $PID``

Use variable ``$$`` which expands to the PID of the shell. In a subshell in expands to the PID of the parent shell, not the subshell.

Variable ``$BASHPID`` expands to the PID of the shell. In a subshell it expands to the PID of the subshell.

Consider ``$BASH_SUBSHELL`` which expands to 0 in the mainshell. It expands to 1 in the first subshell. It expands to n indicating the nesting level in further subshells of subshells.

Q: List jobs with PID
>``$ jobs -l``
List process for process group id PGID 22631
>``$ ps -g 22631 -o pid,pgid,sid,tty,bsdstart,comm``
List process for process group id PGID 22631
>``$ ps -g 22631 -o pid,pgid,sid,tty,bsdstart,comm``


Q: What happens with a running child process when the terminal bash exits?
A: bash receives a SIGHUB signal which is sent to child processes that are listed in the job table (unless not disowned)

Q: What is the job control table?

Q: How can I remove a job from the job control table?

Q: What does it mean to remove a job from the control table? A: The subprocess wont receive a SIGHUB signal on main shell termination.

Starting a foreground process:
> ``$ foo.sh``
+ process is created
+ inherits STDIN STDOUT STDERR
+ if shell receives SIGHUB it will send SIGHUB to child. Child normally terminates.

Starting a background process:
> ``$ foo.sh &``
+ process is created
+ inherits STDIN STDOUT STDERR. BUT: attempt to read from STDIN makes child process suspend
+ put into the job control list.
+ if shell receives SIGHUB it will send SIGHUB to child. Child normally terminates

Builtin command ``disown`` removes a job from the table. 

``disown`` (builtin) removes the child process from the job control list (-> so no SIGHUB is sent to the child) but child still connected with STDIN STDOUT STDERR. When the parent bash terminates this will make the child fail because inherited IO-Streams are not available.

>``$  disown [-ar] [-h] [%jobspec | pid ]``
+ without ``-h`` : remove from job table
+ with ``-h``: mark in job table, so that SIGHUB to bash does not get sent to child
+ with ``-a``: remove or mark all jobs


External command ``nohup [command]`` isolates command from the shell.

``$ nohup command`` (external) effectively separate the child from the terminal. "Run program, ignore hangup signals".

> ``$ nohup command`` 
+ provide an unreadable STDIN to the child
+ redirect STDOUT and STDERR for the child to $HOME/nohup.out
+ prevent from receiving SIGHUB signals 
+ BUT: child is still in the job control list!

``$ wait %jobspec `` (builtin) wait for a job to terminate. Usage: in scripts.


Process States:<br>
+ RUNNING == running or scheduled to run
+ WAITING(UNINTERRUPTLY)
+ WAITING(INTERRUPTLY)
+ STOPPED
+ ZOMBIE

How does the process change state?<br>
(1) Interactively through the shell with ``jobs | bg | fg | Ctrl-Z | Ctrl-D | Ctrl-C``:
+ Start a process 
+ then use ``[Ctrl-Z]`` to stop/suspend it. State transits to STOPPED
+ then use ``$ bg`` to resume operation in the background for the process
+ then use ``$ fg %n`` with the jobspecifier to continue in the foreground.

(2) When the process terminates

(3) When a background process tries to read from STDIN<br>
new State: ```STOPPED`` (suspended)

(2) Through signals:
+ ``$ kill -l`` to list available SIGNALS
+ ``$ kill 9 2678`` == send signal 9 to pid 2678
+ or ``$ kill -SIGKILL 2678``

Important Signals:<br>
+ ``1 == SIGHUB`` gets sent when terminal closes. Normal reaction of a child is termination.
+ ``2 == SIGINT`` gets sent upon [Ctrl-C] == interrupt
+ ``3 == SIGQUIT`` gets sent upon [Ctrl-D] == quit
+ ``9 == SIGKILL`` sent through KILL. Terminates the process immediately. Does NOT PERFORM CLEANUP!
+ ``15 == SIGTERM`` sent through KILL default. Terminated orderly.
+ ``20 == SIGSTP`` sent through [Ctrl-Z] == stop=suspend?

#### Process identifier - parent process id - processgroup id

Process Identifier PID and PPID (ParentProcessID)
Processgroup identifier PGID. Session id SID.

A (main) bash has a PID. The main bash is a session leader, main bash PID is the session id. A started subshell bash has a PID of its own, but the ParentID is the PID of the (main) bash. 

All of the subprocesses have the same sessionid which is the PID for the main bash. Main bash is the sessionleader. 

A pipeline consititues a progress group. 

Server Monitoring Tools:
``dstat, vmstat, netstat, iostat, ifstat, mpstat``
all of which can be installed via ``$sudo apt-get install xxx``

External Tools:

>``$ pidof systemd``

>``$ pgrep - u [user]``

>``$ pkill [programname]``

See also ``glances``


### 5. IO - pipes - redirections - files - pipelines

Redirection is interpreted by the shell and takes place BEFORE command execution.
Redirection can OPEN and CLOSE files and provide them to the command. 
Redirection operators can appear anywhere before and after the command and are processed left-to-right. 

<em>File descriptor number</em> ``0..9`` are used implicitly and are reserved. There is implicit defauls: 0 for STDIN and 1 for STDOUT.

``{VARNAME}`` can appear: opens a file descriptor ( greater 10) and assign it to {VARNAME}.

Order of redirections is significant.
>`` ls > DIRLIST 2>&1`` == STDOUT and STDERROR are redirected to the file DIRLIST

>`` ls 2>&1 > DIRLIST `` == only STDOUT is redirected. Reason: 2>&1 happens before redirection to dirlist and STDERR was made a copy of the STDOUT before.

Redirecting INPUT:

>`` [n]<WORD`` == WORD gets expanded. File with name is opened for reading. if n=0 it is STDIN.

Redirecting OUTPUT:

>`` [n]>WORD`` == WORD gets expanded. Files is created and opened for writing. If file x exists (a) it is truncated to size 0 or (b) the redirection may fail (see 'noglobber' option in 'set')
>`` [n]>|WORD`` == attemt redirect even if the file exists.

Redirecting STDOUT and STDERR

Two formats: ``&>WORD`` ( and ``>&WORD``). (`` &>>WORD`` when appending).

Same as: `` >WORD 2>&1`` (``>>WORD 2>&1`` when appending)


Duplicate FileDescriptors:

>`` [n]<&WORD`` == fd n is made a copy of fd word, if fd word is open for reading.

>`` [n]<&-`` == fd n is closed.

>`` [n]>&word`` == fd n is dublicated, if fd word is open for writing.

Move FileDescriptors:

>`` [n]<&DIGIT-`` == moves fd digit to fd n. digit is closed after duplication.

>`` [n]>&DIGIT`` == moves fd digit to fd n. digit is closed after duplication.

Open FileDescriptor for read and write.

>``[n]<>WORD`` == 



### y. Process substitution

>`` <(LIST)``

>`` >(LIST)``

-> LIST is run  with STDIN resp STDOUT connected to a FIFO or some /dev/fd/n file.

Note: No whitespaces between ``<``/``>`` and ``(``!




### 6. Prompt - Editing - Completion - History - Alias - directorystack?

Prompt Editing / Completion



### 7. Files - umask - chmod - globbing



### 8. builtins and externals



### 9. Shell-Scripting

Shell Grammar - SheBang - language constructs - Tests - 

#### 9.1 SOURCE == .

Read and execute commands from a file in the context of the current shell ( no subshell).


> ``source FILENAME [Arguments]``<br>
> ``. FILENAME [Arguments]``

Usage: load functions, variables, configurations

Search for FILENAME in $PATH, currentWorkDir.
ExitCode=0 if found, =1 otherwise

#### Grouping

There is two ways of grouping ``(...LIST...)`` and ``{ ...LIST;... }``.

Redirections may be applied to the entire command list.

``(..)`` creates a subshell and executes all commands in this subshell.

`` {... , } `` does NOT create a subshell but executes in the main shell. Needs a semicolon after list, the curly brackets must be surrounded by whitespace!




#### 9.2 Shell Grammar

Comment in a script.
>`` # Some comment``


Shebang line. See Processes:
>`` #! /bin/bash == Shebang line``

<code>
 { list; } == "group command". executed in current shell context. Whitespaces important!
 Example
 <br>
 ((expression)) == Arithmetic Expression, returning 0 or 1.
 <br>
 [[ expression ]] == conditional expression, returns 0 or 1.
 <br>
 ! expression
 <br>
 expression1 && expression2
 <br>
 expression1 || expression2
 <br>
 for name [[ in [ word ... ]]; ] do list; done
 <br>
 for ((expr1; expr2; expr3)) ; do list; done
 <br>
 select name [ in word] ; do list; done
 <br>
 case word in [] list ... esac
 <br>
 if list; then list; elif list; then list; else list; fi
 <br>
 while list; do list; done
 <br>
 until list; do list; done

</code>

xx. Shell Function definitions

Definition of a function:

> `` function name() compound-command [redirection]``

compound == { list; }. ExitCode is exitcode of the last executed command in the body. Redirections are performed when function is executed.

xx. Parameters and Variables

Shell paramater expansion

``${NAME}``

``${!NAME}`` is <em>parameter indirection</em>


>`` name=[value]``
Null String allowed.
All values undergo certain EXPANSIONs (tilde, etc). PATH-EXPANSION is not performed.

Special Parameters:

<code>
 * ==> "$*" = "$1 $2 ...", when $IFS=' ' 
 <br>
 @ expands to positional p "$@" = "$1" "$2" ... 
 <br>
 # == number of positional parameters
 <br>
 ? == ExitCode of the most recent execute foreground pipeline
 <br>
 - == expands to option flags
 <br>
 $ == expands to processID of the shell
 <br>
 ! == expands to processID of the most recently 
 executed background command
 <br>
 0 == expands to name of shell or shell script
 <br>
 - == expands to absolute pathname of shell 
 when started, lateron changes to last argument of the previous command
</code>

Variables. ARRAY. One-dimensional, zero-based indexed OR associative.
Any variable can be used that way.

>`` declare -a name == declare an indexed array``

>`` declare -A name == declare associative array``


#### xx. Quoting
Three quoting mechanisms: ESC-Chacter, single Quote, double Qoute.

Esc-Character is ``\``
Single quotes: presever meaning. nesting not allowed.

#### xx. command Pipeline: ``|`` and ``|&``

<em>Pipeline</em> is a sequence of commands separated by the pipeline operators.

>`` 'time' -p ! command1 | command2``

Output is connected to the input though a pipe, this happens before any redirection takes place.

Using ``|&`` makes STDERR of command1 be redirected to STDIN of c2, implicit redirection AFTER explicit redirections specified by the command. 

Each command is executed in its own subshell.

``time`` outputs timing statistics. ``!`` negates exit code.

#### xx comand list: ``; & && || ``

<em>command list</cm> is a sequence of commands or pipelines separated by ``; & && || `` and followed by ``; newline &``

if a command is followed by ``&`` => control returns immediately with exit code 0, command gets asynchronously executed in the background. 

``c1 && c2 == AND `` => c2 executed if and only if c1 returns 0.

``c1 || c2 == OR `` => c2 executed if and only if c1 returns non zero.

Return status is exit code of the last command executed.


if commands are separated by ``;`` => commands are executed sequentially, 


#### xx. Wordsplitting and Expansions and command substitution

Listed in order:

<em>brace expansion</em>
<br>May change number of words.
>``somestring{A,B,C}postfix => somestringApostfix somestringBpostfix ..``
<br>
>``st{5..11} => st5 st6 st7 st8.. st11 ``
<br>
>``st{5..11..2} => st5 st7 st9 st11 ``


<em>tilde expansion</em>
<br>
>``~/ABC => HOME /home/rs/ABC``
<br>
>``~+/ABC => CWD /somedir/ABC``
<br>
>``~0/ABC => from DIRS /directoryStack0/ABC``

Tilde must be first character. Tilde-prefix extends to first SLASH.


<em>parameter and variable expansion</em>

<em>command substitution</em>

Comes in two forms $(command) or backtick form \`command\`

Standard output of command will be substiatuted. trailing newlines will be stripped. Embedded newlines will be kept (but may be removed during wordsplitting ).

Substition may be nested.

if in double quotes ` "$(command)"` there is no word splitting or pathname expansion done. 

<em>arithmetic expansion</em>

>``$(( EXPRESSION ))``


<em>word splitting</em>

Performed only when outside double quotes! Perfomed only when there is some form of expansion. 

IFS is used as a word delimiter. 



<em>pathname expansion / globbing</em>

#### xx. Execution environment and Environment

Elements of the execution environment: (1) open files, (2) current working dir, (3) file creation mode mask (umask), (4) traps (trap), (5) shell parameters set by variables or inherited or set (set), (6) shell functions, (7) options (shopt), (8) aliases (alias), (9) various processIDs

Subshell (when simple command == not builtin, not shell function): separate execution environment, inherited from the calling shell. Subshell can not affect environment of the calling shell. 

Note: traps in the subshell are reset to the traps when mainshell.


10. Shell and subshell

+ See Variables $$ $BASHPID $BASH_SUBSHELL
+ bash builtins dont fork a subshell
+ commands do fork a subshell
+ Curly brackets {} dont fork.
+ Group operator () does fork!
    >``$ ( echo "executed in a subshell" )``
+ Concept of "nested subshells". Subshell of a subshell.
+ Usage: Variable isolation. Set up a dedicated environment for mulitple commands:
<code><br>
    COMMAND1 <br>
    ( <br>
        IFS=: <br>
        PATH=\bin <br>
        COMMAND100 <br>
        COMMAND101 <br>
        exit <br>
    ) <br>
    COMMAND2 <br>
</code>


11. ALIAS


12. Environment


13. FUNCTION - typset - declare -F

#### xx

scp

>`` scp root@10.10.0.12:/test.file /root/geheim/cptest.file``

(1) directories must exist
(2) ``-r `` recursively copys entire directories


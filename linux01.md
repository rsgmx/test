# Linux

## Shells

Advanced Bash Scripting Guide
https://tldp.org/LDP/abs/html/index.html

### 1. Concepts

Different Shells - history. cornshell.



### 2. bash

bash = borne again shell

Types of shells:

<em>(1) Nontinteractive Shell</em>

<em>(2) Interactive Shell:</em> <br>A shell started without a ``-c command`` argument. IO-Streams will be connected to STDIN and STDOUT. ``$i`` will include ``i``

<code>
$ echo $i <br>
> HimB
</code>

<em>(3) Login Shell:</em> A shell started with ``--login`` or first argument first character is ``-``

### 2. bash-Startup - Invocation
Invocation of the bash shell.
> ``bash -i -r``

>`` -i == interactive shell``<br>
`` -r | --restricted == restricted shell``<br>
``--login == login shell``<br>
``--noprofile == do not load any of the profile files``


> ``bash -c "command" [arguments]``

> ``- c ``




### 3. bash-Configuration




### 4. process - job - coprocess

Keywords: execution context - environment - process group - signals - background vs foreground - signals - pipelines - subshell - process and child process

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


Q: Can a background process access STDOUT / STDERR ? A: Yes. 

Q: What happens if a background process waits for input from STDIN? A: it gets suspended

Q: Get process ID for the running shell: ``$ echo $PID``

Use ``$$`` which expands to the PID of the shell. In a subshell in expands to the PID of the parent shell, not the subshell.

``$BASHPID`` expands to the PID of the shell. In a subshell it expands to the PID of the subshell.

Consider ``$BASH_SUBSHELL`` which expands to 0 in the mainshell. It expands to 1 in the first subshell. It expands to n indicating the nesting level in furth subshells of subshells.

Q: List jobs with PID
>``$ jobs -l``
List process for process group id PGID 22631
>``$ ps -g 22631 -o pid,pgid,sid,tty,bsdstart,comm``
List process for process group id PGID 22631
>``$ ps -g 22631 -o pid,pgid,sid,tty,bsdstart,comm``


Q: What happens with a running child process when the terminal bash exits?
A: bash receives a SIGHUB signal which is sent to child processes that are listed in the job table (unless not disowned)

Starting a foreground process ``$ foo.sh``
+ process is created
+ inherits STDIN STDOUT STDERR
+ if shell receives SIGHUB it will send SIGHUB to child. Child normally terminates.

Starting a background process ``$ foo.sh &``
+ process is created
+ inherits STDIN STDOUT STDERR. BUT: attempt to read from STDIN makes child process suspend
+ put into the job control list.
+ if shell receives SIGHUB it will send SIGHUB to child. Child normally terminates

``disown`` (builtin) removes the child process from the job control list (-> so no SIGHUB is sent to the child) but child still connected with STDIN STDOUT STDERR. When the parent bash terminates this will make the child fail because inherited IO-Streams are not available.

>``$  disown [-ar] [-h] [%jobspec | pid ]``
+ without ``-h`` : remove from job table
+ with ``-h``: mark in job table, so that SIGHUB to bash does not get sent to child
+ with ``-a``: remove or mark all jobs


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

Change prozess states:<br>
(1) Through the shell with jobs/bg/fg:
+ Start a process 
+ then use [Ctrl-Z] to suspend it. State transits to STOPPED
+ then use ``$ bg`` to resume operation in the background for the process
+ then use ``$ fg %n`` with the jobspecifier to continue in the foreground.

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

Process Identifiers PID and PPID (ParentProcessID)
Processgroup identifier PGID. SessionID.

A bash has a PID. A started subshell has a PID of its own, but the ParentID is the bash. 

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

<em>word splitting</em>

Performed only when outside double quotes! Perfomed only when there is some form of expnasion. 

IFS is used as a word delimiter. 



<em>pathname expansion / globbing</em>






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

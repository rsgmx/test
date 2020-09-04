# ssh ecosystem

openSSH protocol 2

## Components

- ssh
- sshd
- ssh-agent 
- ssh-add 
- ssh-keygen 
- file ssh_config (/etc/ssh/ssh_config and ~/.ssh/config )
- file ~/.ssh/known_hosts
- file ~/.ssh/authorized_keys

*Supported encryption methods*: AES, 3DES, etc, etc

*Supported authentication methods*:<br>
GSSAP, host-based, public-key-based, challenge-response, password, all of which are tried in order.

*Host-based*:<br> 
source machine is listed on the server machine /etc/hosts.equiv et /etc/ssh/shosts.equiv AND user is non-root AND username is same on both sides, then login is allowed. Server must be able to verify the client's host key.

*public-key-based*:<br> 
Based on keypairs or certificates. 
(Server) ~/.ssh/authorized_keys list all public (user) keys that are permitted to login. (Client) ssh proves that it has access to the private key. 

*challenge-response*:

*password*:

ssh database of known hosts (~/ssh/known_hosts), updated automatically.

## Reference

https://docstore.mik.ua/orelly/networking_2ndEd/ssh/index.htm|OReilly



## Client Configuration

Client config files are ``~/.ssh/config`` and ``/etc/ssh/ssh_config``

## Functions and usage of ``ssh``
- login shell 
- execute a remote command.
- tcp Forwarding
- X11 forwarding

- generate keys and certificates

### ``ssh`` login shell

> ``$ ssh user@host -i [identityfile]``

### ``ssh`` execute remote command

> ``$ ssh user@host command``

will execute the command on the host instead of a login shell and exit. ``command`` can also be a complex command like 'sudo xxx | more '. stdout of the command will be sent back to the client.

### ``ssh`` tcp-forwarding:
> ``$ ssh -f -L 1234:localhost:6667 remote.server.org sleep 10  `` <br>
> ``$ irc 1234 ...``

*Notes*: ``-f`` to put ssh in the background. ``sleep 10`` to exit ssh if there is no connect within 10 seconds.

### Generate a public/private keypair:

``ssh-keygen`` provides functions
* generate a public private key pair
* change passphrase
* export from a key in another format ( see -m)
* sign a certificate

> ``$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"``

Will prompt for a location for the keyfile [rsafile]


### Generate a user or host certificate by signing a keypair:

*Note*: ssh certificates are different from X.509 certificates

*User certifcates*

*System certificates*


### Make key available to the ssh agent system

upon login to the shell:

 ```edit file: ~/.profile:```
```
 eval `ssh-agent -s` 
 ssh-add ~/.ssh/[identityfile] 
```

*Comment:*
``ssh-agent -s`` outputs the  shell commands to stdout (setting and exporting environment variables) which are then executed via eval. Notably SSL_AUTH_SOCK=/tmp/xxxx defines the unix socket that ``ssh`` and ``ssh-add`` use for communication with the ''ssh-agent'' process.

Identifyfile generated via ``ssh-keygen``

### ssh-add

With `shh-add` you can provide a *idenity* to the `` ssh-agent``.
The ssh-agent process must be running.

> ``$ ssh-add ~/.ssh/[identityfile] -t 5``

*Comment*:
Optional Parameter `-t` specifies time to live of the credentials in seconds. If you issue the below command the ssh-system will ask you for the passphrase for the identity and make it available for 5/five seconds.


### script

<code>
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

\# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2= agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add ~/.ssh/rs_rsa
	ssh-add ~/.ssh/rs_gmx
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add ~/.ssh/rs_rsa
	ssh-add ~/.ssh/rs_gmx
fi

ssh-add -l

unset env

</code>



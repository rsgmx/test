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

*Supported authentication methods*:
GSSAP, host-based, public-key-based, challenge-response, password, all of which are tried in order.

*Host-based*: source machine is listed in server machine /etc/hosts.equiv et al. AND user is non-root AND username is same on both sides, then login is allowed. Server must be able to verify the client's host key.

*public-key-based*: based on keypairs or certificates. 
(Server) ~/.ssh/authorized_keys list all public keys that are permitted to login. (Client) ssh proves thaat it has access to the private key. 

*challenge-response*:

*password*:

ssh database of known hosts (~/ssh/known_hosts), updated automatically.



## Functions and usage
- login shell 
- execute a remote command.
- tcp Forwarding
- generate keys and certificates


### Use for tcp-forwarding:
> ``$ ssh -f 6667:localhost:6667 remote.server.org sleep 10  `` <br>
> `` irc 6667 ...``

* with ``-f``: put ssh in the background
* with ``sleep 10``: exit ssh if there is no connect within 10 seconds

### Generate a public/private keypair:

> ``$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"``

Will prompt for a location for the keyfile [rsafile]


### Generate a user or host certificate by signing a keypair:

- Note: ssh certificates are different from X.509 certificates

*User certifcates*

*System certificates*




### Make key available to the ssh agent system

 ```edit file: ~/.profile:```
```
 eval `ssh-agent -s` 
 ssh-add ~/.ssh/[rsafile] 
```

*Comment:*
``ssh-agent -s`` outputs the  shell commands to stdout (setting and exporting environment variables) which are then executed via eval. Notably SSL_AUTH_SOCK=/tmp/xxxx defines the unix socket that ``ssh`` and ``ssh-add`` use for communication with the ''ssh-agent'' process.

### ssh-add

With `shh-add` you can provide a *idenity* to the `` ssh-agent``.
The ssh-agent process must be running.
Optional Parameter `-t` specifies time to live of the credentials in seconds. If you issue the below command the ssh-system will ask you for the passphrase for the identity and make it available for 5 seconds.

> ``$ ssh-add ~/.ssh/[identityfile] -t 5``





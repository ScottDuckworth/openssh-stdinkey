# openssh-stdinkey

This project is a fork of [OpenSSH][1] which modifies the
`AuthorizedKeysCommand` directive in `sshd_config` (present in OpenSSH 6.2 and
above) such that the public key of the incoming connection is sent to standard
input of the command, thus providing a means to identify the connecting user
based solely on their public key and not by the username.

The inspiration for this project was to be able to provide a service similar
to some popular version control repository hosting sites, where a user uploads
their SSH public key(s) via a web interface and accesses their repositories
over SSH using a common SSH user account like "git" or "hg".  However, there
are likely many other use cases for this project.

## Branches

This git repository is organized into this branch (master), pristine OpenSSH
branches (those that are just version numbers), and patched OpenSSH branches
(those that end with `-stdinkey`).  The master branch contains this README.md
file and patches suitable for input to the `patch` command against a specific
version of the OpenSSH source code.

## Caveats

* **Warning:** Specifying a command in `AuthorizedKeysCommand` that does not
  consume its standard input *can lead to deadlock* if this patch is applied.
  Specifically, sshd will `write()` all data (the incoming user's key) to
  standard input of the command before it attempts to `read()` any data from
  the command.  If the incoming user's key is greater than the pipe buffer
  size then writing that data will block until what's in the pipe is consumed,
  but in this case there is no consumer.

    * If you must use a command that does not consume its standard input, then
      you should at least close standard input in a wrapper script:

          #!/bin/sh
          exec 0>&- # close stdin
          exec /usr/libexec/command-that-does-not-consume-stdin

* It is probably a good idea to limit the use of the `AuthorizedKeysCommand`
  directive to the specific user which you would like to have this behavior
  using a `Match` block:

      Match user git
          AuthorizedKeysCommand /usr/libexec/lookup-ssh-key

[1]: http://www.openssh.com/

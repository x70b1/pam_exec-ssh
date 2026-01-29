# pam_exec-ssh

[![Actions](https://github.com/x70b1/pam_exec-ssh/actions/workflows/shellcheck.yml/badge.svg)](https://github.com/x70b1/pam_exec-ssh/actions)
[![Contributors](https://img.shields.io/github/contributors/x70b1/pam_exec-ssh.svg)](https://github.com/x70b1/pam_exec-ssh/graphs/contributors)
[![License](https://img.shields.io/github/license/x70b1/pam_exec-ssh.svg)](https://github.com/x70b1/pam_exec-ssh/blob/master/LICENSE)

Unlock SSH keys on login using PAM.

As `pam_ssh` did not the job for me, I wrote `pam_exec-ssh` as a small replacement.
It is assumed that your login password is identical to the password of the keys.


## Installation

For Arch Linux users is already a [pam_exec-ssh-git](https://aur.archlinux.org/packages/pam_exec-ssh-git/) package in the AUR.

Otherwise just copy the script, set the permissions and install the dependencies `pam` and `expect` and optionally `keychain`.

```sh
cp pam_exec-ssh /usr/bin/pam_exec-ssh
chown root:root /usr/bin/pam_exec-ssh
chmod 755 /usr/bin/pam_exec-ssh
```


## Configuration

You need a running `ssh-agent` that have to be started before you login.
If you have `keychain` installed, the script will run it and start the `ssh-agent` automatically.
Be sure to also start it in your `~/.bashrc` to get the environment variables set correctly, for example:

```
eval $(keychain --eval --quiet)
```

Of course you can also start your agent [manually](https://wiki.archlinux.org/index.php/SSH_keys#ssh-agent) or as a [systemd user service](https://wiki.archlinux.org/index.php/SSH_keys#Start_ssh-agent_with_systemd_user).

Make sure that the socket path is correct.
`pam_exec-ssh` use `/run/user/YOUR-USER-ID/ssh-agent.socket` for it.

`pam_exec-ssh` does not to unlock all ssh keys at login.
It might be better to unlock only a selection of frequently used keys.
Create a directory that contains symlinks to all keys that are to be unlocked.
There are several locations that are checked for that directory:

* `~/.ssh/unlock.d`
* `~/.ssh/pam.d`
* `~/.config/ssh/unlock.d`
* `~/.config/ssh/pam.d`

```sh
mkdir ~/.ssh/unlock.d
ln -s ~/.ssh/id_rsa ~/.ssh/unlock.d/id_rsa
```

You can check which keys are unlocked with `ssh-add -l`.

Add the PAM call to your PAM config:

```
auth		optional	pam_exec.so expose_authtok /usr/bin/pam_exec-ssh
```


To make sure that your keys are locked again you can restart your `ssh-agent`.
A good time to do this is when you lock your screen, so all keys are locked when you leave your device but the agent is still prepared for the next use.

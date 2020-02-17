# pam_exec-ssh

[![Codecheck](https://github.com/x70b1/pam_exec-ssh/workflows/Codecheck/badge.svg?branch=master)](https://github.com/x70b1/check_routeros-upgrade/actions)
[![GitHub contributors](https://img.shields.io/github/contributors/x70b1/pam_exec-ssh.svg)](https://github.com/x70b1/pam_exec-ssh/graphs/contributors)
[![license](https://img.shields.io/github/license/x70b1/pam_exec-ssh.svg)](https://github.com/x70b1/pam_exec-ssh/blob/master/LICENSE)

Unlock SSH keys on login using PAM. As `pam_ssh` did not the job for me, I wrote `pam_exec-ssh` as a small replacement.

It is assumed that your login password is identical to the password of the keys.


# Dependencies

* `pam`
* `expect`


# Installation

Just copy the script and set the permissions.

```sh
cp pam_exec-ssh /usr/bin/pam_exec-ssh
chown root:root /usr/bin/pam_exec-ssh
chmod 755 /usr/bin/pam_exec-ssh
```


# Configuration

You need a running `ssh-agent`. You can start your agent [manually](https://wiki.archlinux.org/index.php/SSH_keys#ssh-agent) or as a [systemd user service](https://wiki.archlinux.org/index.php/SSH_keys#Start_ssh-agent_with_systemd_user).

Make sure that the socket path is correct. `pam_exec-ssh` use `/run/user/YOUR-USER-ID/ssh-agent.socket` for it.

Add the call to your PAM config:

```
auth		optional	pam_exec.so expose_authtok /usr/bin/pam_exec-ssh
```

The `ssh-agent` can be slow. So it is sometimes useful not to unlock all ssh keys at the login. It is better to unlock a selection of often used keys. You need a directory `unlock.d` at your local `.ssh` path. There are a few symlinks to all keys that should be unlocked for you.

```sh
mkdir ~/.ssh/unlock.d
ln -s ~/.ssh/id_rsa ~/.ssh/unlock.d/id_rsa
```

You can check which keys are unlocked with `ssh-add -l`.

To make sure that your keys are locked again you can restart your `ssh-agent`. A good time to do this is when you lock your screen, so all keys are locked when you leave your device but the agent is still prepared for the next use.

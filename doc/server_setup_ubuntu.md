# Running `projectxserver` on Ubuntu 16.04

These instructions are part of the instructions for
[Setting up `projectxserver`](/doc/server_setup.md).
Please make sure you have read that document first.

## Introduction

These instructions assume you have access to an Debian or Ubuntu linux
server, and that the server is reachable at your chosen host name.
(`projectx.example.com`)

> Note that these instructions have been verified to work against Ubuntu 16.04.
> The exact commands may differ on your system.

Once the server is running you should log in to it as root and configure it to
run the appropriate `projectxserver` by following these instructions.

## Create a user

Throughout this document, we will mark commands to be run on your
local machine with the shell prompt `local$` and commands to be
run on your server with `server%`, or if running as `root`, `server#`.

The following commands must be executed on the server as the super user, `root`,
perhaps via `sudo su`.

Create a Unix account named `projectx`:

```
server# useradd -m projectx
```

Give yourself SSH access to the `projectx` account on the server (a convenience):

```
server# su projectx
server% cd $HOME
server% mkdir .ssh
server% chmod 0700 .ssh
server% cat > .ssh/authorized_keys
(Paste your SSH public key here and type Control-D and Enter)
server% chmod 0600 .ssh/authorized_keys
```

Create the letsencrypt cache directory:

```
server% mkdir -p projectx/letsencrypt
```

## Copy the `projectxserver` binary to the server

When following the [setup instructions](server_setup.md) you built an
`projectxserver` server binary appropriate for your chosen storage technology
(local disk or a cloud storage provider).

The binary may be named `projectxserver` or have a cloud service suffix,
as with `projectxserver-gcp`.
When we copy the binary we always install it as simply `projectxserver` so the
subsequent instructions are the same for all configurations.

From your workstation, copy the binary to the server (substitute the name of
your binary for `projectxserver-foo`):

```
local$ scp projectxserver-foo projectx@projectx.example.com:projectxserver
```

## Run `projectxserver` on server startup

The following commands must be executed on the server as the super user, `root`.

These instructions assume that your Linux server is running `systemd`.

Create the file `/etc/systemd/system/projectxserver.service` that contains
the following service definition.

```
[Unit]
Description=ProjectX server

[Service]
ExecStart=/home/projectx/projectxserver
User=projectx
Group=projectx
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### Allow `projectxserver` to listen on ports `80` and `443`

The `projectxserver` binary needs to listen on ports `80` and `443` in order to
obtain its TLS certificates through [LetsEncrypt](https://letsencrypt.org/).

Normally only user `root` can bind ports below `1024`.
Instead of running `projectxserver` as `root` (which is generally discouraged),
we will grant the `projectxserver` binary this capability by using `setcap` (as
`root`):

```
server# setcap cap_net_bind_service=+ep /home/projectx/projectxserver
```

Note that you need to run this `setcap` command whenever the `projectxserver`
binary is updated.

### Start the service

Use `systemctl` to enable and start the service:

```
server# systemctl enable --now /etc/systemd/system/projectxserver.service
```

You may also use `systemctl stop projectxserver` and `systemctl restart
projectxserver` to stop and restart the server, respectively.

You can use `journalctl` to see the log output of the server:

```
server# journalctl -f -u projectxserver

```

## Continue

You can now continue following the instructions in
[Setting up `projectxserver`](/doc/server_setup.md).

# Signing up a new user

## Introduction

Before you sign up, you may want to read the
[ProjectX overview](/doc/overview.md) document.
It introduces the concepts and terminology you'll need to understand to use the
system.

## Install the ProjectX tools {#install}

Download an archive of the ProjectX command-line tools from [the download
page](/dl/), and extract it to a directory that's in your system `PATH`.

The archive includes:

- the `projectx` command-line tool (to create, access, share, and administer data
  stored in ProjectX),
- the `projectx-ui` graphical user interface (a visual helper for signing up,
  deploying ProjectX servers, and working with ProjectX data),
- the `cacheserver` daemon (a cache for remote ProjectX data),
- the `projectx-audit` command-line tool (for auditing storage consumption),
- and, on macOS and Linux systems, the `projectxfs` program
  (a [FUSE](https://github.com/libfuse/libfuse) filesystem, to mount the ProjectX
  file system in your local file tree).

> If your operating system is not listed on the download page, you can obtain
> the binaries by installing ProjectX from source.
> First [install Go](https://golang.org/doc/install) and then use `go get` to
> fetch the ProjectX tools and their dependencies and build them:
>
```
$ go get projectx.io/cmd/...
$ go get augie.projectx.io/cmd/projectx-ui
```
> This will install the ProjectX commands to `$GOPATH/bin`, which you should add
> to your system `PATH` if you haven't already.

## Create an ProjectX user {#signup}

You will need to choose an ProjectX user name, which is just an email address you
own.
Your user name is how the ProjectX system and its users will identify you and your
files.
Note that your chosen user name will become a matter of public record in our
[key server log](https://key.projectx.io/log).

Any valid email address is almost certainly a valid ProjectX user name
(see [the faq](faq.md#email-restrictions) for the exceptions to this).

> You may use your regular email address or an ProjectX-specific one; either way
> is fine.
> The address is published in key server logs as well as in any ProjectX path
> name you share, so be sure your email account has whatever spam, anonymity,
> or other protection you feel is necessary.
>
> No email will be sent to the address after this signup step. All future
> ProjectX operations, even updating later to a new key pair, will be validated
> exclusively with the key pair generated during signup.
> Someone with future access to your email can’t masquerade as you in ProjectX.
> Conversely, if you lose your keys your email account is not useful for
> recovery.

**Start the `projectx-ui` program to start the signup process.**

The first step asks for your user name, generates a key pair (one secret key,
one public key), saves the keys locally, and sends your details, including
your public key, to the key server.
The public key is published to the shared ProjectX key server, but the secret key
is stored only on your local computer.

After generating your key pair, `projectx-ui` will display a "secret seed" that
serves as a human-readable version of the key.
(The computer-readable version is just a very long number.)
**Write down this secret seed, keep it somewhere safe and do not lose it. It is
literally your key to ProjectX.**

> ProjectX's security model is based on public key encryption, in which each
> ProjectX user has a pair of keys called the public and private keys.
> The public key is registered with the public key server and is available to
> everyone, while the private key is kept in secret by the user, such as on a
> local workstation or other private device.
>
> It is vital that you do not lose or share your secret key or its "secret
> seed" (which is equivalent to the key itself).
> **If you lose your key and its secret seed
> you will lose access to this ProjectX identity,
> including all the data you have stored and even the ability
> to use your registered user name.**
> There is no way to recover a lost key.
> The high security that ProjectX offers would be compromised if
> there were an account recovery mechanism.

The second step is to receive an email message from the key server and to click
the confirmation link that it contains.
Visiting that link proves to the key server that you control the email address
that you are registering and completes the signup process.

> From here on, the email address serves as your ProjectX user name.
> However, after this account verification step ProjectX will never use it as an
> actual email address again.
> At this point you could even cancel the email account, if you chose to do so,
> without affecting your ProjectX user name.
> In fact, even if the email account is later hijacked, the
> attacker will not be able to get access to your ProjectX account.

## Nominate (and maybe deploy) your ProjectX servers {#deploy}

Next you need to decide whether you are going to deploy your own ProjectX
directory and store servers, use those maintained by someone else, or
skip specifying ProjectX servers entirely.

After you have registered your account, `projectx-ui` prompts you to select one
of three options:

- I will use existing ProjectX servers.
- I will deploy new ProjectX servers to the Google Cloud Platform.
- Skip configuring my servers; I'll use ProjectX in read-only mode for now.

Choose the first option if you want to use ProjectX servers provided by somebody
else, or if you want to deploy your own servers manually (see the [Setting up
`projectxserver`](/doc/server_setup.md) document for how to do this).

Choose the second option to deploy your servers to the Google Cloud Platform
using the `projectx-ui` program, and follow the on-screen instructions to
complete the deployment.

Choose the third option if you wish to use ProjectX as a read-only user.

> If you're unsure, choose the third option, as you can always go back to this
> step later.
>
> To go back, edit your `$HOME/projectx/config` file and remove its `dirserver:`
> and `storeserver:` lines and restart `projectx-ui`.

## Creating your ProjectX directory {#mkdir}

If you choose to use existing servers or to deploy your own, the `projectx-ui`
program will attempt to create a directory in the nominated directory server
that will host your ProjectX tree (your "user root").

> If you are planning to join an existing ProjectX directory and store server,
> make sure to ask the administrator to add your user name to the server's
> `Writers` group. (They'll know what to do.)
> This will grant you permission to create your user root in that directory
> server and store data in that store server.

With the servers running and granting you access permission, and with your
correct information registered in the key server, `projectx-ui` will create
your user root and display its contents.

## Hello, world {#hello}

To prove that your user root was created successfully, try copying a file to
the system.

Do this by dragging a file into the `projectx-ui` directory pane.
If the directory pane refreshes and your file is there, then you are ready to
use ProjectX.
If something is wrong then you will see an error message.

Another way is to use the `projectx cp` command:

```
$ projectx cp ./hello.jpg you@gmail.com/
```

To check that everything worked, copy it back and verify its contents:

```
$ projectx cp you@gmail.com/hello.jpg ./ciao.jpg
$ sum hello.jpg ciao.jpg
1600 21 hello.jpg
1600 21 ciao.jpg
```

Although the `projectx-ui` and `projectx` tools support all the functionality of
the system, for smoother operation you may want to install the FUSE daemon,
`projectxfs`, and a cache server that improves performance.
The cache server is particularly important, and the setup instructions are in
the [ProjectX configuration](/doc/config.md) document.

## Browsing ProjectX Files on Linux and macOS {#projectxfs}

ProjectX includes a tool called `projectxfs` that creates a virtual filesystem
where you can access the ProjectX name space as a regular mounted file system.

Here is an example of its use.

Make a directory in which to mount the ProjectX name space:

```
$ mkdir $HOME/up
```

Then run the `projectxfs` command giving that directory as its sole argument:

```
$ projectxfs $HOME/up
```

Now you have access to the full ProjectX name space:

```
$ ls $HOME/up/you@gmail.com
```

The `projectxfs` command will exit when the file system is unmounted.

If you encounter an error when you run `projectxfs` the second time, such as:

```
mount helper error: fusermount: failed to open mountpoint for reading: Transport endpoint is not connected
fuse.Mount failed: fusermount: exit status 1
```

just unmount the directory and try again.

To learn more about `projectxfs`, see [its documentation](https://godoc.org/projectx.io/cmd/projectxfs).

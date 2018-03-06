# Frequently Asked Questions

### Introduction

In command-line examples on this page, commands to be typed to the shell begin
with a dollar sign "`$`".
Lines that do not begin with "`$`" show command output.

The text uses `you@example.com` as a stand-in for your user name.
If you run one of these commands, substitute your own name for that one.

The examples are Unix-oriented but it should be easy to adapt them to a Windows environment.

## The project

### What is ProjectX? {#what}

ProjectX attempts to provide a secure platform for naming, storing, and sharing
information.

It is not a storage system first, it is a naming system.
It provides protocols for secure storage and sharing that can be
used by people, programs, providers, and products to build a single coherent
way to talk about and manage data.

### What are the goals of the project? {#goals}

The cloud-oriented information world brought much to celebrate,
but it is not always a seamless environment.
It's too often necessary to download a file only to upload it again to
another service, or to make an item publicly accessible when the
goal is just to make it accessible to one person.

ProjectX addresses some of this clumsiness by providing a single,
consistent experience that, if the project succeeds, will allow
people to share data just by naming it, not by copying it around;
to share it securely;
to know who has access;
and to unify the way information is accessed.

### Why create something new rather than work on an existing system? {#why-new}

There are many systems out there that can provide some of what
ProjectX offers, but there is none that addresses its goals directly.

The combination of:

- a single global name space;
- a single global user space;
- a focus on naming rather than storage;
- federated protocols;
- secure end-to-end encryption;
- secure, easily audited sharing;
- open source definition and implementations;

and other properties make ProjectX unique.

Because of the dependence on simple federated protocols rather than a
central service provider, it should be possible to unify other
information service providers by connecting them through
ProjectX.

We prefer to think of ProjectX as a complement to existing services
rather than as a competitor to them.

### Why use a central key server? {#central-keyserver}

The global name space requires that there be only one meaning
for any user name or path name.
This approach, at least conceptually, requires a central key server.
With multiple key servers, names could be ambiguous.

We are aware of the problems of a single point of failure that a
central key server represents, but feel that with suitable
engineering the risk can be mitigated.

The key server is a very simple service, and it could
easily be replaced with another implementation.
We hope one day to replace the existing server with
one built on a public service such as Google's
[Key Transparency](https://github.com/google/keytransparency)
project.

### Can I use ProjectX on my phone? {#mobile}

The story for ProjectX on mobile devices is still evolving.
It's worth stressing that the idea of ProjectX covers more than an app:
it provides a way for all programs to see a consistent view of your data.
Still, an ProjectX app could be useful one day to simplify things
like account management and sharing.

### Can I access ProjectX from a web browser? {#browser}

To access arbitrary ProjectX content from a web browser,
the [`projectx-ui`](https://godoc.org/augie.projectx.io/cmd/projectx-ui) command
serves a local web interface for working with ProjectX trees.

Portions of ProjectX may be made publicly accessible from the web,
as is the case with the [release tree](https://projectx.io/release@projectx.io),
but there is no general web gateway for accessing ProjectX content.

### Why do writes always go to my own store server, even when writing to someone else's directory? {#my-store}

When you create a file in another user's directory, the storage for that
file is placed in your storage server, not that of the directory's owner.
There are several reasons for this, but the most important is that it
keeps accounting clean: you pay only for your storage; others pay
for theirs.

There are other valuable results of this approach.
It allows someone to build a shared directory without bearing all the
cost of hosting everyone's data.
It also means that someone can't consume
your storage quota by filling your storage server with lots of data,
either maliciously or accidentally.

## Users

### Why use email addresses to identify users? {#email}

Email addresses are easily verified, externally managed names for people.
They therefore provide a simple foundation upon which ProjectX can build.

It's important to realize that the name is only used as an actual email
address during account creation.
Once the user is registered, the user name looks like an email address,
of course, but internally it is just a string.
After signup it is never used by ProjectX to send or receive mail.

Those worried about exposing a private email address
can create a temporary one, register with it, and delete the email
account afterwards.
It would still serve as an ProjectX user name.
As a corollary, if the email account is hacked, lost, or transferred,
that would have no effect on the security of the user's data in ProjectX.

The decision to use email addresses as user names would in principle be
easy to change.
It is just a name, after all.
ProjectX could replace the naming mechanism and one day may do so,
although in practice we find email addresses work well.

### Are there restrictions on ProjectX user names? {#email-restrictions}

Any valid email address is likely fine but since it will become as a user name
within ProjectX, the system applies some constraints that could
invalidate some (very unusual) addresses.
Also, within ProjectX user names are case-sensitive,
although in some email services the names are case-insensitive.
The full story is a bit technical and irrelevant for most users, but
if you are interested please see the description
[here](https://godoc.org/projectx.io/user#Parse).

### How do I change the location of my directory and/or store servers? {#change-servers}

If you change the location of your directory and store servers, you must
update the values stored in the key server.
One easy way to do this is to update the values in `$HOME/config` and run:

```
$ projectx user | projectx user -put
```

## Security

### How do ProjectX servers authenticate users? {#server-auth}

ProjectX clients identify themselves to ProjectX servers with HTTP
headers containing an authentication request signed with their private key.
The details are described in
[the rpc package](http://godoc.org/projectx.io/rpc#hdr-Authentication).

### Are my files encrypted? {#encryption}

Although ProjectX supports unencrypted ("plaintext") files,
we expect most users to employ end-to-end encryption to store their data.
EE packing is the default for a storage server.

Files are also cryptographically signed, to detect tampering.

The process of encryption and decryption happens on the client, which means
that neither the directory server nor the store server—nothing in the network—sees
the unencrypted contents of the file.

### How does encryption work? {#encryption-mechanism}

For simplification, let's discuss the most common case here,
creating a file to be read by the owner alone.

When a file is created, it is encrypted using
[AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
encryption with a unique "file key" that is generated by a random
process at the time of creation of the file.
AES is a _symmetric encryption_ technology: it uses the same key
to decrypt as to encrypt.

The file key is stored in the directory entry for the file,
but that key is itself encrypted such that only the set of users
with "read" access can decrypt it.

That second level of encryption is done using an
[elliptic curve algorithm](https://www.imperialviolet.org/2010/12/04/ecc.html),
which uses distinct keys for encrypting and decrypting.
The key for that encryption is the writer's _public_ key, the one
saved in the key server at [key.projectx.io](key.projectx.io).
This approach means that only the person with the corresponding
_private_ key—the owner of the file—can recover the file
key and use it to decrypt the original data.

To read the file, the owner takes the encrypted file key from the
directory entry, decrypts it using the owner's _private_ key,
and then decrypts the file's contents using the file key.

The system is thus a two-level encryption method involving three keys: the AES
file key and the writer's public and private elliptic curve keys. The file key
is unique for every file, while a given user's public and private keys are the
same in every file operation by that user throughout ProjectX.

### How does sharing work? {#sharing}

To be able to read an encrypted file, the reader must be able to acquire a
cleartext version of the file key.
Thus, every potential reader needs to have
an encrypted file key stored in the directory entry.

If the list of readers is known when the file is created, the solution is
simple: add to the directory entry an encrypted file key for _each_ reader.

If a _new_ reader, say a friend, is to be granted the right to read a file after
it has been created, the first step is to use access control to give permission
to the friend.

Then, the owner of the file decrypts the file key, just as in
the case of reading the file, and encrypts another copy using the _friend's_
public key. The owner then saves both encryptions of the file key in the
directory entry for the file.

In either case, when the friend wants to read the file, it's the same procedure
as for the owner: get the encrypted file key from the directory entry, decrypt
that, and use it to decrypt the file.

### What is the relationship between encryption and access control? {#encryption-access}

Access control defines who has access to a file or directory.
There are several distinct rights that may be granted,
including the right to read a file, but also to write one,
to create one, to delete one, or to "list" one, that is, to see if it exists.

For a user to be able to read a file, the user must be granted
access using the access control mechanisms described
[elsewhere](/doc/access_control.md).
If the file is encrypted there must also be a copy of the file key encrypted
with the user's public key and stored in the directory entry, as described above,

Access control is a programmatic way to protect information,
but is theoretically possible to subvert by malicious means
such as through a compromised directory server.
The file encryption technique, however, is much stronger
and therefore provides the most important
of protection of the data.

The relationship between access control and the storage
of users' keys in the directory entry can get out of sync.
To bring them into alignment, use the `projectx share`
command.

One day we hope to automate this reconciliation, but for
now it must be done manually if read access control rights
are changed after a file is written.

### How can I trust the files I write are not modified? {#integrity}

Data stored in ProjectX is cryptographically signed.
The signature depends not only on the data, but also who wrote it and
the original path name of the data, among other things.
This means that the client can detect if a file has been tampered
with, as it will see that the data (or writer or path name) does not
match the signature with which it was created.

A packing (data storage method) called end-to-end integrity packing is
available.
This packing does not encrypt the data, but it still signs it, so even though
the data can be stored in clear text, if it is tampered with, the tampering
will be caught.

### Are world-readable ("read:all") files encrypted? {#read-all}

If you use the EE Packing (the default), world-readable files (`Access` and
`Group` files, and any file shared with the permission `read: all`) are
encrypted in StoreServer, but the decryption key is stored in clear text in the
directory entry on the DirServer. This means that anyone with access to read
the DirEntry may decrypt the objects in the StoreServer.

### Can I share an individual file with other users? {#share-one-file}

ProjectX's [access control mechanism](access_control.md) works at the directory
level and so there is no way to specify the access control rights for a
specific file unless it is the only file in the directory. This seems
inconvenient when you wish to share a specific file while keeping the others in
that directory private, but there is a workaround.

File copies in ProjectX are essentially free: if you `projectx cp` a file from one
ProjectX location to another then both files share the same underlying storage.
To share a specific file with a specific user, create a directory for that user
(`you@gmail.com/them@example.org`, for example), put an `Access` file in that
directory granting them read rights, and then use `projectx cp` to copy the file
there.

## Keys

ProjectX users have a key pair for authentication and security.
The details of their operation is covered in the [ProjectX Security](/doc/security.md) document.
The questions here cover the main points at a high level.

### How are keys set up? {#keys}

Every user has two keys.
A public one is stored in the key server at `key.projectx.io` and is visible to anyone.
A secret key is coupled to the public key but is stored only on the local machine.
It should never be shared with anyone or published.
It should also be kept safely; if you lose it, you lose all access to ProjectX, including
the right to use your registered user name with the system.

Locally, these keys are stored, by default, in the directory

```
$HOME/.ssh/you@example.com
```

in files named `public.projectxkey` and `secret.projectxkey`.

See the document "not yet written" for a discussion of how keys provide
security and safe sharing.

### I lost my private key. How do I recover it? {#lost-key}

When you signed up, you were encouraged to write down a "secret seed",
a string of letters that can be used to recreate your keys.
It looks something like this:

```
zapal-zuhiv-visop-gagil.dadij-lnjul-takiv-fomin
```

Use that secret to recreate your public and private keys:

```
$ projectx keygen -secretseed zapal-zuhiv-visop-gagil.dadij-lnjul-takiv-fomin $HOME/.ssh/you@example.com
```

This will write the private and public keys to the named directory.
If you have a `secrets` field in your `config` that names a different
directory, use that directory instead as the argument.

You can test that this has succeeded by running,

```
$ projectx user
```

This command will show you your configuration and also compare it with
the record stored in the public key server at `key.projectx.io`.
If there is any discrepancy, it will let you know.

### How do I recreate my keys and config on another machine? {#recreate-keys}

First generate the keys with the `projectx` `keygen` command and its
`-secretseed` flag as described in the previous answer.

Then you must recreate your config file on the new machine.
The easiest way is just to copy the existing one using `scp` or some equivalent.
Examine it after copying and correct any settings that should
be different on the new machine.
See the [ProjectX configuration](/doc/config.md) document for details.

### I lost my secret seed. How do I recover it? {#lost-seed}

It is stored in your private key file, so if you still have that you can just
look inside (the format is just one long line):

```
$ cat $HOME/.ssh/you@example.com/secret.projectxkey
25500039906333220349315666410231979076544554594099570935052437564736835632126 # javam-lumat-ganiv-jizan.vodos-huhuz-jogud-darin
$
```

If you have neither your secret seed nor your private key saved, you will be unable to access ProjectX.
There is no account recovery mechanism.

### I lost my keys. How do I recover my account? {#lost-account}

If you have your secret seed, see the previous sections.

Otherwise, you cannot recover your keys or your ProjectX account.

Without your private key or secret seed to regenerate it, there is no mechanism to restore a lost account.
ProjectX attempts to be very secure, and it is our experience that account
recovery mechanisms have historically been the weak link in account security.
We believe the system is more secure by not providing one.

**Do not lose your secret seed!**


### How do I delete my account on the key server? {#delete-account}

If you decide to stop using ProjectX, there is no need to remove anything from the key server.

The [transaction log](https://key.projectx.io/log) is cryptographically protected by a hash chain,
so deletions are impossible anyway.
This protects you from an adversary removing your public key and
re-registering the account with their own key,
locking you out and tricking your friends into leaking secrets.


### How do I change my key? {#rotate-keys}

Most people will never need to change their key;
ProjectX's intent is that you share access to your files, not your keys.
But sometimes a key change may be justified, so here are the details.

When you change your key, you migrate from one working key to another.
The key change process maintains your old private key
so you can authenticate you to the servers during the transition.
When done, all your old keys will be saved so that you can still decrypt files from backups,
but the old keys will no longer work for authentication.

Suppose you are on a client device with existing keys in the default directory
`$HOME/.ssh/you@example.com`.
Here are the commands to switch to using a new key.
Eventually these steps may be combined but at the moment it must
be done in several steps, both
to make it easier to recover if a step fails and so we can better understand
people's needs before designing a smoother process.

```
$ projectx keygen -rotate $HOME/.ssh/you@example.com
$ projectx countersign
$ projectx rotate
$ projectx share -r -fix you@example.com/
```

As always, you must write down the secret seed
printed by `keygen` and save it someplace safe!

The `countersign` and `share` commands may run for a few minutes
as they walk your whole ProjectX file tree.
Also, if you have written
files to other users' directories, you must run `share` there as well.

After running those commands, you're done!
But if you want to understand what these commands mean,
please read on.

The `keygen` command creates a new key pair in the two files,

```
$HOME/.ssh/you@example.com/public.projectxkey
$HOME/.ssh/you@example.com/secret.projectxkey
```

It also saves old key pairs in

```
$HOME/.ssh/you@example.com/secret2.projectxkey
```

After running this one command, nothing has changed yet except the contents
of these three files on your local system; you could revert to the old ones
and be back where you started.

The `countersign` command walks through your ProjectX file tree and adds
signatures using the new key alongside those using the old.
Recall that each file in ProjectX is signed by the writer.
There is room in each directory entry for a second signature,
which in principle could be written by anyone else that wants to certify the file contents,
somewhat analogous to the physical world requirement for some bank checks to be
signed by one person and countersigned by another.
Here the "other person" is you with your new key identity.
After this command, files in your tree can be read and verified by
anyone who knows your old or your new public key.

The `rotate` command pushes the new key to the public key server at `key.projectx.io`,
making it visible to others,
who must now use your new key when sharing files with you.
If there is a file you wrote (and signed) in someone else's tree a long time ago and have not updated,
readers will see an error because the signature will no longer verify;
this can be fixed by running the `share` command.

The `share` command walks the specified file trees, re-wrapping the encryption keys to use
the new keys instead.
It's a good idea to do this, but it's not urgent because the `secret2.projectxkey` file holds the
information necessary for the system to decrypt files that have not been updated.

However, if you are changing keys because you fear your old keys leaked, run `share` as soon as possible,
to disassociate the files from your old key.

## Programming

### What does '"remote" not registered mean'? {#remote}

If you write your own ProjectX program and on startup it fails with an error like,

```
bind.KeyServer: service with transport "remote" not registered
```

it means your program is missing the code that connects it to the
`KeyServer` implementation specified in the ProjectX `config` file.
The layer that makes this connection is called a _transport_, and
`remote` is the transport that connects to a network service.

The simplest fix is to add the import declaration,

```
import _ "projectx.io/key/transports"
```

to the Go source file that defines the `main` function.
This import collects all the transport implementations in the core ProjectX
libraries and adds them to your program.

The underscore in the import declaration is necessary.
It tells the Go compiler that the import statement is being done for
side effects only, in this case the linking into the binary of the
full suite of transports.


## Server administration

This section shows how to perform some common tasks around
administrating an ProjectX server installation.
The examples assume you have signed up as `ann@example.com` as
described in [Signing up a new user](/doc/signup.md) and have [set up
`projectxserver`](/doc/server_setup.md) as a combined dirserver/storeserver for
the domain `example.com`.
Substitute your own user and domain names in the examples that follow.

### How do I add a family member to my ProjectX installation?

When you followed [the server setup instructions](/doc/server_setup.md),
`projectx setupserver` created an initial version of the access control
files for your ProjectX installation that allows access for `ann@example.com` and
the server user `projectx@example.com`.
The key piece is a group file, owned by the server user, called `Writers`.
This file specifies the complete list of ProjectX users that,
once authenticated, are allowed to write blocks to the server.

The easiest way to modify the `Writers` file is to use the `projectx
setupwriters` command. The arguments list the users (or wildcards) to be
granted access.
The server user `projectx@example.com` is always included automatically.

```
$ projectx setupwriters -domain=example.com ann@example.com anotherone@example.com third@example.com
# ... or with wildcards
$ projectx setupwriters -domain=example.com ann@example.com '*@example.com'
```

Every time you call `projectx setupwriters` the `Writers` file is completely replaced;
you must always include the full set of users to have write permission.

Another way to maintain the file is to edit it locally on your machine.
To do this, you need to run the `projectx` command using the config file
defined for the server user:

```
$ projectx -config=$HOME/projectx/deploy/example.com/config cp projectx@example.com/Group/Writers /tmp/Writers
```

Then edit it locally and copy it back once done:

```
$ projectx -config=$HOME/projectx/deploy/example.com/config cp /tmp/Writers projectx@example.com/Group/Writers
```


### How do I delete unused blocks from my storage server?

The `projectx audit` command has a number of subcommands that allow an
administrator to determine how much storage a given user is consuming,
to see how much storage is used overall by the storage server,
and  to create a list of active blocks in the storage servers.
After collecting such data, it can perform other tasks such as removing
blocks from the storage server that are not referenced by the directory
server.

Run

```
$ projectx audit help
```

for more information.

**The `projectx audit` command is simpleminded in its accounting
and should be used with great care.
In the presence of snapshots or multiple users sharing a storage
server it is easy to make a mistake and delete active data.
Use `projectx audit` with great care.**

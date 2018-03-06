# Dropbox-specific server setup instructions

These instructions are part of the instructions for
[Setting up `projectxserver`](/doc/server_setup.md).
Please make sure you have read that document first.

## Build `projectxserver-dropbox` and `projectx-setupstorage-dropbox`

To use Dropbox Storage fetch the `dropbox.projectx.io` repository and use the
`projectxserver-dropbox` and `projectx-setupstorage-dropbox` variants.

Fetch the repository and its dependencies:

```
local$ go get -d dropbox.projectx.io/cmd/...
```

Install the `projectx-setupstorage-dropbox` command:

```
local$ go install dropbox.projectx.io/cmd/projectx-setupstorage-dropbox
```

Build the `projectxserver-dropbox` binary:

```
local$ GOOS=linux GOARCH=amd64 go build dropbox.projectx.io/cmd/projectxserver-dropbox
```

## Get an Dropbox authorization code

1. Visit the following [site](https://www.dropbox.com/oauth2/authorize?client_id=wt1281n3q768jj3&response_type=code).
2. If necessary, log in with your Dropbox credentials and click on "Allow".
   <img src="/images/dropbox/allow.png" alt="Allow ProjectX storage server to access Dropbox"/>
3. Copy the displayed authorization code.
   <img src="/images/dropbox/code.png" alt="Dropbox API code"/>

Now run the `projectx-setupstorage-dropbox` command and pass the previously copied code as
argument:

```
local$ projectx setupstorage-dropbox -domain=example.com <code>
```

It should produce an output like this:

```
You should now deploy the projectxserver binary and run 'projectx setupserver'.
```


## Notes

All ProjectX data will be stored under the app folder `projectx` in the user Dropbox (`/App/projectx`).
Users should [disable syncing](https://www.dropbox.com/lp/pro/pro_onboarding_selective_sync) for
this folder on their Dropbox clients.

## Continue

You can now continue following the instructions in
[Setting up `projectxserver`](/doc/server_setup.md).

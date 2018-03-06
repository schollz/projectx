# GCP-specific server setup instructions

These instructions are part of the instructions for
[Setting up `projectxserver`](/doc/server_setup.md).
Please make sure you have read that document first.

## Build `projectxserver-gcp` and `projectx-setupstorage-gcp`

To use Google Cloud Storage fetch the `gcp.projectx.io` repository and use the
`projectxserver-gcp` and `projectx-setupstorage-gcp` variants.

Fetch the repository and its dependencies:

```
local$ go get -d gcp.projectx.io/cmd/...
```

Install the `projectx-setupstorage-gcp` command:

```
local$ go install gcp.projectx.io/cmd/projectx-setupstorage-gcp
```

Build the `projectxserver-gcp` binary:

```
local$ GOOS=linux GOARCH=amd64 go build gcp.projectx.io/cmd/projectxserver-gcp
```

## Create a Google Cloud Project

First create a Google Cloud Project and associated Billing Account by visiting the
[Cloud Console](https://cloud.google.com/console).
See the corresponding documentation
[here](https://support.google.com/cloud/answer/6251787?hl=en) and
[here](https://support.google.com/cloud/answer/6288653?hl=en)
for help.
For the project name, we suggest you use a string similar to your domain.
(We will use `example-com`.)

Then, install the Google Cloud SDK by following
[the official instructions](https://cloud.google.com/sdk/downloads).

Finally, use the `gcloud` tool to enable the required APIs:

```
local$ gcloud components install beta
local$ gcloud config set project example-com
local$ gcloud auth login
local$ gcloud beta services enable iam.googleapis.com
local$ gcloud beta services enable storage_api
```

## Create a Google Cloud Storage bucket

Use the `gcloud` tool to obtain "application default credentials" so that the
`projectx setupstorage-gcp` command can make changes to your Google Cloud Project:

```
local$ gcloud auth application-default login
```

Now use `projectx setupstorage-gcp` to create a storage bucket and an associated
service account for accessing the bucket.
Note that the bucket name must be globally unique among all Google Cloud
Storage users, so it is prudent to include your domain name in the bucket name.
(We will use `example-com-projectx`.)

```
local$ projectx setupstorage-gcp -domain=example.com -project=<project> example-com-projectx
```

It should produce output like this:

```
Service account "projectxstorage@example-com.iam.gserviceaccount.com" created.
Bucket "example-com-projectx" created.
You should now deploy the projectxserver binary and run 'projectx setupserver'.
```

## Continue

You can now continue following the instructions in
[Setting up `projectxserver`](/doc/server_setup.md).

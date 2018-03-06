# AWS-specific server setup instructions

These instructions are part of the instructions for
[Setting up `projectxserver`](/doc/server_setup.md).
Please make sure you have read that document first.

## Build `projectxserver-aws` and `projectx-setupstorage-aws`

To use Amazon Web Services fetch the `aws.projectx.io` repository and use the
`projectxserver-aws` and `projectx-setupstorage-aws` variants.

Fetch the repository and its dependencies:

```
local$ go get -d aws.projectx.io/cmd/...
```

Install the `projectx-setupstorage-aws` command:

```
local$ go install aws.projectx.io/cmd/projectx-setupstorage-aws
```

Build the `projectxserver-aws` binary:

```
local$ GOOS=linux GOARCH=amd64 go build aws.projectx.io/cmd/projectxserver-aws
```

## Install the AWS CLI

Ensure you have a working AWS environment set up before continuing and that you
are able to run basic commands using the
[CLI tool](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html).

## Set up storage, role account, and instance profile

Use `projectx setupstorage-aws` to create an S3 bucket, an associated
role account, and instance profile for accessing the bucket and provisioning.
Note that the bucket name must be globally unique among all AWS users, so it is
prudent to include your domain name in the bucket name.
(We will use `example-com-projectx`.)

```
local$ projectx setupstorage-aws -domain=example.com example-com-projectx
```

It should produce output like this:

```
You should now deploy the projectxserver binary and run 'projectx setupserver'.
```

If the command fails, it may leave things in an incomplete state.
You can use the -clean flag to clean up any potential entities created:

```
local$ projectx setupstorage-aws -clean -role_name=projectxstorage -domain=example.com example-com-projectx
```

**Notes**:

- The role has access to all S3 buckets by default. To restrict its access to
  only one bucket, follow [this guide](https://aws.amazon.com/blogs/security/how-to-restrict-amazon-s3-bucket-access-to-a-specific-iam-role/).
- The role name is also used as the name for the
  [instance profile](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)
  you should use to provision the instance.
- If you are running `projectxserver` on an EC2 instance, ensure that your
  security group allows inbound TCP traffic on ports 80 and 443.

## Continue

You can now continue following the instructions in
[Setting up `projectxserver`](/doc/server_setup.md).

---
title: Contributing to Terragrunt
layout: single
author_profile: true
sidebar:
  nav: "terragrunt-details"
---

## Running locally

To run Terragrunt locally, use the `go run` command:

```bash
go run main.go plan
```

## Dependencies

* Terragrunt uses `glide`, a vendor package management tool for golang. See the glide repo for
  [installation instructions](https://github.com/Masterminds/glide#install).

#### Running tests

**Note**: The tests in the `dynamodb` folder for Terragrunt run against a real AWS account and will add and remove
real data from DynamoDB. DO NOT hit `CTRL+C` while the tests are running, as this will prevent them from cleaning up
temporary tables and data in DynamoDB. We are not responsible for any charges you may incur.

Before running the tests, you must configure your [AWS credentials](#aws-credentials) and [AWS IAM policies](#aws-iam-policies).

To run all the tests:

```bash
go test -v -parallel 128 $(glide novendor)
```

To run only the tests in a specific package, such as the package `remote`:

```bash
cd remote
go test -v -parallel 128
```

And to run a specific test, such as `TestToTerraformRemoteConfigArgsNoBackendConfigs` in package `remote`:

```bash
cd remote
go test -v -parallel 128 -run TestToTerraformRemoteConfigArgsNoBackendConfigs
```


## Debug logging

If you set the `TERRAGRUNT_DEBUG` environment variable to "true", the stack trace for any error will be printed to
stdout when you run the app.


## Error handling

In this project, we try to ensure that:

1. Every error has a stacktrace. This makes debugging easier.
1. Every error generated by our own code (as opposed to errors from Go built-in functions or errors from 3rd party
   libraries) has a custom type. This makes error handling more precise, as we can decide to handle different types of
   errors differently.

To accomplish these two goals, we have created an `errors` package that has several helper methods, such as
`errors.WithStackTrace(err error)`, which wraps the given `error` in an Error object that contains a stacktrace. Under
the hood, the `errors` package is using the [go-errors](https://github.com/go-errors/errors) library, but this may
change in the future, so the rest of the code should not depend on `go-errors` directly.

Here is how the `errors` package should be used:

1. Any time you want to create your own error, create a custom type for it, and when instantiating that type, wrap it
   with a call to `errors.WithStackTrace`. That way, any time you call a method defined in the Terragrunt code, you
   know the error it returns already has a stacktrace and you don't have to wrap it yourself.
1. Any time you get back an error object from a function built into Go or a 3rd party library, immediately wrap it with
   `errors.WithStackTrace`. This gives us a stacktrace as close to the source as possible.
1. If you need to get back the underlying error, you can use the `errors.IsError` and `errors.Unwrap` functions.


## Formatting

Every source file in this project should be formatted with `go fmt`. There are few helper scripts and targets in the
Makefile that can help with this (mostly taken from the [terraform repo](https://github.com/hashicorp/terraform/)):

1. `make fmtcheck`

   Checks to see if all source files are formatted. Exits 1 if there are unformatted files.
1. `make fmt`

    Formats all source files with `gofmt`.
1. `make install-pre-commit-hook`

    Installs a git pre-commit hook that will run all of the source files through `gofmt`.

To ensure that your changes get properly formatted, please install the git pre-commit hook with `make install-pre-commit-hook`.


## Releasing new versions

To release a new version, just go to the [Releases Page](https://github.com/gruntwork-io/terragrunt/releases) and
create a new release. The CircleCI job for this repo has been configured to:

1. Automatically detect new tags.
1. Build binaries for every OS using that tag as a version number.
1. Upload the binaries to the release in GitHub.

See `circle.yml` and `_ci/build-and-push-release-asset.sh` for details.
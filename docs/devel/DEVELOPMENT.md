## Development

OPA is written in the [Go](https://golang.org) programming language.

If you are not familiar with Go we recommend you read through the [How to Write Go
Code](https://golang.org/doc/code.html) article to familiarize yourself with the standard Go development environment.

Requirements:

- Git
- GitHub account (if you are contributing)
- Go (version 1.14+ is supported though older versions are likely to work)
- GNU Make

## Getting Started

After cloning the repository, just run `make`. This will:

- Build the OPA binary.
- Run all of the tests.
- Run all of the static analysis checks.

If the build was successful, a binary will be produced in the top directory (`opa_<OS>_<ARCH>`).

Verify the build was successful with `./opa_<OS>_<ARCH> run`.

You can re-build the project with `make build`, execute all of the tests
with `make test`, and execute all of the performance benchmarks with `make perf`.

The static analysis checks (e.g., `go fmt`, `golint`, `go vet`) can be run
with `make check`.

> To correct any imports or style errors run `make fmt`.

## Workflow

1. Go to [https://github.com/open-policy-agent/opa](https://github.com/open-policy-agent/opa) and fork the repository
   into your account by clicking the "Fork" button.

1. Clone the fork to your local machine.

    ```bash
    # Note: With Go modules this repo can be in _any_ location,
    # and does not need to be in the GOSRC path.
    git clone git@github.com/<GITHUB USERNAME>/opa.git opa
    cd opa
    git remote add upstream https://github.com/open-policy-agent/opa.git
    ```

1. Create a branch for your changes.

    ```
    git checkout -b somefeature
    ```

1. Update your local branch with upstream.

    ```
    git fetch upstream
    git rebase upstream/main
    ```

1. Develop your changes and regularly update your local branch against upstream.

    - Be sure to run `make check` before submitting your pull request. You
      may need to run `go fmt` on your code to make it comply with standard Go
      style.

1. Commit changes and push to your fork.

    ```
    git commit -s
    git push origin somefeature
    ```

   > Make sure to use a [good commit message](../../CONTRIBUTING.md#commit-messages)

1. Submit a Pull Request from your fork. See the official [GitHub Documentation](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork)
   for instructions to create the request.

    > Hint: You should be prompted to with a "Compare and Pull Request" button
      that mentions your new branch on [https://github.com/open-policy-agent/opa](https://github.com/open-policy-agent/opa)

1. Once your Pull Request has been reviewed and signed off please squash your
   commits. If you have a specific reason to leave multiple commits in the
   Pull Request, please mention it in the discussion.

   > If you are not familiar with squashing commits, see [the following blog post for a good overview](http://gitready.com/advanced/2009/02/10/squashing-commits-with-rebase.html).

## Built-in Functions

[Built-in Functions](https://www.openpolicyagent.org/docs/latest/policy-reference/#built-in-functions)
can be added inside the `topdown` package in this repository.

Built-in functions may be upstreamed if they are generally useful and provide functionality that would be
impractical to implement natively in Rego (e.g., CIDR arithmetic). Implementations should avoid thirdparty
dependencies. If absolutely necessary, consider importing the code manually into the `internal` package.

All built-in function implementations must include a test suite. See [test/cases/testdata/helloworld](https://github.com/open-policy-agent/opa/blob/main/test/cases/testdata/helloworld)
in this repository for an example of how to implement tests for your built-in functions.

## Benchmarks

Several packages in this repository implement benchmark tests. To execute the
benchmarks you can run `make perf` in the top-level directory. We use the Go
benchmarking framework for all benchmarks. The benchmarks run on every pull
request.

To help catch performance regressions we also run a batch job that compares the
benchmark results from the tip of main against the last major release. All of
the results are posted and can be viewed
[here](https://opa-benchmark-results.s3.amazonaws.com/index.html).

## Dependencies

OPA is a Go module [https://github.com/golang/go/wiki/Modules](https://github.com/golang/go/wiki/Modules)
and dependencies are tracked with the standard [go.mod](../../go.mod) file.

We also keep a full copy of the dependencies in the [vendor](../../vendor)
directory. All `go` commands from the [Makefile](../../Makefile) will enable
module mode by setting `GO111MODULE=on GOFLAGS=-mod=vendor` which will also
force using the `vendor` directory.

To update a dependency ensure that `GO111MODULE` is either on, or the repository
qualifies for `auto` to enable module mode. Then simply use `go get ..` to get
the version desired. This should update the [go.mod](../../go.mod) and (potentially)
[go.sum](../../go.sum) files. After this you *MUST* run `go mod vendor` to ensure
that the `vendor` directory is in sync.

Example workflow for updating a dependency:

```bash
go get -u github.com/sirupsen/logrus@v1.4.2  # Get the specified version of the package.
go mod tidy                                  # (Somewhat optional) Prunes removed dependencies.
go mod vendor                                # Ensure the vendor directory is up to date.
```

If dependencies have been removed ensure to run `go mod tidy` to clean them up.


### Tool Dependencies

Sometimes we use some tools which are versioned and vendored
with OPA as dependencies. For now, we have none, but any we use in the future
should go in [tools.go](../../tools.go).

More details on the pattern: [https://github.com/go-modules-by-example/index/blob/master/010_tools/README.md](https://github.com/go-modules-by-example/index/blob/master/010_tools/README.md)

Update these the same way as any other Go package. Ensure that any build script
only uses `go run ./vendor/<tool pkg>` to force using the correct version.

## Go

If you need to update the version of Go used to build OPA you must update these
files in the root of this repository:

* `.go-version`- which is used by the Makefile and CI tooling. Put the exact go
  version that OPA should use.

# CI Configuration

OPA uses Github Actions defined in the [.github/workflows](../../.github/workflows)
directory.

## Github Action Secrets

The following secrets are used by the Github Action workflows:

| Name | Description |
|------|-------------|
| S3_RELEASE_BUCKET | AWS S3 Bucket name to upload `edge` release binaries to. Optional -- If not provided the release upload steps are skipped. |
| AWS_ACCESS_KEY_ID | AWS credentials required to upload to the configured `S3_RELEASE_BUCKET`. Optional -- If not provided the release upload steps are skipped. |
| AWS_SECRET_ACCESS_KEY | AWS credentials required to upload to the configured `S3_RELEASE_BUCKET`. Optional -- If not provided the release upload steps are skipped. |
| DOCKER_IMAGE | Full docker image name (with org) to tag and publish OPA images. Optional -- If not provided the image defaults to `openpolicyagent/opa`. |
| DOCKER_WASM_BUILDER_IMAGE | Full docker image name (with org) to tag and publish WASM builder images. Optional -- If not provided the image defaults to `openpolicyagent/opa-wasm-builder`. |
| DOCKER_USER | Docker username for uploading release images. Will be used with `docker login`. Optional -- If not provided the image push steps are skipped. |
| DOCKER_PASSWORD | Docker password or API token for the configured `DOCKER_USER`. Will be used with `docker login`. Optional -- If not provided the image push steps are skipped. |
| SLACK_NOTIFICATION_WEBHOOK | Slack webhook for sending notifications. Optional -- If not provided the notification steps are skipped. |
| TELEMETRY_URL | URL to inject at build-time for OPA version reporting. Optional -- If not provided the default value in OPA's source is used. |

## Periodic Workflows

Some of the Github Action workflows are triggered on a schedule, and not included in the
post-merge, pull-request, etc actions. These are reserved for time consuming or potentially
non-deterministic jobs (race detection tests, fuzzing, etc).

Below is a list of workflows and links to their status:

| Workflow | Description |
|----------|-------------|
| [![Nightly](https://github.com/open-policy-agent/opa/workflows/Nightly/badge.svg?branch=main)](https://github.com/open-policy-agent/opa/actions?query=workflow%3A"Nightly") | Runs once per day at 8:00 UTC. |

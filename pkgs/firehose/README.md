[![pub package](https://img.shields.io/pub/v/firehose.svg)](https://pub.dev/packages/firehose)
[![package publisher](https://img.shields.io/pub/publisher/firehose.svg)](https://pub.dev/packages/firehose/publisher)

## What's this?

This is a tool to automate publishing of pub packages from GitHub actions.

## Conventions and setup

When run from a PR, this tool will validate the package pubspecs and
changelogs and indicate whether the criteria for publishing has been met.
Generally, each PR should add a new entry to the changelog, rev the pubspec
version, and the changelog version and pubspec version should agree.

When run in response to a git tag event (a tag with a pattern like `v1.2.3` or
`name_v1.2.3` for monorepos), this tool will publish the indicated package.

## Pre-release versions

Pre-release versions (aka, `'1.2.3-dev'`) are handled specially; this tool will
validate the package but will not auto-publish it. This can be used to
accumulate several changes and later publish them as a group.

## Disabling auto-publishing

In order to disable package validation and auto-publishing, add the
`publish_to: none` key to your pubspec.

## PR branch actions

For PRs, this tool:

- determines repo packages
- validates that the changelog version equals the pubspec version
- performs a `dart pub publish --dry-run`

## Git tag actions

In response to a git tag event, this tool:

- validates the tag is well-formed
- determines the indicated package
- attempts to publish that package (`dart pub publish --force`)

## Mono-repos

This tool can work with either single package repos or with mono-repos (repos
containing several packages). It will scan for and detect packages in a mono
repo; to omit packages from validation and auto-publishing, add a
`publish_to: none` key to its pubspec.

For single package repos, the tag pattern should be `v1.2.3`. For mono-repos,
the tag pattern must be prefixed with the package name, e.g. `foo-v1.2.3`.

## Integrating this tool into a repo

- copy the yaml below into a `.github/workflows/publish.yaml` file in your repo
- update the target branch below if necessary (currently, `main`)
- if publishing from a mono-repo, adjust the 'tags' line below to
  `tags: [ '[A-z]+-v[0-9]+.[0-9]+.[0-9]+' ]`
- from the pub.dev admin page of your package, enable publishing from GitHub
  Actions

```yaml
# A CI configuration to auto-publish pub packages.

name: Publish

on:
  pull_request:
    branches: [ main ]
    types: [opened, synchronize, reopened, labeled, unlabeled]
  push:
    tags: [ 'v[0-9]+.[0-9]+.[0-9]+' ]

jobs:
  publish:
    uses: dart-lang/ecosystem/.github/workflows/publish.yaml@main
```

## Publishing from a specific version of the SDK

Callers may optionally specify the version of the SDK to use when publishing a
package. This can be useful if your package has a very recent minimum SDK
constraint. This is done via the `sdk` input parameter. Note that this parameter
is not required; it defaults to `stable` - using the most recent stable release
of the Dart SDK. To pass this value:

```yaml
jobs:
  publish:
    uses: dart-lang/ecosystem/.github/workflows/publish.yml@main
    with:
      sdk: beta
```

## Workflow docs

The description of the common workflow for repos using this tool can be found at
https://github.com/dart-lang/ecosystem/wiki/Publishing-automation.

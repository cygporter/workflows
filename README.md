# cygporter workflows

This repository contains [reusable workflows][] for creating builds of Cygwin
packages using [cygport][].

## Usage

For all these reusable workflows, set them up as jobs in your GitHub Actions
workflows.

### Build and test

This workflow runs the cygport download, prep, compile, test, install and
package steps.  It will store the build results as both an artefact (for
downloading and analysis, and use in other jobs within the same workflow) and a
cache (for use in other workflows)

The workflow parses the cygport file to get the build dependencies.  Some
cygclass cygport includes will cause parsing the cygport file to fail if
they're not installed already; if you're using one of those, you'll need to
define `bootstrap_packages` to specify what additional packages to install for
the cygport file to be parsable.

```yaml
uses: cygporter/workflows/.github/workflows/build-test.yml@v1
with:
  # Required: Path to the cygport file that you want to build and test.
  cygport_file: <path>

  # Optional: Space-separated list of packages required to parse the cygport
  # file.  Defaults to "cygport".
  bootstrap_packages: <list>
```

The workflow creates the following outputs:

-   `pv`: the value of the [`$PV`][PV] variable per `cygport vars`.
-   `cache-found`: whether the build stage was skipped because an up-to-date
    build cache was available.

### Prepare release

This workflow creates a release on GitHub.  By default, the release isn't
publicly visible until you manually publish it, and this workflow does not do
anything to push the release to the Cygwin mirrors.

This workflow expects to find matching build output in the cache, so you will
need to have recently run the `build-test.yml` workflow on the same branch and
for the same commit before you can run this one.

```yaml
uses: cygporter/workflows/.github/workflows/prep-release.yml@v1
with:
  # Required: Path to the cygport file that you want to release from.
  cygport_file: <path>

  # Optional: Name to use for the release.  Defaults to v$PVR from the cygport
  # file (e.g. if your cygport file contains VERSION=2.38.2 and RELEASE=1, the
  # tag will default to "v2.38.2-1"), and if specified, the tag must be
  # prefixed by that (e.g.  "v2.38.2-1-rc1" is allowed, but "v2.39.0-1" or
  # "parrot" would be blocked).
  release_tag: <tag-name>

  # Optional: Space-separated list of packages required to parse the cygport
  # file.  Defaults to "cygport".
  bootstrap_packages: <list>

  # Optional: Whether to publish the release on GitHub.  Defaults to "false".
  publish: (true|false)

# Needed to be able to create the release.
permissions:
  contents: write
```

The workflow outputs the tag that was used for the release as `release_tag`.
This will be the same as the value passed in as `release_tag` if one was passed
in, or `v$PVR` if not.

### Release

This workflow pushes the release to the Cygwin mirrors, and also pushes the
release tag to the Cygwin Git repositories, to aid discoverability.

This workflow expects to be called after the `prep-release.yml` workflow.

```yaml
uses: cygporter/workflows/.github/workflows/release.yml@v1
with:
  # Required: Path to the cygport file that you want to release from.
  cygport_file: <path>

  # Required: Name of the tag used in the GitHub release.  This release must be
  # published, and will be used to get the correct build to upload to the
  # Cygwin mirrors.
  tag_name: <tag-name>

  # Optional: Space-separated list of packages required to parse the cygport
  # file.  Defaults to "cygport".
  bootstrap_packages: <list>

secrets:
  # Required: The RSA private key to connect to the Cygwin servers, as would
  # normally be stored at ~/.ssh/id_rsa.
  maintainer_key: <string>
```

## Versions, branches and tags

This repository uses [semver][] for version numbers.

Tags will only specify version numbers, and should never change.  This is
different to the behaviour for many GitHub official actions, which also use
semver, but where the latest release of any given version is specified by a
_moving_ tag, e.g. [actions/checkout@v4][] is a tag pointing to the latest
v4.x.x release of actions/checkout.

Branches are as follows:

-   `main` points to the code that's expected to be in the next release.
-   `v*` points to the latest release within a major or minor release sequence.
-   `maint/v*` points to the next release within a major or minor release
    sequence.
-   All other branches are for work in development.

The `main`, `v*` and `maint/v*` branches should only move forward; work on
branches in progress may sometimes be rebased or otherwise changed and are not
expected to have a persistent history.

Bug fixes are, where possible, made in a single commit on a branch taken from
the commit that introduced the bug, as suggested in ["git bugfix branches:
choose the root wisely"][gcbenison], and then merged wherever they need
merging.  Possibly this is a terrible idea -- it's definitely not one that
seems to have caught on -- but I like it in theory and want to give it a fair
trial.

[reusable workflows]: https://docs.github.com/en/actions/using-workflows/reusing-workflows
[cygport]: https://cygwin.github.io/cygport/
[PV]: https://cygwin.github.io/cygport/syntax_cygpart.html#PV
[semver]: https://semver.org/spec/v2.0.0.html
[actions/checkout@v4]: https://github.com/actions/checkout/tree/v4
[gcbenison]: https://gcbenison.wordpress.com/2012/01/17/git-bugfix-branches-choose-the-root-wisely/

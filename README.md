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
uses: cygporter/workflows/.github/workflows/ci.yml@main
with:
  # Required: Path to the cygport file that you want to build and test.
  cygport_file: <path>

  # Optional: Space-separated list of packages required to parse the cygport
  # file.  Defaults to "cygport".
  bootstrap_packages: <list>
```

### Prepare release

This workflow creates a draft release on GitHub.  The release isn't publicly
visible until you manually publish it, and this workflow does not do anything
to push the release to the Cygwin mirrors.

This workflow expects to find matching build output in the cache, so you will
need to have recently run the `ci.yml` workflow on the same branch and for the
same commit before you can run this one.

```yaml
uses: cygporter/workflows/.github/workflows/prep-release.yml@main
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
```

[reusable workflows]: https://docs.github.com/en/actions/using-workflows/reusing-workflows
[cygport]: https://cygwin.github.io/cygport/
[cygporter/action]: https://github.com/cygporter/action
[actions]: https://docs.github.com/en/actions/creating-actions/about-custom-actions

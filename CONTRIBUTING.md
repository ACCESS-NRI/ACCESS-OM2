# Contributing to `ACCESS-OM2`s `spack` Environment

## Via Pull Request

### PRs for New Features

For new features, the workflow is fairly straightforward.

Simply open a pull request from your own branch to `main`, in which you make your modifications to the `spack.yaml` (and optionally the `config/versions.json`). The CI/CD will deploy these changes as a `Prerelease` version of `ACCESS-OM2` on `vk83/prerelease`, so you can verify that the changes will work before being merged formally into `main`.

Some things to be mindful of:

* The `config/versions.json` file is used to bundle the deployment with a particular version of `spack-packages` and `spack-config`. Don't forget to change this if there are cool new features in either of the repositories that you want to incorporate into your deployment.

### PRs for Backported Bugfixes

Since we use the `main` branch as a place for the bleeding-edge `ACCESS-OM2` changes, how do we go about backporting bugfixes?

We use dedicated `backport/*.*` branches for bugfixes and additions to past versions of `ACCESS-OM2`. For example, say we have `2024.01.1` version of `ACCESS-OM2` on `main`, which we need to backport a bugfix.

We should:

* Branch off that commit with a `backport/2024.01` branch (if it doesn't already exist)
* Open a PR off the `backport/2024.01` branch with the fixes, and when it is merged, will be tagged with `2024.01.2` on the `backport/2024.01` branch.

We need to be mindful of the same things as merges of features into `main`:

* The `config/versions.json` file is used to bundle the deployment with a particular version of `spack-packages` and `spack-config`. Don't forget to change this if there are cool new features in either of the repositories that you want to incorporate into your deployment.

> [!NOTE]
> For bugfixes, changing the versions for `spack-config` or `spack-packages` could have a large effect on the deployed model. Make sure this kind of fix is required.

### More on the Prerelease Environment

The `Prerelease` Environment is an ephemeral, completely separated place in `vk83` where you can test the changes you have made. Each commit in the PR is deployed to this environment, where you can test between versions in the PR, as well.

These `spack env`s are of the form `access-om2-VERSION-BUILD`, such as `access-om2-2024_02_1-3` for the 3rd commit in the PR for `access-om2` version `2024.02.1`.

These ephemeral `spack env`s are removed upon closing of the pull request, where the proper deployment is then done if the PR is merged.

<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one
  ~ or more contributor license agreements.  See the NOTICE file
  ~ distributed with this work for additional information
  ~ regarding copyright ownership.  The ASF licenses this file
  ~ to you under the Apache License, Version 2.0 (the
  ~ "License"); you may not use this file except in compliance
  ~ with the License.  You may obtain a copy of the License at
  ~
  ~   http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing,
  ~ software distributed under the License is distributed on an
  ~ "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  ~ KIND, either express or implied.  See the License for the
  ~ specific language governing permissions and limitations
  ~ under the License.
  -->

# Release scripts

The scripts in this directory run the Parquet-Java release. GitHub Actions
workflows run them; you can also run them on your own machine.

For the full release process, including the votes and the manual steps, see
[Releasing Parquet-Java](https://parquet.apache.org/docs/contribution-guidelines/releasing/).

| Workflow | Script | Purpose |
| --- | --- | --- |
| [Release - Prepare RC](../.github/workflows/release-prepare-rc.yml) | `bin/prepare-rc.sh` | Tag an RC, stage it in Nexus and `dist/dev`, create the GitHub pre-release |
| [Release - Cancel RC](../.github/workflows/release-cancel-rc.yml) | `bin/cancel-rc.sh` | Drop the Nexus staging repository and remove the RC from `dist/dev` |
| [Release - Publish After Vote](../.github/workflows/release-publish.yml) | `bin/publish-release.sh` | Tag the release, release Nexus, create the GitHub release, set the next development version |

The workflows run only in `apache/parquet-java`. Each one writes its results,
the next steps, and any e-mail text to the run's **Summary** page.

Publish does not change `dist/release`: the workflow credentials can write
only to `dist/dev`. Its summary has the `svn` commands that a PMC member runs
to move the release and remove the releases it replaces.

## Running a script locally

Run the scripts from the repository root. `--help` shows the arguments of each
script.

The scripts run in dry-run mode unless `DRY_RUN=0` is set. In dry-run mode
they print each command that changes something, and do not run it:

```sh
DRY_RUN=1 ./release/bin/prepare-rc.sh 1.18.0
DRY_RUN=1 ./release/bin/cancel-rc.sh 1.18.0 0 orgapacheparquet-1234
DRY_RUN=1 ./release/bin/publish-release.sh 1.18.0 orgapacheparquet-1234
```

### `bin/prepare-rc.sh`

```
prepare-rc.sh <version> [--rc <num>] [--skip-branch-creation]
```

| Argument | Workflow input | Description |
| --- | --- | --- |
| `<version>` | `version` | The release version, for example `1.18.0`. |
| `--rc <num>` | `rc_number` | Use this RC number instead of the next free one. The script stops if the RC tag already exists. |
| `--skip-branch-creation` | (none) | Do not create the `parquet-X.Y.x` release branch when it does not exist. |

### `bin/cancel-rc.sh`

```
cancel-rc.sh <version> <rc-num> <staging-repo-id> [--allow-description-mismatch]
```

| Argument | Workflow input | Description |
| --- | --- | --- |
| `<version>` | `version` | The release version, for example `1.18.0`. |
| `<rc-num>` | `rc_number` | The RC number that failed the vote, for example `0`. |
| `<staging-repo-id>` | `staging_repo_id` | The Nexus staging repository to drop, for example `orgapacheparquet-1234`. |
| `--allow-description-mismatch` | `allow_description_mismatch` | Skip the check that the staging repository description is `Apache Parquet <version> RC<num>`. For recovery only. |

### `bin/publish-release.sh`

```
publish-release.sh <version> <staging-repo-id> [--rc <num>] [--allow-description-mismatch]
```

| Argument | Workflow input | Description |
| --- | --- | --- |
| `<version>` | `version` | The release version, for example `1.18.0`. |
| `<staging-repo-id>` | `staging_repo_id` | The Nexus staging repository to release, for example `orgapacheparquet-1234`. |
| `--rc <num>` | `rc_number` | The RC number that passed the vote. Default: the latest RC. It must be the latest RC for the version. |
| `--allow-description-mismatch` | `allow_description_mismatch` | Skip the check that the staging repository description is `Apache Parquet <version> RC<num>`. For recovery only. |

The workflow `dry_run` input sets `DRY_RUN`: selected is `DRY_RUN=1`, cleared
is `DRY_RUN=0`.

### Environment variables

For a real run, set these environment variables:

| Variable | Used by | Purpose |
| --- | --- | --- |
| `NEXUS_USERNAME`, `NEXUS_PASSWORD` | all | Deploy to, release, and drop Nexus staging repositories |
| `SVN_USERNAME`, `SVN_PASSWORD` | prepare, cancel | Write to `dist/dev/parquet` |
| `GITHUB_TOKEN` | prepare, publish | Check CI and create GitHub releases |

`prepare-rc.sh` also needs `gpg` with a signing key that is in
[KEYS](https://downloads.apache.org/parquet/KEYS), `svn`, `gh`, and the Thrift
compiler (see `dev/ci-before_install.sh`).

## Layout

- `bin/`: the entry points that the workflows run.
- `libs/`: shared helpers for Nexus, SVN, Maven, GitHub, and version handling.
- `tests/`: [bats](https://github.com/bats-core/bats-core) tests for the helpers.

## Tests

```sh
bats release/tests/*.bats
```

The [Test Release Scripts](../.github/workflows/ci-release-scripts.yml)
workflow runs these tests on each pull request and each push to `master` that
changes `release/`.

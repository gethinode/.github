# gethinode/.github — shared CI

Reusable GitHub Actions CI for the `gethinode` Hugo module ecosystem (~27 repos). Downstream repos call the workflows here instead of copy-pasting CI YAML.

The org profile page lives at [profile/README.md](profile/README.md) and is unaffected by this document.

## Pinning contract

Callers **must** pin to a tag, not a branch or SHA-less ref:

```yaml
uses: gethinode/.github/.github/workflows/test.yml@v1
```

`@v1` is a moving major tag updated on release. Breaking changes to inputs/secrets bump the major and are announced before the tag moves.

## `setup-hinode` composite action

`actions/setup-hinode@v1` installs Go, Node.js, and the chosen package manager, then runs a frozen/deterministic install. This is the single place the pnpm-default/npm-opt-in choice lives — workflows should not reimplement it.

| Input | Default | Description |
| --- | --- | --- |
| `package-manager` | `pnpm` | `pnpm` (default) or `npm`. |
| `node-version` | `22.x` | Node.js version passed to `actions/setup-node`. |
| `go-version` | `>1.0.0` | Go version passed to `actions/setup-go`. |

Behavior:

- `pnpm` (default): enables Corepack, then runs `pnpm install --frozen-lockfile`.
- `npm`: runs `npm install`.

## Reusable workflows

### `test.yml`

Matrix test build across OS and Node versions using `setup-hinode`, then `<package-manager> run test`.

| Input | Default | Description |
| --- | --- | --- |
| `package-manager` | `pnpm` | Package manager to use. |
| `os-matrix` | `["ubuntu-latest","macos-latest","windows-latest"]` | JSON array of runner images. |
| `node-matrix` | `["22.x","24.x"]` | JSON array of Node.js versions. |

No secrets required. `permissions: contents: read`.

### `release.yml`

Runs `semantic-release` via `pnpm exec semantic-release` after `setup-hinode`.

| Input | Default | Description |
| --- | --- | --- |
| `package-manager` | `pnpm` | Package manager to use. |

| Secret | Required | Description |
| --- | --- | --- |
| `SEMANTIC_RELEASE_GIT` | yes | Token used as `GITHUB_TOKEN` for release/tag/push operations. |
| `NPM_TOKEN` | no | Published to npm registries that require auth. |

`permissions: contents: read` (the token comes from the secret, not the default `GITHUB_TOKEN`).

### `mod-update.yml`

Runs `<package-manager> run mod:update` to refresh Hugo module dependencies, then opens a pull request via `gethinode-actions/create-pull-request@v8`.

| Input | Default | Description |
| --- | --- | --- |
| `package-manager` | `pnpm` | Package manager to use. |

| Secret | Required | Description |
| --- | --- | --- |
| `SEMANTIC_RELEASE_GIT` | yes | Token used to open the update pull request. |

`permissions: contents: write`, `pull-requests: write`.

### `npm-compat.yml`

Verifies the repo builds for npm-only consumers: a clean `npm install` with no lockfile (the real npm-consumer path, deliberately bypassing `setup-hinode`), followed by `npm run <build-script>`.

| Input | Default | Description |
| --- | --- | --- |
| `build-script` | `build:example` | npm script to run after install. |

No secrets required. `permissions: contents: read`.

## Calling from a downstream repo

```yaml
jobs:
  test:
    uses: gethinode/.github/.github/workflows/test.yml@v1
    with:
      package-manager: pnpm
    secrets: inherit
```

## Not yet included

`auto-merge.yml` and `lint-build.yml` are planned as a follow-up and are not part of this foundation.

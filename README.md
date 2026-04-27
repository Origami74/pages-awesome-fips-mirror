# pages-awesome-fips-mirror

A GitHub Pages mirror of an [nsite](https://nsyte.run/), plus the reusable
GitHub Action that does the work — pulls a Nostr nsite from its relays +
Blossom servers and publishes it to Pages.

This repo serves two purposes:

1. **Hosts the [`nsite-deploy-pages`](action.yml) action** — a composite
   action that any repo can call to mirror an nsite to its own Pages site.
2. **Mirrors awesome.fips.network itself** via
   [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml), which
   calls the action on a 6-hourly schedule.

## Using the action in another repo

```yaml
# .github/workflows/deploy.yml in <your-repo>
name: Mirror nsite to Pages

on:
  schedule: [{ cron: "0 */6 * * *" }]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  mirror:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.mirror.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4
      - id: mirror
        uses: <owner>/pages-awesome-fips-mirror@v1
        with:
          pubkey: npub1y0gja7r4re0wyelmvdqa03qmjs62rwvcd8szzt4nf4t2hd43969qj000ly
          # name: my-named-site   # optional, for kind-35128 named nsites
```

### Inputs

| input        | required | description                                                          |
| ------------ | -------- | -------------------------------------------------------------------- |
| `pubkey`     | yes      | npub, hex pubkey, or NIP-05 (e.g. `name@host`) of the nsite publisher |
| `name`       | no       | name of a *named* nsite (kind 35128); blank → root site (kind 15128) |
| `output-dir` | no       | local directory the nsite is downloaded into (default: `site`)       |
| `relays`     | no       | comma-separated relay overrides for the download                     |
| `servers`    | no       | comma-separated Blossom server overrides for the download            |

### Outputs

| output       | description                                                   |
| ------------ | ------------------------------------------------------------- |
| `page_url`   | URL of the published Pages site                               |
| `file_count` | number of files downloaded from the nsite                     |

### Requirements on the calling workflow

- A runner with `curl` + `bash` (any `ubuntu-latest` works)
- `pages: write` and `id-token: write` permissions
- The `github-pages` environment on the deploying job
- A `actions/checkout@v4` step before calling the action

The action handles the rest: install `nsyte`, download the nsite, fail if
the download is empty, then `actions/configure-pages` →
`actions/upload-pages-artifact` → `actions/deploy-pages`.

## Mirroring this repo (awesome.fips.network)

The FIPS npub is hardcoded in [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml),
so setup is just:

1. **Settings → Pages**: set Source to **GitHub Actions**.
2. **Actions tab → Mirror nsite to Pages → Run workflow** for the first run.

After that, the 6-hourly cron keeps it fresh. The **Run workflow** dialog
accepts one-off `pubkey` / `name` overrides for ad-hoc mirrors of other
nsites without editing the workflow.

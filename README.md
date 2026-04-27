# pages-awesome-fips-mirror

A GitHub Pages mirror of [awesome.fips.network](https://awesome.fips.network)
— the nsite, pulled from Nostr + Blossom on a 6-hourly schedule and
republished as a regular static site for users without an nsite resolver.

The actual mirroring is done by the
[`deploy-nsite-to-pages`](https://github.com/Origami74/deploy-nsite-to-pages) action;
this repo just wires it up with the FIPS publisher's pubkey.

## Setup

1. **Settings → Pages**: set Source to **GitHub Actions**.
2. **Actions tab → Mirror nsite to Pages → Run workflow** for the first run.

After that, the cron in [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml)
keeps it fresh. The **Run workflow** dialog accepts one-off `pubkey` /
`name` overrides for ad-hoc mirrors of other nsites without editing the
workflow.

## Configuration

The FIPS-specific defaults live in
[`.github/workflows/deploy.yml`](.github/workflows/deploy.yml) as `env:`
entries:

- `AWESOME_FIPS_NPUB` — the publisher
- `AWESOME_FIPS_NAME` — `awesome` (the named-nsite identifier)
- `AWESOME_FIPS_SERVERS` — Blossom servers used by the publisher; remove
  this once a kind-10063 server-list event is gossiped widely enough that
  `nsyte download` can discover them automatically

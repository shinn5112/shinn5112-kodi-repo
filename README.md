# shinn5112 Kodi Repository

A Kodi add-on repository, generated and published automatically by GitHub Actions. Requires Kodi 20 (Nexus) or newer.

## Add-ons

### Crunchyroll (`plugin.video.crunchyroll`)

Source: [shinn5112/plugin.video.crunchyroll](https://github.com/shinn5112/plugin.video.crunchyroll)

This is a fork of [smirgol/plugin.video.crunchyroll](https://github.com/smirgol/plugin.video.crunchyroll) that mirrors its release cycle: when upstream publishes a stable release, the fork publishes the same version, so the version numbers here match upstream's.

The fork adds [Up Next](https://kodi.wiki/view/Add-on:Up_Next) integration, which shows a next-episode prompt across seasons. It was submitted upstream in [pull request #142](https://github.com/smirgol/plugin.video.crunchyroll/pull/142), but the upstream maintainer declined it, so it is only available in this fork.

### HDMI CEC Power Controls (`script.module.hdmi-cec-power-controls`)

Source: [shinn5112/kodi-hdmi-cec-power-controls](https://github.com/shinn5112/kodi-hdmi-cec-power-controls)

Lets you turn a TV on or off over HDMI CEC by calling the add-on through Kodi's JSON-RPC API.

## Install

1. Download [repository.shinn5112-1.0.0.zip](https://raw.githubusercontent.com/shinn5112/shinn5112-kodi-repo/main/repository.shinn5112/repository.shinn5112-1.0.0.zip).
2. In Kodi: **Add-ons -> Install from zip file** and select the zip (enable installs from unknown sources if prompted).
3. **Add-ons -> Install from repository -> shinn5112 Kodi Repository**, then pick an add-on.

## How it works

- [build-repository.yml](.github/workflows/build-repository.yml) runs `scripts/create_repository.py` on every `addon-release` dispatch and on manual runs, then commits the result to `main`.
- Add-on repositories send the dispatch when they publish a stable release. They need a `KODI_REPO_TOKEN` secret: a fine-grained personal access token with **Contents: read and write** on this repository.
- `scripts/create_repository.py` also copies each add-on's icon, fanart and screenshots (as declared in its `<assets>`) next to the generated zip so Kodi can show them.
- `repository.shinn5112` is built from [src/repository.shinn5112](src/repository.shinn5112).
- Everything outside `src/`, `scripts/`, `docs/` and `.github/` is generated output. Do not edit it by hand.

## Configuration

The add-ons to publish come from the `ADDON_SOURCES` Actions variable (Settings -> Secrets and variables -> Actions -> Variables), one per line:

```
https://github.com/shinn5112/plugin.video.crunchyroll.git#main
https://github.com/shinn5112/kodi-hdmi-cec-power-controls.git#master
```

Each entry is `REPOSITORY_URL#BRANCH_OR_TAG:PATH`, where the branch and path are optional. If the variable is unset, the two add-ons above are used. Setting it replaces that list, so include every add-on you want published.

Add-on zips are built with `git archive`, so an add-on repository can keep development-only files out of its zip with `export-ignore` rules in a `.gitattributes` file.

## Adding an add-on

1. Add its Git URL to `ADDON_SOURCES`.
2. Optionally add a workflow to that repository that sends an `addon-release` `repository_dispatch` to this one on release (see the notify workflows in the existing add-ons).
3. Run **Build Kodi repository** from the Actions tab.

More detail is in [docs/steering](docs/steering).

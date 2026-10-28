# Kodi Add-on Repository Steering Guide

Last reviewed: 2026-10-27
Primary source: https://kodi.wiki/view/Add-on_repositories

## Purpose

Use this guide when changing how this repository (`shinn5112-kodi-repo`) generates, lays out, or serves Kodi add-ons. The repository is static content served over HTTPS (GitHub raw/Pages) and built by `scripts/create_repository.py` through GitHub Actions.

## 1) Required Ingredients

A Kodi repository needs:

- An HTTP server (any). Here: GitHub-hosted static files.
- Add-on zips.
- A master XML file (`addons.xml`) describing every add-on.
- A checksum file for the master XML (`addons.xml.md5`).
- A repository add-on, which users install so Kodi knows where the repository lives.

Steering rule:
- All five must exist and be consistent. A missing repository add-on means users cannot install from this repository at all.

## 2) Directory Layout

```
<datadir>/
  addons.xml
  addons.xml.md5
  <addon.id>/
    <addon.id>-<x.y.z>.zip
    <addon.id>-<x.y.z>.zip.md5
    addon.xml
    icon.png
    fanart.jpg
    changelog-<x.y.z>.txt
    (other files referenced by the add-on's <assets>)
```

Rules:
- The directory name must equal the add-on `id`, and the id must be unique.
- `x.y.z` in the zip name must equal the `version` attribute in that add-on's `addon.xml`.
- The zip path Kodi requests is `<datadir>/<addon.id>/<addon.id>-<x.y.z>.zip`.
- Each add-on directory also carries its icon, fanart, changelog and any `<assets>` files.
- Zip contents must have a single top-level folder named after the add-on id.

## 3) addons.xml

- A single `<addons>` root wrapping each add-on's full `addon.xml` element.
- UTF-8 with an XML declaration.
- Regenerate it every time any add-on version changes. Never hand-edit it.
- It may be pre-gzipped as `addons.xml.gz`. The `<info>` URL must then end in `.gz`.

## 4) Checksum File

- `addons.xml.md5` identifies the content of `addons.xml`.
- Kodi fetches it first and only downloads `addons.xml` if the value changed. It is not verified as a hash, but it MUST change whenever `addons.xml` changes.
- Always regenerate it together with `addons.xml` (the script does this).

## 5) Repository Add-on

Distributed like any other add-on, with an `xbmc.addon.repository` extension:

```xml
<extension point="xbmc.addon.repository" name="Example Repository">
  <dir>
    <info compressed="false">https://example.com/addons.xml</info>
    <checksum>https://example.com/addons.xml.md5</checksum>
    <datadir zip="true">https://example.com/</datadir>
    <hashes>sha256</hashes>
  </dir>
</extension>
```

Element rules:
- `<dir>`: wraps one repository location. Multiple `<dir>` elements are allowed. Use `minversion` / `maxversion` (maxversion since Kodi 19) to serve different directories to different Kodi versions.
- Do not put `<info>`/`<checksum>`/`<datadir>` directly under the extension; that form was removed in Kodi 20.
- `<info>`: URL of `addons.xml`. Kodi requests gzip encoding; a pre-gzipped file must end in `.gz`.
- `<checksum>`: URL of the checksum file.
- `<datadir>`: root URL containing the `<addon.id>/` directories. Keep `zip="true"`; the `zip` attribute was dropped in Kodi 17 and zips are the only supported form for online repositories.
- `<hashes>`: `sha256`, `sha512`, `sha1`, `md5`, or `false`. Kodi sends a HEAD request and compares the `content-<hash>` HTTP header to its local copy to skip unchanged downloads. Only set it if the host actually sends that header; otherwise use `false`.
- The repository add-on is itself an add-on: it needs its own id (for example `repository.shinn5112`), version, and zip entry in this repository, and it must appear in `addons.xml`.

## 6) Security

- Serve everything over HTTPS only. Kodi logs a warning for non-HTTPS URIs, and plain HTTP lets anyone serve malicious add-ons to users.
- Keep zips verifiable (valid archives). Do not host unzipped add-on folders.
- Treat the add-on sources listed in `ADDON_SOURCES` as trusted code: anything they contain is published to users.

## 7) Tooling: scripts/create_repository.py

- Accepts local folders, local zips, or Git URLs formatted as `REPOSITORY_URL#BRANCH_OR_TAG:PATH`.
- Options: `--datadir` (output directory), `--compressed` (gzip `addons.xml`), `--no-parallel` (serial, easier diagnostics).
- Requires `gitpython` for Git URL sources.
- It writes the zip, checksum, and copied metadata files per add-on, then `addons.xml` and `addons.xml.md5`.
- It builds the head of the given branch, not a release tag, unless a `#tag` is given.
- The script is third-party (GPLv2, Chad Parry) with one local modification: `copy_asset_files` (marked "Local modification"). Keep further changes minimal so upstream updates stay easy to merge.

## 8) Automation in This Repository

- [build-repository.yml](../../.github/workflows/build-repository.yml) runs on `repository_dispatch` (`addon-release`) and manually.
- Sources come from the `ADDON_SOURCES` Actions variable (whitespace/newline-separated list). If unset, it defaults to the Crunchyroll plugin and the HDMI CEC add-on on `main`.
- `scripts/create_repository.py` copies each add-on's `<assets>` files (icon, fanart, screenshots) to the same relative path under `<addon.id>/`. The upstream script only copies root-level `icon.png`/`fanart.jpg`.
- Add-on zips come from `git archive`, so source repositories exclude dev-only files with `export-ignore` in `.gitattributes`.
- The repository add-on source lives in `src/repository.shinn5112/` (kept out of the root so it does not collide with the generated `repository.shinn5112/` output directory). The workflow always builds it alongside `ADDON_SOURCES`. Its URLs point at `raw.githubusercontent.com/shinn5112/shinn5112-kodi-repo/main/`, which does not send `content-<hash>` headers, so `<hashes>` is `false`.
- Changes are committed to `main` by `github-actions[bot]`; commits only happen when output changed.
- Generated files (`addons.xml`, `addons.xml.md5`, add-on directories) are build output. Do not edit them by hand; they are overwritten on the next run.

## 9) Decision Gates for Changes

1. Adding an add-on: add its Git URL to `ADDON_SOURCES`; confirm its `addon.xml` id and version are valid and the directory name matches the id.
2. Changing hosting URL: update the repository add-on's `<info>`, `<checksum>`, `<datadir>`, bump its version, and republish it. Existing users only move over after updating the repository add-on.
3. Supporting multiple Kodi versions: use separate `<dir minversion/maxversion>` entries pointing to separate datadirs, each generated with its own `create_repository.py` run.
4. Enabling `<hashes>`: first verify the host returns `content-<hash>` headers.

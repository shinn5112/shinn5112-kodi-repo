# Repository Publish Checklist

Use this before and after a repository rebuild. Background: [kodi-repository-steering.md](./kodi-repository-steering.md).

## A. Inputs

- [ ] `ADDON_SOURCES` (or the default) lists only trusted `https://` Git URLs.
- [ ] Each source's `addon.xml` is valid XML with `id`, `name`, `version`, `provider-name`.
- [ ] Each add-on version was bumped upstream for the change being published.

## B. Generated Output

- [ ] `addons.xml` starts with an XML declaration and wraps every add-on in `<addons>`.
- [ ] `addons.xml.md5` changed along with `addons.xml`.
- [ ] Each `<addon.id>/` directory contains `<addon.id>-<x.y.z>.zip`, its `.md5`, `addon.xml`, `icon.png`, `fanart.jpg` and `changelog-<x.y.z>.txt`.
- [ ] Zip filename version equals the `version` in `addon.xml` and in `addons.xml`.
- [ ] Zip has a single top-level folder named after the add-on id.

## C. Repository Add-on

- [ ] A repository add-on exists, is listed in `addons.xml`, and has a zip.
- [ ] Its extension uses `<dir>` wrappers (not the removed Kodi 20 root-level form).
- [ ] `<info>`, `<checksum>`, `<datadir>` are `https://` URLs that resolve to the published files.
- [ ] `<datadir zip="true">` is set; `<hashes>` is `false` unless the host sends `content-<hash>` headers.

## D. Verification

- [ ] The workflow run succeeded and pushed a commit to `main` (or correctly reported no changes).
- [ ] The raw URLs for `addons.xml` and `addons.xml.md5` return the new content.
- [ ] In Kodi: install or update the repository add-on, then confirm the new add-on version appears under Install from repository.

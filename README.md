# LuCI Package Manager — Untrusted Local APK Support

An unofficial, security-sensitive patch for OpenWrt's stock
`luci-app-package-manager`. It adds an opt-in setting for installing uploaded
local APK packages that are not trusted by `apk`.

This is not a separate LuCI application and does not patch system files from a
package maintainer script. The released APK is a rebuilt
`luci-app-package-manager` from the OpenWrt LuCI source tree.

> **Target platform:** OpenWrt 25.12 with the APK package manager.

## Release scope

| Item | Value |
| --- | --- |
| OpenWrt release line | 25.12 |
| Upstream base | [`067535eaf51a59582b775a8b588a9b05810f8030`](https://github.com/openwrt/luci/commit/067535eaf51a59582b775a8b588a9b05810f8030) |
| Patched source | [`e1fb46d3ecae5ede2eabcfe1792697ac734bb07a`](https://github.com/RzandAl/luci/commit/e1fb46d3ecae5ede2eabcfe1792697ac734bb07a) |
| Release tag | `openwrt-25.12-067535e-r2` |
| APK package version | `26.278.01645~e1fb46d` |
| Release APK asset | `luci-app-package-manager-openwrt-25.12-067535e-r2.apk` |
| Package architecture | `noarch` |

## What the patch changes

- Adds **Allow untrusted local packages** to **Configure APK**.
- Stores the setting as
  `luci.package_manager.allow_untrusted_uploads` in UCI.
- Keeps the setting disabled by default.
- Shows the current **Blocked** or **Allowed** state on the Software page.
- Keeps the existing confirmation dialog for every uploaded package.
- Reloads the persisted UCI value if saving the setting fails, so the displayed
  state cannot remain out of sync with the backend.
- Adds `--allow-untrusted` and `--force-non-repository` in the backend only
  when all of the following are true:
  - the package manager is `apk`;
  - the requested action is `install`;
  - exactly one package argument was supplied;
  - that argument is exactly `/tmp/upload.apk`;
  - the UCI setting is enabled.
- Prevents package names and paths from being interpreted as `apk` options by
  inserting the `--` option delimiter.
- Does not allow the frontend to supply arbitrary trust flags.

## Compatibility

| Area | Target or verified environment |
| --- | --- |
| Target | OpenWrt 25.12 with APK |
| Build | Official OpenWrt 25.12.5 SDK for `ramips/mt7621` |
| Runtime — Xiaomi Mi Router 3G | r2 on OpenWrt 25.12.5; r1 baseline on 25.12.2 (`ramips/mt7621`) |
| Runtime — Cudy WR3000S v1 | r1 baseline on OpenWrt 25.12.5 (`mediatek/filogic`) |
| Browser UI | Firefox, Chrome, and Microsoft Edge |
| Validation | Blocked by default; explicit opt-in and per-upload confirmation required |

The released APK is architecture-independent (`noarch`). Compatibility outside
the OpenWrt 25.12 release line is not claimed.

## Screenshots

### Blocked by default

The Software page shows that untrusted local APK installation is blocked.

![Blocked untrusted local APK state on the LuCI Software page](docs/screenshots/luci-software-untrusted-blocked.png)

With the setting blocked, the upload confirmation explains that unsigned or
otherwise untrusted APKs require explicit opt-in.

![Upload warning while untrusted local APK installation is blocked](docs/screenshots/luci-untrusted-upload-blocked.png)

### Explicit opt-in

The setting is available under **System → Software → Configure APK** and remains
disabled by default.

![Configure APK dialog with the Allow untrusted local packages option](docs/screenshots/luci-configure-untrusted-apk.png)

### Allowed after opt-in

After the setting is saved, the Software page shows the enabled state with a
yellow warning badge.

![Allowed untrusted local APK state on the LuCI Software page](docs/screenshots/luci-software-untrusted-allowed.png)

Each uploaded APK still requires a separate confirmation.

![Per-upload confirmation while untrusted local APK installation is allowed](docs/screenshots/luci-untrusted-upload-allowed.png)

## Security model

Enabling this option bypasses APK signature trust only for the uploaded local
file and does not silently approve installation. Verify the file's origin and
checksum before enabling the option or confirming an upload.

The backend scope and operations that remain unaffected are documented in
[Security model](docs/SECURITY.md).

## Release packages

Assets for
[`openwrt-25.12-067535e-r2`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/tag/openwrt-25.12-067535e-r2):

- [`luci-app-package-manager-openwrt-25.12-067535e-r2.apk`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/download/openwrt-25.12-067535e-r2/luci-app-package-manager-openwrt-25.12-067535e-r2.apk)
- [`luci-app-package-manager-untrusted-upload-067535e.patch`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/download/openwrt-25.12-067535e-r2/luci-app-package-manager-untrusted-upload-067535e.patch)
- [`SHA256SUMS`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/download/openwrt-25.12-067535e-r2/SHA256SUMS)

The APK is intentionally untrusted. Verify `SHA256SUMS` and perform the first
installation from a terminal; see [Installation and rollback](docs/INSTALLATION.md)
for the complete procedure.

## Documentation

- [Installation, configuration, verification, and rollback](docs/INSTALLATION.md)
- [Security model](docs/SECURITY.md)
- [Release validation](docs/VALIDATION.md)
- [Build and release verification](docs/BUILDING.md)
- [Automated patch validation](.github/workflows/validate.yml)

## Tests and validation

The `r2` release validation covered exact source and patch provenance, clean
standalone patch application, APK metadata and payload inspection, release
checksums, and the complete blocked and allowed runtime paths on OpenWrt 25.12.5.
Earlier r1 device results are retained separately in
[Compatibility](#compatibility) instead of being attributed to the r2 binary.

The complete recorded scope is in [Release validation](docs/VALIDATION.md),
with reproduction commands in [BUILDING.md](docs/BUILDING.md). The repository CI
also repeats clean patch application and syntax validation on every change.

## Source provenance

- Development branch:
  [`fix/package-manager-untrusted-upload-067535e`](https://github.com/RzandAl/luci/tree/fix/package-manager-untrusted-upload-067535e)
- Exact signed commit:
  [`e1fb46d3ecae5ede2eabcfe1792697ac734bb07a`](https://github.com/RzandAl/luci/commit/e1fb46d3ecae5ede2eabcfe1792697ac734bb07a)
- Standalone source patch:
  [`patches/luci-app-package-manager-untrusted-upload-067535e.patch`](patches/luci-app-package-manager-untrusted-upload-067535e.patch)
- Related upstream report:
  [`openwrt/luci#8482`](https://github.com/openwrt/luci/issues/8482)

## Upstream status

The long-term goal is to submit the change to OpenWrt LuCI. Once equivalent
support is available in the official OpenWrt package, this unofficial build
will be marked obsolete and users should return to the repository package.

## Maintainers

Developed and tested together by [AmleyID](https://github.com/AmleyID) and
[RazisID12](https://github.com/RazisID12).

## License

The patch and accompanying documentation are distributed under the
[Apache License 2.0](LICENSE), matching OpenWrt LuCI.

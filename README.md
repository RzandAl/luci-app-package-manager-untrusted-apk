# LuCI Package Manager — Untrusted Local APK Support

An unofficial, security-sensitive patch for OpenWrt's stock
`luci-app-package-manager`. It adds an opt-in setting for installing uploaded
local APK packages that are not trusted by `apk`.

This is not a separate LuCI application and does not patch system files from a
package maintainer script. The released APK is a rebuilt
`luci-app-package-manager` from the OpenWrt LuCI source tree.

> **Target platform:** OpenWrt 25.12 with the APK package manager.

Release **r3** uses a dedicated UCI configuration and passes only
`--allow-untrusted` for explicitly enabled local uploads.

## Release scope

| Item | Value |
| --- | --- |
| OpenWrt release line | 25.12 |
| Upstream base | [`067535eaf51a59582b775a8b588a9b05810f8030`](https://github.com/openwrt/luci/commit/067535eaf51a59582b775a8b588a9b05810f8030) |
| Patched source | [`1d3542d3a5348c96edbc3e155c49a293b1272f3f`](https://github.com/RzandAl/luci/commit/1d3542d3a5348c96edbc3e155c49a293b1272f3f) |
| Release tag | `openwrt-25.12-067535e-r3` |
| APK package version | `26.280.00238~1d3542d` |
| Release APK asset | `luci-app-package-manager-openwrt-25.12-067535e-r3.apk` |
| Package architecture | `noarch` |

## What the patch changes

- Adds **Allow untrusted local packages** to **Configure APK**.
- Stores the setting as
  `luci-package-manager.main.allow_untrusted_uploads` in
  `/etc/config/luci-package-manager`.
- Keeps the setting disabled by default.
- Shows the current **Blocked** or **Allowed** state on the Software page.
- Keeps the existing confirmation dialog for every uploaded package.
- Commits and reverts only the dedicated configuration, preserving unrelated
  shared LuCI values and pending changes.
- Reverts pending dedicated changes and reloads the saved value after a failed
  save, restoring the page status.
- Adds only `--allow-untrusted` in the backend
  when all of the following are true:
  - the package manager is `apk`;
  - the requested action is `install`;
  - exactly one package argument was supplied;
  - that argument is exactly `/tmp/upload.apk`;
  - the dedicated UCI setting is exactly `1`.
- Prevents package names and paths from being interpreted as `apk` options by
  inserting the `--` option delimiter.
- Does not allow the frontend to supply arbitrary trust flags.

## Compatibility

| Area | Target or verified environment |
| --- | --- |
| Target | OpenWrt 25.12 with APK |
| Build | Official OpenWrt 25.12.5 SDK for `ramips/mt7621` |
| r3 runtime | Xiaomi Mi Router 3G, OpenWrt 25.12.5 (`ramips/mt7621`) |
| r3 browser UI | Firefox |
| Earlier r2 coverage | Xiaomi on 25.12.2/25.12.5, Cudy WR3000S v1 on 25.12.5; Firefox, Chrome, and Edge |
| Validation | Dedicated save isolation, failed-save recovery, reboot persistence, uploads, and repository-install simulation |

The released APK is architecture-independent (`noarch`). This metadata does not
establish runtime compatibility on untested devices. The r3 scope is recorded in
[Release validation](docs/VALIDATION.md); earlier results remain in the
[r2 validation archive](docs/VALIDATION-R2.md).

On the first upgrade from r1 or r2, the old shared `luci` option is ignored and
the new setting defaults to **Blocked**. An existing dedicated configuration is
preserved on subsequent upgrades. See [Installation](docs/INSTALLATION.md).

## Screenshots

These images were captured from the r2 interface on OpenWrt 25.12.5. They
illustrate the controls and confirmation flow retained in r3.

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
[`openwrt-25.12-067535e-r3`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/tag/openwrt-25.12-067535e-r3):

- [`luci-app-package-manager-openwrt-25.12-067535e-r3.apk`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/download/openwrt-25.12-067535e-r3/luci-app-package-manager-openwrt-25.12-067535e-r3.apk)
- [`luci-app-package-manager-untrusted-upload-067535e.patch`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/download/openwrt-25.12-067535e-r3/luci-app-package-manager-untrusted-upload-067535e.patch)
- [`SHA256SUMS`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/download/openwrt-25.12-067535e-r3/SHA256SUMS)

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

The r3 APK was built and inspected from the exact signed source commit, then
checked on Xiaomi Mi Router 3G with OpenWrt 25.12.5. Tests covered isolated
saving, real dedicated rollback after an injected commit failure, reboot
persistence, successful opt-in uploads, blocked uploads with APK exit code 99,
and upload cleanup. A repository-install simulation preserved both configuration
files and `/etc/apk/world`.

The complete scope is in [Release validation](docs/VALIDATION.md), with
reproduction commands and hashes in [BUILDING.md](docs/BUILDING.md). CI verifies
the complete patched source tree, syntax, configuration registration, and ACLs.

## Source provenance

- Development branch:
  [`fix/package-manager-untrusted-upload-067535e`](https://github.com/RzandAl/luci/tree/fix/package-manager-untrusted-upload-067535e)
- Exact signed commit:
  [`1d3542d3a5348c96edbc3e155c49a293b1272f3f`](https://github.com/RzandAl/luci/commit/1d3542d3a5348c96edbc3e155c49a293b1272f3f)
- Standalone source patch:
  [`patches/luci-app-package-manager-untrusted-upload-067535e.patch`](patches/luci-app-package-manager-untrusted-upload-067535e.patch)
- Related upstream report:
  [`openwrt/luci#8482`](https://github.com/openwrt/luci/issues/8482)

## Upstream status

The change is proposed in [openwrt/luci#9110](https://github.com/openwrt/luci/pull/9110)
and is awaiting review. It has not been merged into official LuCI.

Once equivalent support is available in the official OpenWrt package, this
unofficial build will be marked obsolete and users should return to the
repository package.

## Maintainers

Developed and tested together by [AmleyID](https://github.com/AmleyID) and
[RazisID12](https://github.com/RazisID12).

## License

The patch and accompanying documentation are distributed under the
[Apache License 2.0](LICENSE), matching OpenWrt LuCI.

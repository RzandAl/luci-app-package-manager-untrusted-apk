# LuCI Package Manager — Untrusted Local APK Support

An unofficial, security-sensitive patch for OpenWrt's stock
`luci-app-package-manager`. It adds an opt-in setting for installing uploaded
local APK packages that are not trusted by `apk`.

This is not a separate LuCI application and does not patch system files from a
package maintainer script. The released APK is a rebuilt
`luci-app-package-manager` from the OpenWrt LuCI source tree.

## Release scope

| Item | Value |
| --- | --- |
| OpenWrt release line | 25.12 |
| Upstream base | [`067535eaf51a59582b775a8b588a9b05810f8030`](https://github.com/openwrt/luci/commit/067535eaf51a59582b775a8b588a9b05810f8030) |
| Patched source | [`4836c113cdbd8c01b4586970d9471541084aaaec`](https://github.com/RzandAl/luci/commit/4836c113cdbd8c01b4586970d9471541084aaaec) |
| Release tag | `openwrt-25.12-067535e-r1` |
| APK package version | `26.272.39633~4836c11` |
| Package architecture | `noarch` |

The release was built with the OpenWrt 25.12.5 SDK for `ramips/mt7621` and
tested on OpenWrt 25.12.2 with apk-tools 3.0.5. Although the APK payload is
architecture-independent, compatibility is only claimed for the OpenWrt 25.12
release line with the corresponding LuCI package manager.

## What the patch changes

- Adds **Allow untrusted local packages** to **Configure APK**.
- Stores the setting as
  `luci.package_manager.allow_untrusted_uploads` in UCI.
- Keeps the setting disabled by default.
- Shows the current **Blocked** or **Allowed** state on the Software page.
- Keeps the existing confirmation dialog for every uploaded package.
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

## Security model

Enabling this option bypasses APK signature trust for the uploaded local file.
Only install APK files whose origin and checksum you have independently
verified.

The setting does not silently approve installations. LuCI still displays a
confirmation dialog for each uploaded package. The extra trust flags are never
applied to package names, URLs, arbitrary filesystem paths, upgrades, removals,
or repository operations.

## Install

Download these assets from the matching GitHub Release:

- `luci-app-package-manager-26.272.39633~4836c11.apk`
- `luci-app-package-manager-untrusted-upload-067535e.patch`
- `SHA256SUMS`

Verify the downloaded files:

```sh
sha256sum -c SHA256SUMS
```

The first installation must be performed in a terminal because the stock LuCI
package manager cannot yet install this untrusted local APK:

```sh
apk add \
    --allow-untrusted \
    --force-non-repository \
    ./luci-app-package-manager-26.272.39633~4836c11.apk
```

After installation, open **System → Software → Configure APK**, enable
**Allow untrusted local packages**, and press **Save**. Every subsequent local
APK upload still requires explicit confirmation.

## Return to the repository package

First update the indexes and determine the package version currently offered by
the configured OpenWrt repositories:

```sh
apk update

REPO_VERSION="$(
    apk query \
        --from repositories \
        --available \
        --fields version \
        luci-app-package-manager |
    sed -n 's/^Version:[[:space:]]*//p' |
    head -n 1
)"

printf 'Repository version: %s\n' "$REPO_VERSION"
```

Simulate the package-only rollback before changing the router:

```sh
apk --simulate add \
    "luci-app-package-manager=$REPO_VERSION"
```

Inspect the transaction. It should replace only `luci-app-package-manager`.
Do not continue if unrelated system packages are listed.

Then install the repository version and remove the temporary exact-version
constraint from APK's world file:

```sh
apk add "luci-app-package-manager=$REPO_VERSION"
apk add luci-app-package-manager
```

The final world constraint should be unversioned:

```sh
grep '^luci-app-package-manager' /etc/apk/world
```

Expected output:

```text
luci-app-package-manager
```

The now-unused setting may optionally be removed:

```sh
uci -q delete luci.package_manager.allow_untrusted_uploads
uci commit luci
```

Do **not** use `apk upgrade --available` as a package-only rollback command.
That option resets package selection more broadly and may schedule unrelated
system upgrades or replacements.

## Source and patch

- Development branch:
  [`fix/package-manager-untrusted-upload-067535e`](https://github.com/RzandAl/luci/tree/fix/package-manager-untrusted-upload-067535e)
- Exact signed commit:
  [`4836c113cdbd8c01b4586970d9471541084aaaec`](https://github.com/RzandAl/luci/commit/4836c113cdbd8c01b4586970d9471541084aaaec)
- Standalone source patch:
  [`patches/luci-app-package-manager-untrusted-upload-067535e.patch`](patches/luci-app-package-manager-untrusted-upload-067535e.patch)
- Related upstream report:
  [`openwrt/luci#8482`](https://github.com/openwrt/luci/issues/8482)

Build instructions are in [BUILDING.md](BUILDING.md).

## Upstream status

The long-term goal is to submit the change to OpenWrt LuCI. Once equivalent
support is available in the official OpenWrt package, this unofficial build
will be marked obsolete and users should return to the repository package.

## Authors

- [RazisID12](https://github.com/RazisID12)
- [AmleyID](https://github.com/AmleyID)

## License

The patch and accompanying documentation are distributed under the
[Apache License 2.0](LICENSE), matching OpenWrt LuCI.

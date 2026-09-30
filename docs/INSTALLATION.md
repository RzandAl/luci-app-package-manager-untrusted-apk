# Installation and rollback

These instructions install the modified `luci-app-package-manager`, enable its
opt-in support for untrusted local APK uploads, and safely return to the package
provided by the configured OpenWrt repositories.

> **Security warning:** enabling untrusted local package installation bypasses
> APK signature trust for the uploaded file. Install only files whose origin
> and checksum you have independently verified.

## Requirements

- OpenWrt 25.12 with the APK package manager.
- Root shell access to the router.
- All three assets from the same GitHub release.

The current release was functionally validated on the devices listed in the
project [Compatibility](../README.md#compatibility) table.

## Download and verify

Download these assets from
[`openwrt-25.12-067535e-r1`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/tag/openwrt-25.12-067535e-r1):

- `luci-app-package-manager-openwrt-25.12-067535e-r1.apk`
- `luci-app-package-manager-untrusted-upload-067535e.patch`
- `SHA256SUMS`

Keep the files in one directory and verify them before installation:

```sh
sha256sum -c SHA256SUMS
```

Both listed files must report `OK`. Do not continue if a checksum fails or the
files came from different release tags.

The published APK uses a GitHub-safe, tag-based filename. Its internal package
version remains `26.272.39633~4836c11`.

## Install

The first installation must be performed in a terminal because the stock LuCI
package manager cannot yet install this untrusted local APK:

```sh
apk add \
    --allow-untrusted \
    --force-non-repository \
    ./luci-app-package-manager-openwrt-25.12-067535e-r1.apk
```

The APK replaces the stock `luci-app-package-manager`; it does not install a
second LuCI application alongside it.

## Enable local untrusted uploads

1. Open **System → Software** in LuCI.
2. Select **Configure APK**.
3. Enable **Allow untrusted local packages**.
4. Press **Save**.

The setting remains disabled by default and is stored as
`luci.package_manager.allow_untrusted_uploads` in UCI. Enabling it does not
silently install anything: every uploaded APK still requires a separate
confirmation.

## Verify behavior

With the setting disabled, the Software page must show **Blocked**, and an
uploaded untrusted APK must not be installable through LuCI.

After explicit opt-in, the page must show **Allowed** with a warning badge. An
uploaded local APK must still display the per-upload confirmation before the
backend starts installation.

See [Security model](SECURITY.md) for the exact backend scope.

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

Do **not** use `apk upgrade --available` as a package-only rollback command.
That option resets package selection more broadly and may schedule unrelated
system upgrades or replacements.

## Remove the optional setting

After returning to the repository package, the now-unused UCI setting may be
removed:

```sh
uci -q delete luci.package_manager.allow_untrusted_uploads
uci commit luci
```

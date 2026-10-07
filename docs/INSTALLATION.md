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

The current release was functionally validated in the environment recorded in
[Release validation](VALIDATION.md). The project
[Compatibility](../README.md#compatibility) table also retains earlier device
baseline results.

## Download and verify

Download these assets from
[`openwrt-25.12-067535e-r3`](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/releases/tag/openwrt-25.12-067535e-r3):

- `luci-app-package-manager-openwrt-25.12-067535e-r3.apk`
- `luci-app-package-manager-untrusted-upload-067535e.patch`
- `SHA256SUMS`

Keep the files in one directory and verify them before installation:

```sh
sha256sum -c SHA256SUMS
```

Both listed files must report `OK`. Do not continue if a checksum fails or the
files came from different release tags.

The published APK uses a GitHub-safe, tag-based filename. Its internal package
version remains `26.280.00238~1d3542d`.

## Install

The first installation must be performed in a terminal because the stock LuCI
package manager cannot yet install this untrusted local APK:

```sh
apk --simulate --allow-untrusted add -- \
    ./luci-app-package-manager-openwrt-25.12-067535e-r3.apk

# After reviewing the simulated transaction:
apk --allow-untrusted add -- \
    ./luci-app-package-manager-openwrt-25.12-067535e-r3.apk
```

The APK replaces the stock `luci-app-package-manager`; it does not install a
second LuCI application alongside it. Review the simulation before installation;
it should replace only this package.

Log out of LuCI, log back in to obtain the new session ACLs, and fully refresh
the Software page (`Ctrl+F5`) before using the new controls.

## Enable local untrusted uploads

1. Open **System → Software** in LuCI.
2. Select **Configure APK**.
3. Enable **Allow untrusted local packages**.
4. Press **Save**.

The setting remains disabled by default and is stored as
`luci-package-manager.main.allow_untrusted_uploads` in
`/etc/config/luci-package-manager`. Enabling it does not
silently install anything: every uploaded APK still requires a separate
confirmation.

## Verify behavior

With the setting disabled, the Software page must show **Blocked**, and an
uploaded untrusted APK must not be installable through LuCI.

After explicit opt-in, the page must show **Allowed** with a warning badge. An
uploaded local APK must still display the per-upload confirmation before the
backend starts installation.

Check the persisted value through SSH:

```sh
uci -q get luci-package-manager.main.allow_untrusted_uploads
```

Expected values are `0` for Blocked and `1` for Allowed. A save error should
restore the saved value and display an error notification. The default and
save boundaries are described in [Security model](SECURITY.md).

## Upgrade behavior

On the first upgrade from r1 or r2, the old shared
`luci.package_manager.allow_untrusted_uploads` setting is ignored. The new
configuration defaults to **Blocked**; enable it explicitly if required. The
shared `/etc/config/luci` file does not need modification.

If `/etc/config/luci-package-manager` already exists, APK preserves the active
configuration during later upgrades. It may place the new shipped default in
`/etc/config/luci-package-manager.apk-new`. Review that file before replacing
any active configuration. The selected setting also persists across reboot.

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

test -n "$REPO_VERSION" || { echo 'No repository version found' >&2; exit 1; }
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

After returning to the repository package, the unused dedicated configuration
can be removed if you no longer need its saved value:

```sh
rm -f /etc/config/luci-package-manager /etc/config/luci-package-manager.apk-new
```

Log out, log back in, and fully refresh LuCI after restoring the repository
package. No changes to the shared `/etc/config/luci` are needed.

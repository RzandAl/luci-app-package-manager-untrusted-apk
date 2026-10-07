# Security model

The setting permits APK signature trust to be bypassed for a confirmed local
upload. It defaults to disabled and does not expose general package-manager
flags to the browser.

## Trust boundary

The backend adds only `--allow-untrusted` when all five conditions are true:

- the configured package manager is `apk`;
- the requested operation is `install`;
- exactly one package argument was supplied;
- that argument is exactly `/tmp/upload.apk`;
- `luci-package-manager.main.allow_untrusted_uploads` is exactly `1` in UCI.

The `--` delimiter appears before package operands, preventing package names
and paths from being interpreted as APK options. The backend derives the trust
flag from the conditions above; the frontend cannot supply it directly.

With the setting disabled, the backend runs `apk add -- /tmp/upload.apk`.
**Blocked** therefore describes the trust bypass: a trusted local package may
still install, while an untrusted package is rejected by APK. The browser does
not reproduce APK signature verification.

## Configuration and save boundary

The option lives in `/etc/config/luci-package-manager`, with a shipped default
of `0`. The package registers this path as a configuration file. The
package-manager ACL grants UCI read and write access only to
`luci-package-manager` and allows the `uci.commit` and `uci.revert` RPC methods
needed to save and recover this configuration.

The frontend commits only the dedicated configuration. If a save fails, it
reverts that configuration's pending changes, reloads the saved value, restores
the page status, and displays the error. It does not commit the shared `luci`
configuration or unrelated pending changes. The real browser-session save and
failed-save paths were verified as described in [Release validation](VALIDATION.md).

The r1/r2 setting in the shared `luci` configuration is ignored by r3. It is not
migrated or deleted; the first upgrade defaults to disabled unless a dedicated
configuration already exists. Later upgrades preserve the dedicated configuration.

## Confirmation boundary

Enabling the setting does not authorize an installation by itself. Every
uploaded APK still requires a separate confirmation before installation.

The bypass does not apply to package names, URLs, other filesystem paths,
multiple operands, upgrades, removals, repository installations, or package
managers other than APK.

## Operator responsibility

A package accepted with `--allow-untrusted` is not authenticated by APK's normal
signature trust chain. Verify its source and checksum, review the package and
version, and confirm the upload only when it is the file you intended to install.
For this release, use all assets from the matching tag and follow
[Installation and rollback](INSTALLATION.md).

Disable **Allow untrusted local packages** to stop the bypass. To return to the
repository package, follow the [package-only rollback](INSTALLATION.md#return-to-the-repository-package).

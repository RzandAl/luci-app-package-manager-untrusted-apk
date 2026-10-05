# Security model

This patch deliberately adds a narrow, opt-in bypass for APK signature trust.
It does not make untrusted installation the default and does not expose general
package-manager flags to the browser.

## Trust boundary

The backend adds `--allow-untrusted` and `--force-non-repository` only when all
of the following conditions are true:

- the configured package manager is `apk`;
- the requested operation is `install`;
- exactly one package argument was supplied;
- that argument is exactly `/tmp/upload.apk`;
- `luci.package_manager.allow_untrusted_uploads` is enabled in UCI.

The `--` option delimiter is inserted before the package argument so a package
name or path cannot be interpreted as an `apk` option.

## Confirmation boundary

The setting is disabled by default. When enabled, LuCI still requires a
separate confirmation for every uploaded APK before installation begins.

The frontend cannot submit arbitrary trust flags. The backend derives the
additional flags only from the exact operation, argument, and UCI state above.

The **Blocked** state blocks the trust bypass, not every local APK upload. LuCI
does not attempt to reproduce APK signature verification in the browser. With
the option disabled, the backend runs the normal `apk add -- /tmp/upload.apk`;
a trusted package may install, while an unsigned or otherwise untrusted package
is rejected by `apk`.

## Operations that remain unaffected

The trust bypass is not applied to:

- package names;
- URLs;
- arbitrary filesystem paths;
- multiple package arguments;
- upgrades or removals;
- repository operations;
- package managers other than `apk`.

## Operator responsibility

An APK accepted by `--allow-untrusted` is not authenticated by APK's normal
signature trust chain. Before installation:

1. obtain the file from a source you trust;
2. verify its checksum independently;
3. review the requested package and version;
4. confirm the upload only when the file is the one you intended to install.

For this project's release, use the `SHA256SUMS` asset from the matching tag and
follow [Installation and rollback](INSTALLATION.md).

## Returning to the default trust model

Disable **Allow untrusted local packages** to block the bypass immediately.
To remove the modified package entirely, follow the package-only rollback in
[Installation and rollback](INSTALLATION.md#return-to-the-repository-package).

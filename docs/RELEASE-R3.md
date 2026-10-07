r3 moves **Allow untrusted local packages** into its own UCI configuration and
uses only `--allow-untrusted` for a confirmed `/tmp/upload.apk` installation.

- Stores the setting in `/etc/config/luci-package-manager`, disabled by default.
- Commits and reverts only that configuration, preserving shared LuCI values and
  unrelated pending changes. Failed saves restore the saved value and page status.
- Grants the scoped UCI access and `commit`/`revert` RPC methods required by Save.
- Keeps the exact upload guard, APK `--` delimiter, and per-upload confirmation.
- Registers the configuration file so later upgrades preserve the selected value.

On the first upgrade from r1/r2, the old shared `luci` option is ignored and the
new option starts **Blocked**. Log out, log back in, and fully refresh LuCI after
installation. Existing dedicated configurations are preserved.

Validated on Xiaomi Mi Router 3G with OpenWrt 25.12.5 and Firefox: isolated
saving, real revert after an injected commit failure, reboot persistence,
Allowed success, Blocked exit code 99, and upload cleanup. A repository-install
simulation left both configurations and `/etc/apk/world` unchanged. Final test
state was `0`. r2's wider device/browser coverage remains archived separately.

Source: [`1d3542d3a5348c96edbc3e155c49a293b1272f3f`](https://github.com/RzandAl/luci/commit/1d3542d3a5348c96edbc3e155c49a293b1272f3f),
signed and verified; upstream base `067535eaf51a59582b775a8b588a9b05810f8030`.
APK version: `26.280.00238~1d3542d`, architecture `noarch`.

The release contains the rebuilt APK, full standalone patch, and `SHA256SUMS`.
Verify both listed files before installation. The APK is intentionally untrusted;
first installation requires the terminal procedure in
[Installation](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/blob/openwrt-25.12-067535e-r3/docs/INSTALLATION.md).
See [Validation](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/blob/openwrt-25.12-067535e-r3/docs/VALIDATION.md)
and [Build and hashes](https://github.com/RzandAl/luci-app-package-manager-untrusted-apk/blob/openwrt-25.12-067535e-r3/docs/BUILDING.md).

This remains an unofficial replacement for the stock package. Upstream
[openwrt/luci#9110](https://github.com/openwrt/luci/pull/9110) is awaiting review.
r1 and r2 remain available with their original assets.

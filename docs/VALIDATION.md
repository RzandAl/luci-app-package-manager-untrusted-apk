# Validation

Project testing covers the environments below. The latest functional checks
completed on 7 October 2026.

## Tested environments

| Area | Environment |
| --- | --- |
| Xiaomi Mi Router 3G | OpenWrt 25.12.2 and 25.12.5, `ramips/mt7621` |
| Cudy WR3000S v1 | OpenWrt 25.12.5, `mediatek/filogic` |
| Browsers | Firefox, Chrome, and Microsoft Edge |
| Build | Official OpenWrt 25.12.5 SDK, `ramips/mt7621`, GCC 14.3.0, musl, Linux x86_64 |

## Functional checks

| Check | Result |
| --- | --- |
| Package update | Installation changed only `luci-app-package-manager`; existing dedicated configuration was preserved |
| Saving | Only the dedicated configuration was committed; shared LuCI values and unrelated pending changes stayed unchanged |
| Failed save | Dedicated pending changes were reverted; the saved value and page status were restored, with an error notification |
| Reboot | The selected setting and configuration checksums were preserved |
| Allowed upload | The untrusted package installed successfully after confirmation |
| Blocked upload | APK rejected the untrusted package with exit code `99` |
| Upload cleanup | `/tmp/upload.apk` was absent after Dismiss for both outcomes |
| Repository-install simulation | Passed for `jsonfilter`; both configuration files and `/etc/apk/world` stayed unchanged |
| Final test state | `allow_untrusted_uploads=0` |

The save checks staged an unrelated shared LuCI change and confirmed that
Configure APK Save preserved it. The failed-save test blocked a commit request
before it reached the router; the frontend then performed a real dedicated
revert and restored the saved state.

The repository test used the installed backend with a temporary PATH wrapper,
which checked `add -- jsonfilter` and forwarded it to `/usr/bin/apk --simulate`.
This checked the repository-install path without changing installed packages.

## Source and artifact checks

The published APK was built from signed source commit
`1d3542d3a5348c96edbc3e155c49a293b1272f3f`. The full patch applies cleanly to
upstream base `067535eaf51a59582b775a8b588a9b05810f8030` and produces Git tree
`4840108cc406996d631b8c3fc8be42a0e15cf14b`, matching that source.

Payload inspection confirmed the backend, ACL, dedicated default configuration,
registered configuration file, package version, and `noarch` metadata. The
frontend matched the SDK staging copy. Hashes and reproduction commands are in
[Building](BUILDING.md).

## Local checks and CI

Local mock checks passed 21 backend cases and five frontend save/error cases.
They covered enabled and disabled uploads, exact upload scope, APK option
delimiters, other operations and package managers, and save recovery.

The [validation workflow](../.github/workflows/validate.yml) verifies the patch
checksum, patched source tree, shell/JavaScript syntax, JSON validity,
configuration registration, disabled default, and UCI/RPC ACL scope.

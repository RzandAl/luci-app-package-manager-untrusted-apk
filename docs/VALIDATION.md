# r3 release validation

Validation for `openwrt-25.12-067535e-r3` completed on 7 October 2026. This
record describes the final APK from signed source commit
`1d3542d3a5348c96edbc3e155c49a293b1272f3f`. Earlier r2 results remain in the
[r2 validation archive](VALIDATION-R2.md).

## Environment

| Area | Verified environment |
| --- | --- |
| Build | Official OpenWrt 25.12.5 SDK, `ramips/mt7621`, GCC 14.3.0, musl, Linux x86_64 |
| Router | Xiaomi Mi Router 3G (`xiaomi,mi-router-3g`), `ramips/mt7621` |
| Firmware | OpenWrt 25.12.5, `r33051-f5dae5ece4`, kernel `6.12.94`, squashfs |
| Browser | Firefox |
| Installed package | `luci-app-package-manager`, `26.280.00238~1d3542d`, `noarch` |

r3 was not separately validated on the Cudy device or on OpenWrt 25.12.2.
The earlier r2 checks on those environments do not establish r3 coverage.

## Source and artifact

The final full patch applies cleanly to upstream base
`067535eaf51a59582b775a8b588a9b05810f8030` and produces Git tree
`4840108cc406996d631b8c3fc8be42a0e15cf14b`, matching the signed source.
Checks covered whitespace, shell and JavaScript syntax, JSON validity, dedicated
configuration registration, scoped UCI access, and the commit/revert ACL methods.

The APK build and payload inspection passed. Its SHA-256 is
`d7464472e3ecb487ccda350985f5b31ed2075e3cd52b452467e201182bd6ecc2`.
Config, backend, and ACL matched the signed source. The frontend matched the
SDK staging copy. Metadata confirmed the version, architecture, and registered
configuration file with the disabled default. Exact hashes and reproduction
commands are in [Building](BUILDING.md).

## Device results

| Check | Result |
| --- | --- |
| Package update | Simulation and installation changed only `luci-app-package-manager`; final version confirmed |
| Existing configuration | Dedicated opt-in `1` retained; shared file and pending CLI changes preserved; shipped default placed in `.apk-new` |
| Successful browser save | Changing `1` to `0` committed only the dedicated configuration; shared values and unrelated browser-session queue retained |
| Failed browser save | Injected commit failure followed by one real dedicated revert; queue emptied, cache and Blocked status restored |
| Error notification | Expected `R3_EXPECTED_COMMIT_FAILURE` notification appeared on Save; test hooks then removed |
| Reboot | Both configuration checksums unchanged; opt-in `1` and final installed version retained |
| Allowed upload | Final APK installed successfully after confirmation; dedicated value remained `1` |
| Blocked upload | APK returned expected exit code `99`; dedicated value remained `0` |
| Upload cleanup | `/tmp/upload.apk` absent after Dismiss for both outcomes |
| Repository-install simulation | Real installed backend invoked APK simulation for `jsonfilter`; both configurations and `/etc/apk/world` unchanged |
| Final state | `allow_untrusted_uploads=0` |

The isolation helper staged an unrelated shared `luci` change in the browser
session. Ordinary Configure APK Save preserved it. The helper then removed its
test change using a successful real shared revert; the shared file checksum
remained unchanged.

The failure helper observed the real dedicated pending change from `0` to `1`,
then rejected its commit before forwarding the request to the router. The
frontend itself performed the real dedicated revert. This verifies recovery
from a failed commit without deliberately failing a disk write. Both
configuration file checksums and all shared values remained unchanged.

The repository test called `/usr/libexec/package-manager-call install jsonfilter`
through a temporary PATH wrapper. The wrapper required exactly `add -- jsonfilter`
and forwarded those arguments to `/usr/bin/apk --simulate`. The backend returned
code `0` and `apk add -- jsonfilter`. This was a simulation, not an actual
repository package installation. The wrapper and persistent reboot checksum
file were removed after verification.

The final upload outcomes and cleanup were reported through LuCI and SSH.
Expected uploaded-file commands are established by payload inspection and local
backend tests; the final device report did not quote those command strings.
The screenshots in this repository are r2 captures.

## Local checks and CI

Separate local mock checks passed 21 backend cases and five frontend save/error
cases. They exercised exact upload scope, disabled and enabled states, APK
option delimiters, other actions and package managers, and save recovery.
These checks support the source review and are distinct from device evidence.

The [validation workflow](../.github/workflows/validate.yml) checks the patch
checksum, applies it to the exact upstream base, compares the complete source
Git tree, and verifies syntax, configuration registration, the disabled default,
and UCI/RPC ACL scope. CI does not replace the device checks above.

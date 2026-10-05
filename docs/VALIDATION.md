# Release validation

This document records the validation scope for release
`openwrt-25.12-067535e-r2`. It is a release record, not a compatibility claim
for later OpenWrt or LuCI revisions.

## Environment matrix

| Area | Environment |
| --- | --- |
| Build | Official OpenWrt 25.12.5 SDK for `ramips/mt7621` |
| r2 runtime — Xiaomi Mi Router 3G | OpenWrt 25.12.2 and 25.12.5 (`ramips/mt7621`) |
| r2 runtime — Cudy WR3000S v1 | OpenWrt 25.12.5 (`mediatek/filogic`) |
| Browser UI | Firefox, Chrome, and Microsoft Edge |

The resulting package reports `arch: noarch`; the target SDK was still used to
resolve and validate the OpenWrt package environment.

## Source and patch validation

The release was checked against:

- upstream LuCI base commit
  `067535eaf51a59582b775a8b588a9b05810f8030`;
- exact patched LuCI commit
  `e1fb46d3ecae5ede2eabcfe1792697ac734bb07a`;
- the standalone patch in
  `patches/luci-app-package-manager-untrusted-upload-067535e.patch`;
- clean standalone patch application to the upstream base;
- `git diff --check` after patch application.

Reproduction commands are documented in [BUILDING.md](BUILDING.md).

## Artifact validation

The published bundle was checked for:

- matching release checksums;
- APK SHA-256
  `eca926ba1a94611df054de07da466937532b96e9f7f5296d6d75a9b929e9bad8`;
- package version `26.278.01645~e1fb46d` and `noarch` metadata;
- expected payload paths;
- valid shell syntax for `package-manager-call`;
- valid JSON for the RPC ACL;
- unchanged APK contents after renaming the SDK output to the release filename.

The exact expected hashes and APK inspection commands are in
[BUILDING.md](BUILDING.md#inspect-the-apk).

## Runtime validation

Runtime validation covered:

- first installation from a terminal with the required trust flags;
- the default **Blocked** state;
- a blocked upload invoking `apk add -- /tmp/upload.apk`, rejected by APK with
  `UNTRUSTED signature` and exit code 99;
- explicit opt-in through **Configure APK**;
- the **Allowed** warning state after opt-in;
- a separate confirmation for each uploaded APK;
- successful installation after confirmation with exactly
  `apk --allow-untrusted --force-non-repository add -- /tmp/upload.apk`;
- restoring `luci.package_manager.allow_untrusted_uploads=0` after the test;
- removal of `/tmp/upload.apk` after package-manager completion.

The exact command, result, UCI-state, and upload-cleanup sequence above was
recorded on the Xiaomi device running OpenWrt 25.12.5. The r2 package and its
user-visible flow were also checked on the Xiaomi device running OpenWrt 25.12.2
and on the Cudy device running OpenWrt 25.12.5.

The screenshots in this repository were captured from the r2 interface on
OpenWrt 25.12.5.

## Continuous validation

The repository [validation workflow](../.github/workflows/validate.yml) repeats
the following checks on every push and pull request:

- fetch the exact upstream base commit;
- verify and apply the standalone patch;
- run `git diff --check`;
- verify the published patch checksum;
- check backend shell syntax, RPC ACL JSON, and frontend JavaScript syntax.

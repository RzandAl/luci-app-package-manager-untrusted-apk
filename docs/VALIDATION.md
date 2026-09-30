# Release validation

This document records the validation scope for release
`openwrt-25.12-067535e-r1`. It is a release record, not a compatibility claim
for later OpenWrt or LuCI revisions.

## Environment matrix

| Area | Environment |
| --- | --- |
| Build | Official OpenWrt 25.12.5 SDK for `ramips/mt7621` |
| Runtime — Xiaomi Mi Router 3G | OpenWrt 25.12.2 and 25.12.5 (`ramips/mt7621`) |
| Runtime — Cudy WR3000S v1 | OpenWrt 25.12.5 (`mediatek/filogic`) |

The resulting package reports `arch: noarch`; the target SDK was still used to
resolve and validate the OpenWrt package environment.

## Source and patch validation

The release was checked against:

- upstream LuCI base commit
  `067535eaf51a59582b775a8b588a9b05810f8030`;
- exact patched LuCI commit
  `4836c113cdbd8c01b4586970d9471541084aaaec`;
- the standalone patch in
  `patches/luci-app-package-manager-untrusted-upload-067535e.patch`;
- clean standalone patch application to the upstream base;
- `git diff --check` after patch application.

Reproduction commands are documented in [BUILDING.md](../BUILDING.md).

## Artifact validation

The published bundle was checked for:

- matching release checksums;
- expected APK package name, version, and `noarch` metadata;
- expected payload paths;
- valid shell syntax for `package-manager-call`;
- valid JSON for the RPC ACL;
- unchanged APK contents after renaming the SDK output to the release filename.

The exact expected hashes and APK inspection commands are in
[BUILDING.md](../BUILDING.md#inspect-the-apk).

## Runtime validation

Runtime validation covered:

- first installation from a terminal with the required trust flags;
- the default **Blocked** state;
- explicit opt-in through **Configure APK**;
- the **Allowed** warning state after opt-in;
- a separate confirmation for each uploaded APK;
- successful installation of an untrusted local APK after confirmation.

The same user-visible flow was checked on the Xiaomi and Cudy devices listed in
the environment matrix.

## Continuous validation

The repository [validation workflow](../.github/workflows/validate.yml) repeats
the following checks on every push and pull request:

- fetch the exact upstream base commit;
- verify and apply the standalone patch;
- run `git diff --check`;
- verify the published patch checksum;
- check backend shell syntax, RPC ACL JSON, and frontend JavaScript syntax.

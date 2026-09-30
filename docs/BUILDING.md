# Building

These instructions build the modified stock `luci-app-package-manager` from the
exact source commit used for the release.

## Requirements

- An OpenWrt 25.12 SDK for the target release and toolchain.
- The SDK feeds initialized according to the OpenWrt SDK documentation.
- Git, Bash, Python 3, and the usual OpenWrt build dependencies.
- Enough free disk space for the SDK build tree.

The published release was built with:

```text
OpenWrt SDK 25.12.5
Target: ramips/mt7621
Toolchain: GCC 14.3.0, musl
Host: Linux x86_64
```

The resulting LuCI package reports `arch: noarch`; the target SDK is still used
to resolve and validate the OpenWrt package environment.

## Build the exact release source

Run the following commands from the SDK root. The `feeds/luci` working tree must
be clean.

Add the development fork once, or update its URL if it already exists:

```sh
git -C feeds/luci remote get-url fork >/dev/null 2>&1 \
    || git -C feeds/luci remote add fork https://github.com/RzandAl/luci.git

git -C feeds/luci remote set-url \
    fork https://github.com/RzandAl/luci.git
```

Fetch the release branch and detach at the exact source commit:

```sh
git -C feeds/luci fetch fork \
    '+refs/heads/fix/package-manager-untrusted-upload-067535e:refs/remotes/fork/fix/package-manager-untrusted-upload-067535e'

git -C feeds/luci switch --detach \
    4836c113cdbd8c01b4586970d9471541084aaaec

test "$(git -C feeds/luci rev-parse HEAD)" = \
    '4836c113cdbd8c01b4586970d9471541084aaaec'
```

Refresh the package link from the checked-out LuCI feed:

```sh
./scripts/feeds install \
    -f \
    -p luci \
    luci-app-package-manager
```

Select `luci-app-package-manager` in `make menuconfig`, then normalize the SDK
configuration:

```sh
make menuconfig
make defconfig
```

Clean and compile only the package:

```sh
make package/feeds/luci/luci-app-package-manager/clean
make package/feeds/luci/luci-app-package-manager/compile V=s
```

Locate the result:

```sh
find bin/packages \
    -type f \
    -name 'luci-app-package-manager-*~4836c11.apk' \
    -print
```

## Inspect the APK

Set `PACKAGE_APK` to the path printed above:

```sh
PACKAGE_APK='bin/packages/ARCH/luci/luci-app-package-manager-26.272.39633~4836c11.apk'
APK_CHECK_DIR="$(mktemp -d)"

sha256sum "$PACKAGE_APK"

./staging_dir/host/bin/apk extract \
    --allow-untrusted \
    --no-chown \
    --destination "$APK_CHECK_DIR" \
    "$PACKAGE_APK"

bash -n \
    "$APK_CHECK_DIR/usr/libexec/package-manager-call"

python3 -m json.tool \
    "$APK_CHECK_DIR/usr/share/rpcd/acl.d/luci-app-package-manager.json" \
    >/dev/null
```

Inspect the package metadata:

```sh
./staging_dir/host/bin/apk adbdump "$PACKAGE_APK"
```

For the published `r1` artifact, the expected APK SHA-256 is:

```text
de487b6d8817ecc12bf057982e1f8e8d760a0ed5a83d56c959af94e0794ec6f5
```

## Validate the standalone patch

The standalone patch is based on upstream commit:

```text
067535eaf51a59582b775a8b588a9b05810f8030
```

Fetching the development branch also fetches its base commit. To validate that
the patch applies independently, start from a clean `feeds/luci` tree:

```sh
git -C feeds/luci switch --detach \
    067535eaf51a59582b775a8b588a9b05810f8030

git -C feeds/luci apply --check \
    /path/to/luci-app-package-manager-untrusted-upload-067535e.patch

git -C feeds/luci apply \
    /path/to/luci-app-package-manager-untrusted-upload-067535e.patch

git -C feeds/luci diff --check
```

Building an uncommitted patch on top of the base commit does not reproduce the
release package version string, because LuCI derives that string from Git
metadata. Check out the exact patched commit when reproducing the released APK.

## Release checksums

The SDK emits `luci-app-package-manager-26.272.39633~4836c11.apk`. The
published copy has the GitHub-safe, tag-based filename shown below; its content
and internal package version are unchanged.

The `r1` release bundle contains:

```text
luci-app-package-manager-openwrt-25.12-067535e-r1.apk
luci-app-package-manager-untrusted-upload-067535e.patch
SHA256SUMS
```

Expected hashes:

```text
de487b6d8817ecc12bf057982e1f8e8d760a0ed5a83d56c959af94e0794ec6f5  luci-app-package-manager-openwrt-25.12-067535e-r1.apk
91e8aeb24e6892897ac5d7298c16be724f737107626af90c7c27ac5df6d5a800  luci-app-package-manager-untrusted-upload-067535e.patch
```

The expected SHA-256 of `SHA256SUMS` itself is:

```text
5a8a8a14cb1e5abc8cfdb47c4c0f9125c543a69bc7bfefe86c3d89224597cbaa
```

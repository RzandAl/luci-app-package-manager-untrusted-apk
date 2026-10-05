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
    e1fb46d3ecae5ede2eabcfe1792697ac734bb07a

test "$(git -C feeds/luci rev-parse HEAD)" = \
    'e1fb46d3ecae5ede2eabcfe1792697ac734bb07a'
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
    -name 'luci-app-package-manager-*~e1fb46d.apk' \
    -print
```

## Inspect the APK

Set `PACKAGE_APK` to the path printed above:

```sh
PACKAGE_APK='bin/packages/ARCH/luci/luci-app-package-manager-26.278.01645~e1fb46d.apk'
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

For the published `r2` artifact, the expected APK SHA-256 is:

```text
eca926ba1a94611df054de07da466937532b96e9f7f5296d6d75a9b929e9bad8
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

The SDK emits `luci-app-package-manager-26.278.01645~e1fb46d.apk`. The
published copy has the GitHub-safe, tag-based filename shown below; its content
and internal package version are unchanged.

The `r2` release bundle contains:

```text
luci-app-package-manager-openwrt-25.12-067535e-r2.apk
luci-app-package-manager-untrusted-upload-067535e.patch
SHA256SUMS
```

Expected hashes:

```text
eca926ba1a94611df054de07da466937532b96e9f7f5296d6d75a9b929e9bad8  luci-app-package-manager-openwrt-25.12-067535e-r2.apk
921cb36984cc9b7374e7554fccf59a42d491aef4b657b6da06aa1830f835d670  luci-app-package-manager-untrusted-upload-067535e.patch
```

The expected SHA-256 of `SHA256SUMS` itself is:

```text
3aafbc6b5b25fac1aeebc482741af4e17059b6ee4453b450773a2a14fa562390
```

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
    1d3542d3a5348c96edbc3e155c49a293b1272f3f

test "$(git -C feeds/luci rev-parse HEAD)" = \
    '1d3542d3a5348c96edbc3e155c49a293b1272f3f'
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
    -name 'luci-app-package-manager-*~1d3542d.apk' \
    -print
```

## Inspect the APK

Set `PACKAGE_APK` to the path printed above:

```sh
PACKAGE_APK='bin/packages/ARCH/luci/luci-app-package-manager-26.280.00238~1d3542d.apk'
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

Confirm that the payload contains the dedicated default configuration and that
metadata registers `/etc/config/luci-package-manager` as a configuration file.
The default is `allow_untrusted_uploads '0'`. The ACL must grant UCI read/write
access only to `luci-package-manager` and write RPC access to `uci.commit` and
`uci.revert`. The backend must contain only `--allow-untrusted` as its trust
bypass flag. Compare config, backend, and ACL with the exact source commit;
compare the frontend with the SDK's staged copy, which may be transformed by
the build.

For the validated r3 artifact, the expected APK SHA-256 is:

```text
d7464472e3ecb487ccda350985f5b31ed2075e3cd52b452467e201182bd6ecc2
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

git -C feeds/luci apply --index \
    /path/to/luci-app-package-manager-untrusted-upload-067535e.patch

git -C feeds/luci diff --cached --check
test "$(git -C feeds/luci write-tree)" = \
    '4840108cc406996d631b8c3fc8be42a0e15cf14b'
```

Building an uncommitted patch on top of the base commit does not reproduce the
release package version string, because LuCI derives that string from Git
metadata. Check out the exact patched commit when reproducing the released APK.

## Release checksums

Build instructions identify the exact source and toolchain. A new build may
produce different APK bytes; compare its payload and metadata rather than
assuming byte-for-byte reproduction of the validated artifact.

The SDK emits `luci-app-package-manager-26.280.00238~1d3542d.apk`. The
published copy has the GitHub-safe, tag-based filename shown below; its content
and internal package version are unchanged.

The `r3` release bundle contains:

```text
luci-app-package-manager-openwrt-25.12-067535e-r3.apk
luci-app-package-manager-untrusted-upload-067535e.patch
SHA256SUMS
```

Expected hashes:

```text
d7464472e3ecb487ccda350985f5b31ed2075e3cd52b452467e201182bd6ecc2  luci-app-package-manager-openwrt-25.12-067535e-r3.apk
8cfbe3d45b11ea4688cbedb0af609584ce531180b61cc613f9400d8a8dbd9412  luci-app-package-manager-untrusted-upload-067535e.patch
```

The expected SHA-256 of `SHA256SUMS` itself is:

```text
bf2f0ee8e3b04c8e44bfe49b5ebbfa43fb660c7815c9cd4de2a0b4f13307d596
```

Generate `SHA256SUMS` from the renamed APK and matching patch in that order:

```sh
sha256sum \
    luci-app-package-manager-openwrt-25.12-067535e-r3.apk \
    luci-app-package-manager-untrusted-upload-067535e.patch > SHA256SUMS
sha256sum -c SHA256SUMS
```

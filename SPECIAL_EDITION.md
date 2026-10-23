# Clash Meta for Android（特供版）

本仓库是 [MetaCubeX/ClashMetaForAndroid](https://github.com/MetaCubeX/ClashMetaForAndroid) 的自动同步分支，内置 [ztyawc/mihomo-cn2](https://github.com/ztyawc/mihomo-cn2) 魔改内核。

## 产物

- 应用名：`Clash Meta（特供版）`
- 包名：`com.github.metacubex.clash.ztyawc`
- 架构：`arm64-v8a`
- 签名：固定发布证书，下载 `-signed.apk` 后可直接安装

APK 仅通过 [GitHub Releases](../../releases) 发布，不提交到 Git 历史。

## 自动维护

`.github/workflows/sync-upstream-release.yml` 每 3 小时运行一次：

1. 合并 `MetaCubeX/ClashMetaForAndroid` 的最新 `main`，保持应用层与上游同步。
2. 将内核子模块更新到 `ztyawc/mihomo-cn2` 的最新 `main`。
3. 同步 Go 模块依赖并把变更推送到本仓库 `main`。
4. 当上游出现尚未镜像的稳定 Release 时，构建、校验、使用固定证书签名，并发布新的 arm64 APK。

如果上游改动与特供版定制发生 Git 冲突，同步会失败并保留日志，需要手动解决后再重跑。

## 预发布版

在 Actions 中手动运行 `Sync upstream and release special APK` 并勾选 `prerelease`（或执行 `gh workflow run sync-upstream-release.yml -f prerelease=true`），会先完成一次同步，再用 `main` 的最新代码和 `ztyawc/mihomo-cn2` 的最新内核构建 APK，发布为预发布版 `special-Prerelease-alpha`。每次发布都会替换上一个预发布版。

预发布版与正式特供版包名相同；使用相同签名证书的版本可以覆盖升级。

## 自定义内核构建

`.github/workflows/build-custom-kernel.yml` 用 `main` 的最新代码和指定的 mihomo 内核仓库（默认 `ztyawc/mihomo-cn2` 的 `main`）构建 arm64 APK，并使用相同的固定发布证书签名：

```bash
gh workflow run build-custom-kernel.yml -f kernel_repository=ztyawc/mihomo-cn2
```

内核替换只发生在构建机上，不会推送代码，也不会发布 Release。APK 仅作为该次运行的构件（artifact）保存，在运行页面下载名称以 `-signed` 结尾的构件。包名、签名证书与正式特供版相同，可以覆盖升级。

## 固定签名证书

自动发布和自定义内核构建统一调用 `.github/workflows/sign-apk.yml`。签名任务在独立构建机上执行，不运行内核代码或 Gradle；生成 APK 后验证签名、证书指纹以及 ZIP 对齐，再提供下载。

- 类型：RSA 4096 位，SHA256withRSA，有效期 100 年
- 密钥别名：`cmfa-release`
- 公开证书：[release-certificate.pem](.github/signing/release-certificate.pem)
- 证书 SHA-256：`4468b44ee732fff612999f4658d4891525ab0b9b531dbf5fe8a4d6e8e54f624c`
- GitHub Secrets：`CMFA_RELEASE_KEYSTORE_BASE64` 和 `CMFA_RELEASE_KEYSTORE_PASSWORD`

私钥及密码只保存在 GitHub Secrets 和本机的 `~/.local/share/cmfa-signing/ClashMetaForAndroid/` 备份目录，不提交到仓库。请完整备份该目录；后续继续使用同一私钥，不要重新生成。Secrets 缺失或证书指纹不匹配时，签名任务会失败，不会生成其他证书或回退使用 debug key。

`-unsigned` 构件是签名前的中间产物，只保留 1 天。发布下载及 `-signed` 构件均使用固定证书签名。此前使用其他证书自行签名的安装包无法直接覆盖升级到本证书签名的版本。

## 手动构建

需要 Java 21、Go 1.26、Android SDK 35、NDK `29.0.14206865` 和 CMake `3.22.1`。在本机的 `signing.properties` 中配置备份的固定证书：

```properties
keystore.path=/path/to/cmfa-release.keystore
keystore.password=<备份中的密码>
key.alias=cmfa-release
key.password=<备份中的密码>
```

```bash
git submodule update --init --recursive
./gradlew --no-daemon app:assembleMetaRelease \
  -Pandroid.injected.build.abi=arm64-v8a
```

未配置 `signing.properties` 时，Release 不会回退使用 debug key。

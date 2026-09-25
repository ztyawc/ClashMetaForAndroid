# Clash Meta for Android（特供版）

本仓库是 [MetaCubeX/ClashMetaForAndroid](https://github.com/MetaCubeX/ClashMetaForAndroid) 的自动同步分支，内置 [ztyawc/mihomo](https://github.com/ztyawc/mihomo) 魔改内核。

## 产物

- 应用名：`Clash Meta（特供版）`
- 包名：`com.github.metacubex.clash.ztyawc`
- 架构：`arm64-v8a`
- 签名：真正未签名 APK，下载后必须自行签名才能安装

APK 仅通过 [GitHub Releases](../../releases) 发布，不提交到 Git 历史。

## 自动维护

`.github/workflows/sync-upstream-release.yml` 每 3 小时运行一次：

1. 合并 `MetaCubeX/ClashMetaForAndroid` 的最新 `main`，保持应用层与上游同步。
2. 将内核子模块更新到 `ztyawc/mihomo` 的最新 `main`。
3. 同步 Go 模块依赖并把变更推送到本仓库 `main`。
4. 当上游出现尚未镜像的稳定 Release 时，构建、校验并发布新的 arm64 未签名 APK。

如果上游改动与特供版定制发生 Git 冲突，同步会失败并保留日志，需要手动解决后再重跑。

## 预发布版

在 Actions 中手动运行 `Sync upstream and release special APK` 并勾选 `prerelease`（或执行 `gh workflow run sync-upstream-release.yml -f prerelease=true`），会先完成一次同步，再用 `main` 的最新代码和 `ztyawc/mihomo` 的最新内核构建 APK，发布为预发布版 `special-Prerelease-alpha`。每次发布都会替换上一个预发布版。

预发布版与正式特供版包名相同，安装会覆盖正式版。

## 自定义内核构建

`.github/workflows/build-custom-kernel.yml` 用 `main` 的最新代码和指定的 mihomo 内核仓库（默认 `ztyawc/mihomo-cn2` 的 `main`）构建 arm64 未签名 APK：

```bash
gh workflow run build-custom-kernel.yml -f kernel_repository=ztyawc/mihomo-cn2
```

内核替换只发生在构建机上，不会推送代码，也不会发布 Release。APK 仅作为该次运行的构件（artifact）保存，在运行页面下载。包名与正式特供版相同，安装会覆盖。

## 手动构建

需要 Java 21、Go 1.26、Android SDK 35、NDK `29.0.14206865` 和 CMake `3.22.1`。

```bash
git submodule update --init --recursive
./gradlew --no-daemon app:assembleMetaRelease \
  -Pandroid.injected.build.abi=arm64-v8a
```

未配置 `signing.properties` 时，Release 不会回退使用 debug key。

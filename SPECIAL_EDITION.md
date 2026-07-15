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

## 手动构建

需要 Java 21、Go 1.26、Android SDK 35、NDK `29.0.14206865` 和 CMake `3.22.1`。

```bash
git submodule update --init --recursive
./gradlew --no-daemon app:assembleMetaRelease \
  -Pandroid.injected.build.abi=arm64-v8a
```

未配置 `signing.properties` 时，Release 不会回退使用 debug key。

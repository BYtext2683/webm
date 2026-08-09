# WebpAlbum —— 类系统相册的本地媒体管理器（Android）

一个原生 Android（Kotlin）应用，操作习惯与系统相册一致：

- **批量导入**：`.webp / .gif / .jpg / .png / .bmp` 等图片，以及 `.mp4` 等视频
- **缩略图网格**浏览，视频 / 动图带角标
- **相册分类**与**重命名**
- **排序**：自定义顺序（长按拖拽）、导入时间、文件大小、文件类型、名称
- **快捷选择**：顶部「选择」进入多选，支持**全选 / 反选**
- **全屏播放**：动画 WebP / GIF 无限循环；MP4 循环播放；**左右滑动翻页**，点击暂停 / 继续

> 全程**不转码为 MP4**，`.webp` 直接播放。

---

## ⚠️ 先确认版本：标题栏应显示 `WebpAlbum v2.0`

从 v2.0 起，App 标题栏会**直接显示版本号**。装好后打开 App 看一眼：

| 标题栏显示 | 说明 |
|---|---|
| `WebpAlbum v2.0` | ✅ 装的是最新版，排序 / 全选 / MP4 播放全都在 |
| `WebpAlbum`（无版本号） | ❌ 还是旧版。说明源码没有完整上传到 GitHub，或 APK 没覆盖成功 |

**若仍是旧版**：多半是在 GitHub 网页上传时只覆盖了部分文件。请把 `WebpAlbum` 目录**整体重新上传一次**（同名文件会自动覆盖），等 Actions 变绿后重新下载 APK 覆盖安装。

### v2.0 修了什么

- **MP4 点开闪退** → 旧版导入时把所有文件一律存成 `.webp` 扩展名、MIME 固定 `image/webp`，MP4 被当图片交给 `ImageDecoder` 解码而崩溃。
  - 现在导入时按**文件头（magic bytes）**判定真实类型，不再信任扩展名；
  - 启动时自动扫描并**修复历史存坏的数据**（会弹「已修复 N 个早期导入的文件类型」），之前导入的 MP4 无需重导即可播放；
  - 播放页新增兜底：解码 / 播放失败只显示一行错误文字，**不再闪退**。
- **排序、全选 / 反选看不到** → 这两项在 v1.1 里本就不存在，属于旧版没有该功能，升到 v2.0 即出现。
- 视频缩略图改为取**第 1 秒的画面**做封面，取帧失败显示灰底占位。

---

## 环境要求

- 最低系统：**Android 9（API 28）**（`ImageDecoder` + `AnimatedImageDrawable` 是 API 28 引入的）
- 构建：JDK 17 + Android SDK 34 + Gradle 8.9（云端 CI 已自动准备好，本地无需安装）

---

## 不装 Android Studio，用 GitHub 云端编译出 APK

### 情况 A：仓库根目录 = 本工程（推荐，最省事）

把 `WebpAlbum` **文件夹里的内容**（`settings.gradle.kts`、`app/`、`.github/` 等）直接放到仓库根目录。
本工程自带的 `.github/workflows/build.yml` 即可直接工作。

### 情况 B：仓库里有个 `WebpAlbum/` 子目录

GitHub Actions 只识别**仓库根目录**的 `.github/workflows/`。此时需要在**仓库根**新建
`.github/workflows/build.yml`，内容与本工程内的相同，但要加上工作目录并修改产物路径：

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: WebpAlbum      # ← 新增
    steps:
      ...
      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: app-debug-apk
          path: WebpAlbum/app/build/outputs/apk/debug/app-debug.apk   # ← 带前缀
```

### 取 APK 并安装

1. 仓库页面顶部 → **Actions** → 找到 **Build Debug APK**
2. 等黄点变**绿色对勾**（首次 3–6 分钟）
3. 该运行页面下滚到 **Artifacts** → 点 **app-debug-apk** 下载
4. 解压得到 `app-debug.apk`，传到手机
5. 手机开启「未知来源应用安装」后点击安装，桌面出现 **WebpAlbum** 图标

> 之后更新：改完代码重新推送 → Actions 自动重编 → 下载新 APK **覆盖安装**（包名不变，数据保留）。

> **上传避坑**：网页拖拽上传时，请拖 `WebpAlbum` **文件夹本身**或其**内部所有内容**，不要在仓库里拖出
> `WebpAlbum/WebpAlbum/...` 这种嵌套结构 —— 嵌套后 CI 找不到 `settings.gradle.kts`，构建必失败。
> 上传前建议先看一眼仓库文件树：`app/`、`build.gradle.kts`、`settings.gradle.kts` 应与 `.github/` 同级。

---

## 使用说明

| 操作 | 位置 |
|---|---|
| 导入文件 | 右下角 **+**，系统选择器可一次多选（图片 + 视频） |
| 切换相册 | 顶部 Chip 横向条 |
| 新建 / 重命名 / 删除相册 | 右上角 **⋮ → 管理相册** |
| 排序 | 右上角 **排序**，选「自定义顺序」后可长按拖动排列 |
| 多选 | 右上角 **选择** 进入，之后**单击**即可勾选；也支持长按进入 |
| 全选 / 反选 | 多选模式下顶部两个按钮 |
| 移动到相册 / 重命名 / 删除 | 多选模式 **⋮** 菜单 |
| 播放 | 点缩略图进入全屏；左右滑翻页；点击暂停 / 继续 |

## 技术要点

- **动图播放**：`ImageDecoder.decodeDrawable` → `AnimatedImageDrawable`，`REPEAT_INFINITE`，由系统 RenderThread 驱动，不占主线程
- **视频播放**：`VideoView` + `MediaPlayer.isLooping = true`，走硬件解码
- **缩略图**：Glide `asBitmap()` 取静态首帧（视频取封面帧），网格滚动不卡
- **存储**：文件复制到 App 私有目录 `filesDir/media`，无需任何存储权限
- **数据库**：Room，`items` 表带 `mimeType / size / position`，含 v1→v2 迁移（老数据不丢）

## 已知限制

- 文件存于 App 私有目录，**卸载 App 会一并删除**。若要卸载不丢，可改用 `MediaStore`。
- 视频缩略图个别格式可能取帧失败，显示灰底占位（不影响播放）。
- 视频按 `fitCenter` 显示，非铺满全屏。

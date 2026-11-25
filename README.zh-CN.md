# 啊哈的工具箱（Aha Toolbox）

**面向学生的离线本地小工具箱**：课表 OCR 导入、题库练习、作业/成绩、笔记、加密相册、一键备份。
数据全部保存在本机：无账号、无云端、无统计上报。

## 功能

- **课表**：多学期周课表、单双周、点按编辑、导出图片，支持**截图 OCR 导入**（Tesseract.js + 内置中文模型，完全离线）
- **学习**：题库（文字题/图片题）、练习模式与掌握度、作业、成绩、单词复习
- **日常**：记账与预算提醒、快递取件码、BMI/体重、习惯打卡、番茄钟、随机抽签/转盘
- **收藏**：链接、公式、行程、活动
- **笔记**：文字/图片笔记、截图标注
- **目标**：进度与归档
- **加密相册**：密码保护的相册（保存原图字节，AES-GCM）与密码本
- **数据安全**：带附件的版本化 JSON 备份、恢复预览与一键撤销、配额失败回滚、本地数据损坏时只读保护
- **离线可用**：带 Service Worker 的可安装 PWA；安卓端内置 OCR 引擎

## 目录结构

```
web/        网页层源码（HTML / CSS / 原生 JS），由 Capacitor WebView 加载
android/    自定义原生插件（系统"另存为"）+ Capacitor 插件注册表
tools/      补丁式构建流水线：javac + d8 → dex、重打包 APK、zipalign、签名
baseline/   放置待补丁的原始 APK（二进制文件未提交）
docs/       构建说明与更新日志
```

## 构建

> 构建是**补丁式**的：安卓壳是预编译 APK，流水线替换其中的 `assets/public` 网页层、
> 追加插件 dex 并重新签名，因此 `baseline/` 需要原始 APK。

依赖：Python 3.10+、JDK 21、Android SDK Build-Tools 35、调试签名证书
（`%USERPROFILE%\.android\debug.keystore`，别名 `androiddebugkey`）。

1. 把原始 APK 放到 `baseline/aha-toolbox-b34-fix5.apk`。
2. 编译原生插件为 dex：
   `python tools/compile_plugin.py --src android/AhaDocumentsPlugin.java --output build/dex/classes2.dex`
3. 重打包并签名：
   `python tools/repack_apk.py --baseline baseline/aha-toolbox-b34-fix5.apk --assets web --dex build/dex/classes2.dex --output build/aha-toolbox.apk`

脚本顶部的 JDK / SDK 路径是生成时的机器路径，换电脑构建时按需修改。

## 更新日志

见 [docs/CHANGELOG.md](docs/CHANGELOG.md)。最新构建 **b34-fix6**。

## 隐私

- 无账号、无注册、无统计上报
- 数据全部保存在本机
- 备份是用户主动导出的 JSON 文件
- OCR 全部在本机运行

## 许可证

MIT License，见 [LICENSE](LICENSE)。

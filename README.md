# LM 工具箱（flutter-tool-for-pdf）

基于 Flutter + [pdf_manipulator](https://pub.dev/packages/pdf_manipulator)（Rust 引擎）的跨平台 PDF 工具箱，支持 **Web 与桌面（Windows）** 双端运行，所有 PDF 处理均在后台线程完成，不阻塞界面。

**在线体验（Web 版已部署）：** <https://37lnn7o7e63bjykb75q.traesite.cn/>

## 功能一览

| 功能 | 说明 |
| --- | --- |
| 合并 PDF | 多选/拖入多个文件，按顺序合并为单个 PDF |
| 拆分 PDF | 按每 N 页拆分为多个文件，输出自动编号 |
| 压缩 PDF | 无损优化 / 高清打印 / 均衡 / 极限多档预设，展示逐项与总体前后体积对比 |
| 图片转 PDF | 多张 JPG/PNG 合并为一个 PDF，每张图片一页 |
| PDF 转图片 | 逐页渲染为 PNG，可限制最大宽度、指定页码范围，实时显示已完成页数 |
| 加密保护 | 设置打开密码 / 权限密码（可控制打印、复制、修改权限，AES-128/256），也可移除已有加密 |
| 页面编辑 | 可视化编辑器：缩略图浏览、旋转、删除、重排、插入其他 PDF 页面、内容擦除、文字水印（支持平铺与透明度）、元数据编辑，保存为新 PDF |

## 界面与交互

- Material 3 界面，左侧 `NavigationRail` 导航（宽屏自动展开文字标签），右侧 `IndexedStack` 切换功能页。
- 各功能页支持**文件拖入**（`desktop_drop`）与多选。
- 统一的**串行批量任务队列**：运行中可继续追加任务，支持单项取消、全部取消、失败重试；压缩任务展示源文件与输出文件的大小对比。
- Web 端结果通过浏览器下载保存；桌面端写入用户指定的输出目录（输出文件名自动带毫秒级时间戳，多次运行互不覆盖）。

## 项目结构

```
lib/
├── main.dart                      # 应用入口 + 主页（NavigationRail + IndexedStack）
├── pages/                         # 七个功能页
│   ├── merge_page.dart            # 合并
│   ├── split_page.dart            # 拆分
│   ├── compress_page.dart         # 压缩
│   ├── images_to_pdf_page.dart    # 图片转 PDF
│   ├── pdf_to_images_page.dart    # PDF 转图片
│   ├── protect_page.dart          # 加密 / 解密
│   └── page_editor_page.dart      # 可视化页面编辑器
├── services/
│   ├── pdf_core.dart              # 字节核心（PdfCore）：Web 与桌面共用，任务可取消
│   ├── pdf_file_service.dart      # 桌面文件门面（PdfService）：File 进 / 文件出
│   ├── pdf_file_service_stub.dart # Web 构建下的条件导出桩
│   ├── pdf_service.dart           # 统一出口（条件导出 pdf_file_service）
│   ├── task_queue.dart            # 串行批量任务队列（取消 / 重试 / 追加）
│   ├── picked_doc.dart            # 已选文件的统一抽象（名称 + 字节）
│   ├── platform_io.dart           # 平台选择/保存文件的条件导出
│   ├── platform_io_web.dart       # Web 端实现（浏览器下载）
│   ├── platform_io_desktop.dart   # 桌面端实现（file_picker + open_filex）
│   └── app_settings.dart          # 全局设置（输出目录）
└── widgets/
    └── common.dart                # 各页面共用组件
```

### 架构要点

- **字节核心 + 平台门面分层**：所有 PDF 操作统一走 `PdfCore`（`Uint8List` 进出），页面代码只依赖字节接口；桌面端的文件读写由 `PdfService` 门面负责。Web 构建通过 `export ... if (dart.library.js_interop)` 条件导出排除文件层。
- **可取消任务模型**：`PdfJob` / `PdfRenderJob` 封装底层 `PdfTask`，支持幂等取消；取消或失败不会产生半成品输出。
- **页面编辑会话**：`PdfEditSession` 渲染逐页缩略图并暂存编辑操作，保存时一次性应用；插入外部 PDF 时按"连续同源段"拆分处理，规避引擎 `mergeFrom` 只能追加的限制。

## 主要依赖

| 依赖 | 用途 |
| --- | --- |
| `pdf_manipulator` | PDF 引擎（Rust）：合并、拆分、压缩、渲染、加密、页面编辑 |
| `file_picker` | 文件/目录选择 |
| `open_filex` | 桌面端打开输出文件 |
| `desktop_drop` | 拖放文件 |

## 本地开发与构建

环境要求：Flutter SDK（Dart SDK `^3.12.0`）。

```bash
# 安装依赖
flutter pub get

# 运行（Web / Windows）
flutter run -d chrome
flutter run -d windows

# 构建 Web 发布版（即当前线上部署版本）
flutter build web

# 构建 Windows 桌面版
flutter build windows

# 运行测试
flutter test
```

## 许可证

仅供内部使用，未发布到 pub.dev。

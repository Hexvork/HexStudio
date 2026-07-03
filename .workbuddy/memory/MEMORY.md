# LMMS 项目长期记忆

## 项目概况
- **项目**：LMMS（D:\Github\lmms），开源跨平台数字音频工作站（DAW）
- **GUI 框架**：Qt5（≥5.15）/ Qt6（≥6.8），基础风格 Fusion
- **构建**：CMake + vcpkg，Qt 需系统级安装（仓库不含）

## UI 主题机制（关键）
- 主题目录：`data/themes/<主题名>/`，含 `style.css`（QSS 语法）+ 图标资源（PNG/SVG）
- 调色板：`src/gui/LmmsPalette.cpp` 是 dummy 类，从 CSS 的 `qproperty`（background/text/button/highlight 等）读取颜色 → **改 CSS 即改调色板，无需动 C++**
- 自定义绘制：`src/gui/LmmsStyle.cpp`（QProxyStyle）手绘像素级斜角边框，部分原生绘制无法被 CSS 覆盖
- 热重载：`QFileSystemWatcher` 监听 style.css，运行中改 CSS 即时生效
- 主题切换：`~/.lmmsrc.xml` 的 `<paths theme="目录路径">` 或 SetupDialog → "Theme directory"
- 资源前缀：`resources:`（style.css/图标）、`artwork:`（背景图），由 `QDir::addSearchPath` 注册

## 翻译机制（关键）
- 翻译源：`data/locale/zh_CN.ts`（XML），3007 条
- 构建：`data/locale/CMakeLists.txt` 用 lupdate/lrelease，`finalize-locales` target 自动生成 .qm
- 加载：`src/core/main.cpp` 用 QTranslator 加载，配置 `[app] language=zh`
- 乐器插件字符串全部用 tr()，纯翻译 .ts 即可，无需改源码
- 插件显示名 displayName 是硬编码英文（内部 ID，不建议翻译）

## 已有主题（2026-07 新增）
- `default`：深色复古拟物（#111314 底 + 绿色强调 + 斜角边框）
- `classic`：浅色经典
- `fluent-dark`：**v2** Win11 Fluent 深色（**#1A1A1A 底 + #60CDFF 强调蓝 + Segoe UI Variable + 微渐变按钮 + 胶囊圆角滚动条 + 四层层级递进**）
- `fluent-light`：Win11 Fluent 浅色（#F3F3F3 底 + #0078D4 强调蓝）

## 已知 UI Bug 及修复（2026-07-03）
- **SubWindow eventFilter 吞噬 WindowStateChange**（L620-622）：导致放大后 QComboBox/QMenu 弹窗定位错误、控件点不了 → 修复为传递基类处理
- **SubWindow changeEvent 缺少 WindowStateChange 分支**：放大/缩小后不更新标题栏按钮 → 新增分支调用 adjustTitleBar()
- **MainWindow 无 changeEvent**：最小化恢复后窗口无焦点/无响应 → 新增 changeEvent 实现 activateWindow() 恢复焦点

## 翻译状态
- zh_CN.ts 已达 100%（2026-07-02 从 59.5% 补全，1217 条未译全部翻译）
- 含乐器二级页面（TripleOscillator/Kicker/Monstro/FreeBoy/Nes/Opulenz/Watsyn/Vibed/Xpressive 等）控件文字
- 音频术语统一：振荡器/包络/起音/衰减/延音/释音/截止频率/共振/压缩器/混响/延迟/失真/镶边/均衡器等

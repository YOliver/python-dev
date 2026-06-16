---
name: python-dev
description: Use when developing Python desktop applications in this project - running the program, building release packages, managing start/release scripts, adding help documentation, or adding menu items
---

# Python 开发工作流

## Overview

项目级 Python 桌面应用开发 skill，管理启动和打包流程。自动检测并维护 `start.bat`（一键启动）和 `release.bat`（一键构建打包），确保打包时同步更新 README.md。

## When to Use

- 用户想要**运行/启动程序**时
- 用户想要**构建安装包/打包**时
- 用户提到 "start"、"run"、"启动"、"运行"
- 用户提到 "build"、"release"、"打包"、"构建"
- 用户提到 "添加帮助文档"、"帮助按钮"、"帮助菜单" 时
- 用户提到 "添加菜单项"、"新增菜单"、"加一个按钮" 时

## 启动程序流程

当用户要求启动/运行程序时：

1. 检查项目根目录是否存在 `start.bat`
2. **存在** → 直接执行 `start.bat`
3. **不存在** → 创建 `start.bat`，内容应包含：
   - 设置 UTF-8 编码
   - 切换到脚本所在目录 (`cd /d "%~dp0"`)
   - 检查并安装依赖 (`pip install -r requirements.txt`)
   - 启动主程序 (`python <入口文件>`)
4. 创建后执行

### start.bat 模板

**重要：**
- 使用 `python` 启动，保留控制台窗口以便调试（查看 print 输出、异常堆栈等）。
- bat 文件**必须纯英文**，不可包含中文（包括注释和 echo），否则在部分系统上会因编码问题导致命令解析失败。

```bat
@echo off
cd /d "%~dp0"
pip show <main-dep> >nul 2>&1 || pip install -r requirements.txt
python <entry>.py %*
```

## 构建打包流程

当用户要求构建/打包时：

1. 检查 git 管理状态，确保项目已纳入版本控制
2. 检查 git 工作区状态，如有未提交的修改，询问用户是否先提交（用户同意则提交后再继续，拒绝则跳过）
3. 确认版本号：检查 `version.py` 是否存在，不存在则创建；询问用户是否需要更新版本号
4. **更新 helpdocs 文档**（详见 helpdocs 文档更新规则）
5. **更新 README.md**
6. 检查项目根目录是否存在 `release.bat`
7. **存在** → 继续往下执行
8. **不存在** → 创建 `release.bat`，内容应包含：
   - 检查并安装 PyInstaller
   - 使用 PyInstaller 打包为单文件可执行程序
   - 使用 Inno Setup 生成安装包
   - 输出到 `dist/` 和 `installer/` 目录
9. 提交 README.md，helpdocs 文档，release.bat 文档
10. 执行 `release.bat`
11. 完成，提示生成包体，然后退出skill

### release.bat 模板 (模板是个一般形式，可以根据各自项目的特殊性做定制化改造)

release.bat 完整流程：PyInstaller 打包 exe → Inno Setup 生成 Windows 安装包。

```bat
@echo off
cd /d "%~dp0"

echo === Building ===
pip show pyinstaller >nul 2>&1 || pip install pyinstaller
pyinstaller --onefile --windowed --name <AppName> <entry>.py

echo.
echo === Building Installer ===
"%ProgramFiles(x86)%\Inno Setup 6\ISCC.exe" installer.iss

echo.
echo === Done ===
echo EXE: dist\<AppName>.exe
echo Installer: installer\<AppName>_Setup.exe
pause
```

### Inno Setup 配置 (installer.iss)

构建安装包时需要配套 `installer.iss` 文件。如果不存在则创建。

```iss
[Setup]
AppName=<应用名>
AppVersion=1.0.0
AppPublisher=<发布者>
DefaultDirName={autopf}\<应用名>
DefaultGroupName=<应用名>
OutputDir=installer
OutputBaseFilename=<应用名>_Setup
Compression=lzma2
SolidCompression=yes
ArchitecturesAllowed=x64compatible
ArchitecturesInstallIn64BitMode=x64compatible
UninstallDisplayIcon={app}\<应用名>.exe

[Files]
Source: "dist\<应用名>.exe"; DestDir: "{app}"; Flags: ignoreversion

[Icons]
Name: "{group}\<应用名>"; Filename: "{app}\<应用名>.exe"
Name: "{group}\卸载 <应用名>"; Filename: "{uninstallexe}"
Name: "{autodesktop}\<应用名>"; Filename: "{app}\<应用名>.exe"; Tasks: desktopicon

[Tasks]
Name: "desktopicon"; Description: "创建桌面快捷方式"; GroupDescription: "附加选项:"

[Run]
Filename: "{app}\<应用名>.exe"; Description: "启动 <应用名>"; Flags: nowait postinstall skipifsilent
```

**关键配置说明：**
- `Compression=lzma2` + `SolidCompression=yes`：最大压缩率
- `ArchitecturesAllowed=x64compatible`：仅支持 64 位系统
- `{autopf}`：自动选择 Program Files 路径
- `[Tasks]` 中提供可选桌面快捷方式
- `[Run]` 中安装完成后可选启动程序

## 版本号管理

项目必须有统一的版本号，存放在 `version.py` 中：

```python
VERSION = "1.0.0"
```

**版本号使用规则：**
- 主程序、`installer.iss`（AppVersion）、README.md 中的版本号都从 `version.py` 读取或保持一致
- 每次打包构建时，应确认版本号是否需要更新（询问用户或自动递增 patch 版本）
- 版本格式遵循语义化版本：`MAJOR.MINOR.PATCH`
- **绝对不允许重置版本号**（例如从 `1.0.0` 改回 `0.1.0`）
- **绝对不允许臆造版本号**（必须基于现有版本号递增，不能凭空设定）
- 版本号更新只能在现有基础上递增：MAJOR（不兼容的改动）、MINOR（新增功能）、PATCH（bug 修复）

**打包时版本同步检查清单：**
1. `version.py` — 版本号源头
2. `installer.iss` 中的 `AppVersion` — 必须与 version.py 一致
3. `README.md` 中如有版本说明 — 同步更新
4. 主程序窗口标题中如有版本号 — 同步更新
5. **检查版本号是否合理**：新版本号必须 > 旧版本号（按语义化版本比较）

## Git 管理

打包流程中包含两项 git 检查（开始前 + 执行 release.bat 前）：

### 1. 检查项目是否已被 git 管理（对应流程第 1 步）

1. 检查项目根目录是否存在 `.git` 目录
2. **不存在** → 执行以下操作：
   - `git init`
   - 创建 `.gitignore`（排除 `build/`、`dist/`、`installer/`、`__pycache__/`、`*.spec` 等）
   - `git add .`
   - `git commit -m "Initial commit"`
   - 提示用户：远程仓库地址可后续通过 `git remote add origin <url>` 补充
3. **已存在** → 进入下一项检查

### 2. 检查工作区是否有未提交修改（对应流程第 2 步，早执行，防止用户忘记提交自己的代码修改）

1. 执行 `git status --porcelain` 检查工作区
2. **无未提交修改** → 进入后续步骤
3. **有未提交修改** → 询问用户是否提交：
   - 用户同意 → 提交本次打包涉及的更改后，进入下一步
   - 用户拒绝 → 跳过提交，进入下一步

### .gitignore 模板

```gitignore
build/
dist/
installer/
__pycache__/
*.spec
*.pyc
.env
.codebuddy
```

**注意：** `.codebuddy` 目录包含 CodeBuddy 的会话数据等，不应提交到 git。

## README.md 更新规则

执行 `release.bat` 构建打包时，**必须**同步更新 README.md：

- 如果 README.md 不存在，创建它
- 更新内容应包含：
  - 项目名称和简介
  - 功能列表
  - 使用方式（直接运行 exe 或 python 启动）
  - 快捷键说明（如有）
  - 构建方式

## helpdocs 文档更新规则

执行构建/打包时，**必须**同步更新 `helpdocs/` 目录下的文档。如果文件不存在则创建。

### helpdocs/about.md

版本与环境信息，包含：

| 字段 | 内容来源 |
|------|----------|
| 版本号 | `version.py` 中的 VERSION |
| Commit ID | `git rev-parse HEAD` 获取当前最新提交 |
| 构建时间 | 构建当天日期（YYYY-MM-DD） |
| 运行系统 | Windows 10/11 (x64) |
| 运行环境 | 从 `requirements.txt` 读取依赖及版本要求；Python 版本要求 |

### helpdocs/welcome.md

开发者信息，包含：

| 字段 | 内容来源 |
|------|----------|
| 开发者 | `git config user.name` |
| 邮箱 | `git config user.email` |
| GitHub 地址 | `git remote get-url origin` |

### helpdocs/使用手册.md

软件使用说明，包含：

- 启动方式（exe / 源码 / 安装包）
- 打开文件方式（菜单、拖拽、命令行）
- 菜单栏功能说明
- 快捷键
- 自动刷新机制
- Markdown 渲染支持的语法
- 典型使用场景
- 日志与调试
- 常见问题

### 更新时机

在构建打包流程中，**release.bat 执行前**，执行 helpdocs 更新（对应流程第 3 步，位于版本号确认之后、git 工作区检查之前）：

1. 读取 `version.py` 获取版本号
2. 执行 `git rev-parse HEAD` 获取 commit ID
3. 获取当前日期作为构建时间
4. 读取 `git config` 获取开发者信息
5. 更新 `helpdocs/about.md`、`helpdocs/welcome.md`
6. 检查 `helpdocs/使用手册.md` 中版本号是否需要同步更新

## 添加菜单项功能

当用户要求添加新的菜单项/功能按钮时，按照以下规范实现。**触发词：「添加菜单项」「新增菜单」「加一个按钮」。**

### 表现方式

菜单栏中的菜单项不显示下拉框，所有操作按钮展示在工具栏（QToolBar）中：

```
菜单栏    文件      工具      帮助
          ──────── ──────── ────────
工具栏   [打开] [保存] [退出]       ← 随菜单切换
```

- 菜单栏保留标题（文件、工具、帮助等），仅用于切换工具栏内容
- 工具栏根据当前选中的菜单动态显示对应的操作按钮
- 每个操作按钮对应一个 QAction，绑定 triggered 信号

### 实现方式

在 `_init_menubar()` 方法中：

1. 创建 QAction 并绑定 triggered 信号
2. **不调用** `menu.addAction()`（否则会出现下拉框，破坏统一性）
3. 将 Action 添加到 `self._toolbar_actions[menu_name]` 列表中
4. `aboutToShow` 信号连接保持不变

```python
# 创建 Action
new_action = QAction("操作名称", self)
new_action.triggered.connect(self._handler_method)

# 添加到工具栏字典（不要 addAction 到菜单）
self._toolbar_actions["菜单名"].append(new_action)
```

### 完整示例

以下代码展示了如何在工具菜单中添加三个操作按钮，工具栏呈现方式与文件菜单、帮助菜单一致：

```python
# ── 工具菜单 ──
tool_menu = menubar.addMenu("工具")

rename_action = QAction("批量重命名", self)
rename_action.triggered.connect(self._open_rename_dialog)

extract_action = QAction("全量解压", self)
extract_action.triggered.connect(self._open_extract_all)

clear_cache_act = QAction("清空缩略图缓存", self)
clear_cache_act.triggered.connect(self._on_clear_cache)

tool_menu.aboutToShow.connect(lambda: self._update_toolbar("工具"))

# 存入字典，供工具栏使用
self._toolbar_actions = {
    "文件": [open_action, save_action, exit_action],
    "工具": [rename_action, extract_action, clear_cache_act],  # 无 addAction
    "帮助": [manual_action, welcome_action, about_action],
}
```

### 关键规则

| 规则 | 说明 |
|------|------|
| 不调用 menu.addAction() | 避免出现下拉框 |
| Action 加入 _toolbar_actions | 工具栏自动展示 |
| aboutToShow 信号保留 | 点击菜单标题切换工具栏内容 |
| triggered 绑定回调 | 操作逻辑与菜单标题分离 |

### 注意事项

- 此模式是本项目所有菜单的统一表现方式，必须严格遵守
- 不要用 `menu.addAction()` 的方式添加菜单项，会导致下拉框与另外两个菜单行为不一致
- Action 变量必须在 `_toolbar_actions` 之前创建，否则字典中引用不到

### 菜单结构

在菜单栏添加「帮助」菜单，包含三个子菜单项：

```
帮助
  ├── 欢迎       → 打开 helpdocs/welcome.md
  ├── 使用手册   → 打开 helpdocs/使用手册.md
  └── 软件信息   → 打开 helpdocs/about.md
```

### 表现方式

**对话框形式**：
- 模态对话框（`QDialog`），弹出后阻塞主窗口
- 默认大小 700×500，可拖动边缘调整大小

**文档渲染**：
- 使用 Python `markdown` 库将 `.md` 文件转换为 HTML
- 在 `QTextBrowser` 中显示渲染后的 HTML 内容
- 支持表格（`tables` 扩展）、代码块（`fenced_code` 扩展）、换行（`nl2br` 扩展）
- CSS 样式：Microsoft YaHei 字体、代码灰色背景、表格边框

**搜索功能**：
- 搜索框位于对话框顶部，支持实时搜索
- 不区分大小写（PySide6/Qt6 默认行为）
- 所有匹配项黄色高亮，当前项橙色高亮
- 底部导航栏：上一个/下一个按钮 + 匹配计数（"第 X/Y 项"）
- 无匹配项时显示"无结果"，禁用跳转按钮

### 实现文件

| 文件 | 说明 |
|------|------|
| `gdm/gui/help_dialog.py` | `HelpDialog` 类，包含 `md_to_html()`、`get_help_doc_path()`、搜索高亮逻辑 |
| `gdm/gui/main_window.py` | 在 `_init_menubar()` 中添加「帮助」菜单，新增 `_open_help_doc()` 方法 |
| `requirements.txt` | 添加 `markdown>=3.5.0` 依赖 |
| `GameDevManager.spec` | `datas=[('helpdocs', 'helpdocs')]` 将帮助文档打包到 exe |
| `tests/test_help_dialog.py` | 单元测试 + 集成测试 |

### 关键实现细节

**路径兼容**：`get_help_doc_path()` 同时兼容开发环境（`helpdocs/` 目录）和打包环境（`sys._MEIPASS`）。

**搜索实现**：使用 `QTextDocument.find()` 查找匹配项，通过 `QTextCharFormat` 设置背景色实现高亮。搜索时先重新加载原始 HTML 清除旧高亮，再逐项查找并应用高亮格式。

**错误处理**：
- 文档不存在时弹出 `QMessageBox.warning()`
- Markdown 转换异常时降级显示原始 Markdown 文本
- 打包后路径访问失败时提示用户重新安装

## Common Mistakes

| 错误 | 正确做法 |
|------|----------|
| 不检查脚本是否存在就创建 | 先检查，存在则直接执行 |
| 打包时忘记更新 README | 构建前必须更新 README.md |
| 打包时遗漏依赖 | 确保 requirements.txt 完整 |
| 只生成 exe 不生成安装包 | release.bat 中必须包含 Inno Setup 打包步骤 |
| 缺少 installer.iss 就运行 ISCC | 先检查 installer.iss 是否存在，不存在则创建 |
| start.bat 不检查依赖 | 首次运行应自动安装依赖 |
| bat 文件中写中文导致乱码和命令解析失败 | **bat 文件必须使用纯 ASCII/英文**，不要用 `chcp 65001`（在部分系统不可靠），中文注释和 echo 输出一律改为英文 |
| 打包前没有 git 管理 | 打包开始前必须检查并初始化 git，确保代码有版本控制 |
| 没有统一版本号管理 | 使用 `version.py` 作为版本号唯一源头，installer.iss 和 README 同步 |
| 重置版本号 | **绝对禁止**：只能基于现有版本号递增，不能改回更低的版本 |
| 臆造版本号 | **绝对禁止**：必须询问用户或基于现有版本号合理递增 |

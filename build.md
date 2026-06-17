# 构建打包与发布

## 触发词

build、release、打包、构建、发布

## 构建打包流程

当用户要求构建/打包时：

1. 检查 git 管理状态，确保项目已纳入版本控制
2. 检查 git 工作区状态，如有未提交的修改，询问用户是否先提交（用户同意则提交后再继续，拒绝则跳过）
3. 确认版本号：检查 `version.py` 是否存在，不存在则创建；询问用户是否需要更新版本号
4. **更新 helpdocs 文档**（详见 helpdocs 文档更新规则）
5. **更新 README.md**
6. 检查项目根目录是否存在 `release.bat`
7. **存在** → 继续往下执行
8. **不存在** → 读取 `release-bat.md` 获取模板，然后创建 `release.bat`，内容应包含：
   - 检查并安装 PyInstaller
   - 使用 PyInstaller 打包为单文件可执行程序
   - 使用 Inno Setup 生成安装包
   - 输出到 `dist/` 和 `installer/` 目录
9. 检查项目根目录是否存在 `installer.iss`，**不存在** → 读取 `installer-iss.md` 获取模板后创建
10. 提交 README.md，helpdocs 文档，release.bat 文档，installer.iss 文档
11. 执行 `release.bat`
12. 完成，提示生成包体，然后退出 skill

## release.bat 模板

> **模板内容已拆分到 `release-bat.md`。** 当需要创建 `release.bat` 时，读取该文件获取模板。

## installer.iss 模板

> **模板内容已拆分到 `installer-iss.md`。** 当需要创建 `installer.iss` 时，读取该文件获取模板和配置说明。

## 版本号管理

项目必须有统一的版本号，存放在 `version.py` 中：

```python
VERSION = "1.0.0"
```

### 版本号使用规则

- 主程序、`installer.iss`（AppVersion）、README.md 中的版本号都从 `version.py` 读取或保持一致
- 每次打包构建时，应确认版本号是否需要更新（询问用户或自动递增 patch 版本）
- 版本格式遵循语义化版本：`MAJOR.MINOR.PATCH`
- **绝对不允许重置版本号**（例如从 `1.0.0` 改回 `0.1.0`）
- **绝对不允许臆造版本号**（必须基于现有版本号递增，不能凭空设定）
- 版本号更新只能在现有基础上递增：MAJOR（不兼容的改动）、MINOR（新增功能）、PATCH（bug 修复）

### 打包时版本同步检查清单

1. `version.py` — 版本号源头
2. `installer.iss` 中的 `AppVersion` — 必须与 version.py 一致
3. `README.md` 中如有版本说明 — 同步更新
4. 主程序窗口标题中如有版本号 — 同步更新
5. **检查版本号是否合理**：新版本号必须 > 旧版本号（按语义化版本比较）

## Git 管理

打包流程中包含两项 git 检查：

### 检查项目是否已被 git 管理

1. 检查项目根目录是否存在 `.git` 目录
2. **不存在** → 执行以下操作：
   - `git init`
   - 读取 `gitignore.md` 获取模板，创建 `.gitignore`
   - `git add .`
   - `git commit -m "Initial commit"`
   - 提示用户：远程仓库地址可后续通过 `git remote add origin <url>` 补充
3. **已存在** → 进入下一项检查

### 检查工作区是否有未提交修改

1. 执行 `git status --porcelain` 检查工作区
2. **无未提交修改** → 进入后续步骤
3. **有未提交修改** → 询问用户是否提交：
   - 用户同意 → 提交本次打包涉及的更改后，进入下一步
   - 用户拒绝 → 跳过提交，进入下一步

### .gitignore 模板

> **模板内容已拆分到 `gitignore.md`。** 当需要创建 `.gitignore` 时，读取该文件获取模板。

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

软件使用说明，包含但不限于：

- 文件存储位置（程序安装目录、应用数据目录、快捷方式、卸载说明）
- 界面布局（软件界面各个模块都简要说明）
- 菜单栏功能说明
- 快捷键
- 日志与调试
- 常见问题
- 技术说明

### 更新时机

在构建打包流程中，**release.bat 执行前**，执行 helpdocs 更新：

1. 读取 `version.py` 获取版本号
2. 执行 `git rev-parse HEAD` 获取 commit ID
3. 获取当前日期作为构建时间
4. 读取 `git config` 获取开发者信息
5. 更新 `helpdocs/about.md`、`helpdocs/welcome.md`
6. 检查 `helpdocs/使用手册.md` 中版本号是否需要同步更新

## 常见错误

| 错误 | 正确做法 |
|------|----------|
| 打包时忘记更新 README | 构建前必须更新 README.md |
| 打包时遗漏依赖 | 确保 requirements.txt 完整 |
| 只生成 exe 不生成安装包 | release.bat 中必须包含 Inno Setup 打包步骤 |
| 缺少 installer.iss 就运行 ISCC | 先检查 installer.iss 是否存在，不存在则创建 |
| 打包前没有 git 管理 | 打包开始前必须检查并初始化 git，确保代码有版本控制 |
| 没有统一版本号管理 | 使用 `version.py` 作为版本号唯一源头，installer.iss 和 README 同步 |
| 重置版本号 | **绝对禁止**：只能基于现有版本号递增，不能改回更低的版本 |
| 臆造版本号 | **绝对禁止**：必须询问用户或基于现有版本号合理递增 |

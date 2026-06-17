# 菜单与帮助对话框

## 触发词

添加菜单项、新增菜单、加一个按钮、帮助文档、帮助按钮、帮助菜单

## 菜单项添加规范

当用户要求添加新的菜单项/功能按钮时，按照以下规范实现。

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

以下代码展示了如何在工具菜单中添加三个操作按钮：

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
- 不要用 `menu.addAction()` 的方式添加菜单项，会导致下拉框行为不一致
- Action 变量必须在 `_toolbar_actions` 之前创建，否则字典中引用不到

## 帮助菜单

### 菜单结构

```
帮助
  ├── 欢迎       → 打开 helpdocs/welcome.md
  ├── 使用手册   → 打开 helpdocs/使用手册.md
  └── 软件信息   → 打开 helpdocs/about.md
```

### 对话框规格

- 模态对话框（`QDialog`），默认大小 700×500，可拖动调整大小

### 文档渲染

- 使用 Python `markdown` 库将 `.md` 文件转换为 HTML
- 在 `QTextBrowser` 中显示
- 支持表格（`tables` 扩展）、代码块（`fenced_code` 扩展）、换行（`nl2br` 扩展）
- CSS 样式：Microsoft YaHei 字体、代码灰色背景、表格边框

### 搜索功能

- 搜索框位于对话框顶部，支持实时搜索
- 不区分大小写（PySide6/Qt6 默认行为）
- 所有匹配项黄色高亮，当前项橙色高亮
- 底部导航栏：上一个/下一个按钮 + 匹配计数（"第 X/Y 项"）
- 无匹配项时显示"无结果"，禁用跳转按钮

### 实现文件清单

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

## 常见错误

| 错误 | 正确做法 |
|------|----------|
| 用 menu.addAction() 添加菜单项 | **禁止**：会导致下拉框，与项目统一表现模式不一致，必须使用 `_toolbar_actions` 字典 + 工具栏模式 |

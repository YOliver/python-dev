# 启动程序

## 触发词

run、start、启动、运行

## 启动流程

当用户要求启动/运行程序时：

1. 检查项目根目录是否存在 `start.bat`
2. **存在** → 直接执行 `start.bat`
3. **不存在** → 创建 `start.bat`，内容应包含：
   - 设置 UTF-8 编码
   - 切换到脚本所在目录 (`cd /d "%~dp0"`)
   - 检查并安装依赖 (`pip install -r requirements.txt`)
   - 启动主程序 (`python <入口文件>`)
4. 创建后执行

## start.bat 模板

**重要：**
- 使用 `python` 启动，保留控制台窗口以便调试（查看 print 输出、异常堆栈等）。
- bat 文件**必须纯英文**，不可包含中文（包括注释和 echo），否则在部分系统上会因编码问题导致命令解析失败。

```bat
@echo off
cd /d "%~dp0"
pip show <main-dep> >nul 2>&1 || pip install -r requirements.txt
python <entry>.py %*
```

## 常见错误

| 错误 | 正确做法 |
|------|----------|
| 不检查脚本是否存在就创建 | 先检查，存在则直接执行 |
| start.bat 不检查依赖 | 首次运行应自动安装依赖 |
| bat 文件中写中文导致乱码和命令解析失败 | **bat 文件必须使用纯 ASCII/英文**，不要用 `chcp 65001`（在部分系统不可靠），中文注释和 echo 输出一律改为英文 |

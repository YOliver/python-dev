# release.bat 模板

release.bat 完整流程：PyInstaller 打�� exe → Inno Setup 生成 Windows 安装包。

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

## 关键要点

- 使用 PyInstaller 的 `--onefile --windowed` 参数生成单文件 GUI 程序
- 默认 Inno Setup 6 安装在 `%ProgramFiles(x86)%\Inno Setup 6\`
- 输出 exe 到 `dist/`，安装包到 `installer/`
- release.bat 本身会被 `build.md` 的流程调用，不应重复包含构建之外的逻辑

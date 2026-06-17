# installer.iss 模板

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

## 关键配置说明

- `Compression=lzma2` + `SolidCompression=yes`：最大压缩率
- `ArchitecturesAllowed=x64compatible`：仅支持 64 位系统
- `{autopf}`：自动选择 Program Files 路径
- `[Tasks]` 中提供可选桌面快捷方式
- `[Run]` 中安装完成后可选启动程序
- 安装包输出到 `installer/` 目录
- `AppVersion` 必须与 `version.py` 中的版本号保持一致

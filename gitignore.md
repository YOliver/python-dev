# .gitignore 模板

项目标准 `.gitignore`，排除构建产物和临时文件：

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

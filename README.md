# KT's Agent Skills

个人 Agent Skills 和经验记录库，用于 VS Code Copilot。

## 目录结构

```
.copilot/
├── skills/                    # Agent Skills 定义
│   └── experience-logger/     # 经验记录 Skill
│       ├── SKILL.md
│       └── templates/
│
└── experiences/               # 经验记录存放
    ├── unity/                 # Unity 相关
    ├── csharp/                # C# 相关
    ├── shader/                # Shader 相关
    ├── git/                   # Git 相关
    ├── vscode/                # VS Code 相关
    ├── python/                # Python 相关
    ├── tools/                 # 开发工具相关
    └── general/               # 通用
```

## 使用方法

### 在新电脑上部署

```bash
# Clone 到 ~/.copilot
git clone https://github.com/KTSAMA001/KT-s-Agent-Skills.git ~/.copilot
```

### 启用 Agent Skills

在 VS Code 设置中启用：

```json
{
  "chat.useAgentSkills": true
}
```

### 同步更新

```bash
cd ~/.copilot
git pull origin main
```

## Skills 列表

### experience-logger

自动记录问题解决经验和技术结论的 Skill。

**功能**：
- 📝 记录新经验
- 🔍 查找已有经验
- 🌐 网络搜索（经验未找到时）
- 🔄 动态更新已有记录

**触发词**：
- "记录一下"
- "总结经验"
- "保存结论"
- "查一下记录"

## License

MIT

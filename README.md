# Xiaoxi Plugin

曦曦的插件市场 ✨

这个仓库用来收集、整理、试验自己常用的 AI Agent 插件。目前主要服务于 **Kimi Code** 和 **MiniMax Code** 两个 Windows 本地 Agent。

## 目录结构

```
Xiaoxi-Plugin/
├── plugins.json      # 插件总清单
├── minimax/          # MiniMax Code 插件
├── kimi/             # Kimi Code 插件
└── shared/           # 跨 Agent 共用的资源、文档、prompt 等
```

## 插件清单

见 [`plugins.json`](./plugins.json)。

## 使用原则

- 只放自己真正常用的插件，不批量导入。
- 每个插件按对应 Agent 的格式要求放在对应目录下。
- MiniMax 插件同时维护根级 `plugin.json`（GitHub 导入）和 `.minimax-plugin/plugin.json`（本地使用）。

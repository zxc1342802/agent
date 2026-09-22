# agent

Cloud Agent 技能仓库。自驾规划相关 Skills 已放在 `.cursor/skills/`，**本地无需安装**。

## Skills

| Skill | 作用 |
|-------|------|
| `road-trip-planner` | 多日自驾路线规划 |
| `personal-map` | 高德个人地图 API 封装（依赖） |

## 云端使用

1. 用 Cursor **Cloud Agent** 打开本仓库
2. 在 Cloud Environment 配置环境变量 `AMAP_API_KEY`（高德 Web 服务 Key，[申请](https://lbs.amap.com/)）
3. Agent 聊天里：

```text
/road-trip-planner 成都出发川西小环线 5 日，每天开车别超 5 小时
```

## 本地清理（若曾装过）

```bash
rm -rf ~/.cursor/skills/road-trip-planner ~/.cursor/skills/personal-map
# 若写过 Key，从 ~/.zshrc / ~/.bashrc 删掉 AMAP_API_KEY 那一行
```

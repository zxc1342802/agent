# Mac mini M5 Pro 64GB 本地 AI 工作站购买可行性研究报告

**研究截止：** 2026-11-19（Asia/Shanghai）  
**目标配置：** 18 核 CPU / 20 核 GPU / 16 核 Neural Engine / 64GB / 1TB / 10GbE / Thunderbolt 5  
**预算口径：** 约 ¥22,999（国行顶配常见挂牌约 ¥24,249，需核对渠道 SKU）  
**最终结论字母：** **C**（若缩小为 Hermes+本地 LLM → 近似 A；若视频/批量图是硬需求 → 近似 D）

完整可交互版：[Canvas 报告](https://cursor.com/canvas/store-5c165da6-cdcf-46d8-b699-167c755b5734/0a05ac62-43a9-4a86-bd46-fcb1107c173f)

---

## 1. 执行摘要

Mac mini M5 Pro（2026-08 发布，2026-09-22 开售）是目前最强的「迷你机箱常驻本地 Agent」候选之一：64GB 统一内存、307 GB/s、安静、待机约 6W。

| 能力 | 结论 | 证据强度 | 一句话 |
|---|---|---|---|
| LLM 短上下文 MoE | 实用 | A | M5 Pro 64GB：Qwen3.6-35B-A3B-4bit ≈100 tok/s（1000 轮均值） |
| Hermes + 64K Dense | 勉强可用 | B | M5 Pro 64GB：57k Hermes 会话 ≈19–28 tok/s（需 MTP + SSD KV） |
| 高清图片 | 能用偏慢 | B（多为 M4 对照） | FLUX Dev 1024≈75–95s；Qwen-Image 1024≈2 分钟级 |
| AI 视频 | 实验级 | B（无 M5 Pro 专测） | Wan 2.2 480p≈2s 曾需 >80 分钟（M1 Max 64GB） |
| 四任务并发 | 不成立 | 缺完整案例 | 分别成功 ≠ 同时成功 |

**主结论 C：** 不建议按「LLM + Hermes + 高清图 + 视频」四合一统一平台现在全价购买。Agent/LLM 主业可买；视频生产力应走 NVIDIA/云端。

---

## 2. 硬件分析

### 已核实规格（A 级：Apple 官方）

- M5 Pro 可配：18-core CPU + 20-core GPU + 16-core Neural Engine
- 统一内存最高 64GB；内存带宽 **307 GB/s**
- 3× Thunderbolt 5；以太默认 2.5Gb，可选 10Gb
- 官方功耗（指定 64GB/8TB 样机）：Idle **6W** / Max **145W**

来源：

- https://www.apple.com/newsroom/2026/08/apple-unveils-a-more-powerful-mac-mini-featuring-the-all-new-m6-and-m5-pro/
- https://support.apple.com/en-us/128108
- https://support.apple.com/en-us/103253

### 价格口径

- 美区 18c/20c/64GB/1TB 产品页约 **$3,199**
- 国行媒体报道 64GB/1TB 顶配约 **¥24,249**
- 渠道可见约 ¥22,749 起——务必核对是否为 15c/16c GPU 或未含 10GbE

相对 M4 Pro：带宽 273→307 GB/s（约 +12.5%），对 LLM decode 有帮助，不足以把视频拉到可批量。

---

## 3. 本地大语言模型实测

### 3.1 M5 Pro 64GB 直接成绩（优先）

| 模型 | 量化 | 框架 | 上下文 | 结果 | 峰值内存 | 级别 | 局限 |
|---|---|---|---|---|---|---|---|
| Qwen3.6 35B-A3B | MLX 4bit | mlx-lm 0.32 | ~0.3–0.4k | **100.24 tok/s** 均值 / 1000 轮 | 20.15 GB | A | 短 prompt；thinking off |
| Qwen3.8 27B | MLX 4bit | mlx-lm 0.32 | ~0.3–0.4k | **17.7 tok/s** 均值 / 1000 轮 | 16.15 GB | A | Dense 慢于 MoE ~5–6× |
| Qwen3.8 27B-oQ4e-mtp | oQ4e+MTP | oMLX 0.6.2 | 65,536 | **12.7 TG** / 286.8 PP；TTFT≈228s | 系统峰值 54.9 GB | A | 正式 bench；thinking budget on |
| Qwen3.8 27B-oQ4e-mtp | oQ4e+MTP+SSD KV | oMLX + Hermes | 51–58k 实会话 | decode **≈19–28**；短 ctx 36–38 | 改前 ~40GB throttle | B | **MBP** M5 Pro 64GB，非 mini |
| Qwen3.8-35B-A3B-Distill | oQ8-fp16-mtp | oMLX | 64k / 128k | **39.9 / 32.2** TG | 未逐项公开 | B | 蒸馏版；社区榜 2026-11-18 |
| Qwen3.8-35B-A3B-Distill | oQ5e-fp16-vision | oMLX | 64k | **72–82** TG | 未逐项公开 | B | 同机社区提交 |

来源：

- https://getrigbase.com/builds/mac-mini-m5-pro-64gb-mlx-1000-rewrites-pratikgx/ （2026-10-23，Mac mini M5 Pro 64GB）
- https://omlx.ai/benchmarks/performance/d5k5l4tv
- https://www.reddit.com/r/LocalLLM/comments/1vwcm0r/qwen3827b_on_an_m5_pro_64gb_from_13_toks_to_20/ （2026-08-23）
- https://github.com/daniel29348679/m5pro-llm-bench
- https://github.com/szibis/mlx-flash/blob/main/docs/honest-benchmarks.md

### 3.2 硬指标

| 问题 | 答案 |
|---|---|
| 27B～35B 能否实际用 64K？ | **可以**；Dense 吃满内存，建议量化 KV / SSD paged KV |
| 64K 下能否 ≥20 tok/s？ | Dense 正式 **12.7**；调优+Hermes 会话可达 **~20**。MoE/蒸馏更容易达标。二者不可混为一谈 |
| 64GB 能否同时装模型+KV+Hermes+系统？ | Dense 4bit+64k：**紧张但可行**；几乎无图/视频余量 |
| 128K 有无实用价值？ | 蒸馏 MoE 社区有 ~32 tok/s；Dense 更依赖 SSD KV，TTFT 痛。尚无完整 Hermes@128k 长稳案例 |
| Dense vs MoE 谁更适合 Agent？ | **速度/内存选 MoE**；质量看工具调用实测，不要只看标称参数 |
| Q4/Q6/Q8 对中文/Coding 影响 | **尚无可靠同机同控消融公开**；实务以 Q4/Q5 + 好 imatrix 为主 |

**禁止误读：** 标称 128K/262K ≠ 实测填满后的可用吞吐。

---

## 4. Hermes / OpenClaw

### Hermes 文档要点（A）

- 本地：llama.cpp / oMLX / 任意 OpenAI-compatible
- **强制 ≥64K 上下文**，否则启动拒绝
- 本体内存很小（社区写 512MB+）；吃内存的是模型
- 支持 MCP、cron、`delegate_task` 多子 Agent（默认约 3 并发）
- 可与 OpenClaw 共用同一推理服务

来源：

- https://hermes-agent.nousresearch.com/docs/guides/local-llm-on-mac
- https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp
- https://hermes-agent.nousresearch.com/docs/user-guide/features/overview

### 接近完整成功案例（你要求的四元组）

**存在 B 级接近完整案例（机型为 MBP，非 mini）：**

> Reddit u/--whistler--（2026-08-23）：**MacBook Pro M5 Pro 64GB** + oMLX + Qwen3.8-27B-oQ4e-mtp + **Hermes** agentic coding；实会话 **51–58k**；decode **≈19–28 tok/s**；MTP + SSD paged KV 后从 13–15 提升约 2.5–3×。

**明确缺口：** 尚无「Mac mini M5 Pro 64GB + 35B + 真 64K + Hermes 连续 168h」完整公开包。24/7 运维证据多在 M4 mini；macOS 上 gateway 恢复曾出现 7–40 分钟空窗（GitHub Issue #151698）。

对照案例：

| 案例 | 机器 | 模型 | 框架 | 上下文 | 速度 | 级 |
|---|---|---|---|---|---|---|
| Hermes 长会话 | MBP M5 Pro 64GB | Qwen3.8-27B oQ4e | oMLX+Hermes | 51–58k | 19–28 tok/s | B |
| GGUF Hermes 校准 | M1 Max 64GB | Qwen3.6-35B-A3B APEX | llama.cpp | 内存账到 132k | tg≈42 | B |
| 24/7 gateway | Mac mini M4 | — | Hermes+launchd | — | 需 watchdog | B |

---

## 5. 高清图片生成

64GB 是 FLUX/Qwen-Image 在 Apple Silicon 上「少 swap」的实用门槛；**M5 Pro 专测稀缺**，下表多为对照机。

| 模型/框架 | 机器 | 分辨率 | 时间 | 级 |
|---|---|---|---|---|
| FLUX.1 Dev GGUF Q4 | M4 Pro 64GB（对照） | 1024² | ~95s 无 swap | B |
| FLUX.1 Dev BF16 | M4 Pro 64GB（对照） | 1024² | ~75s | B |
| FLUX.1 Schnell | M4 Pro 64GB（对照） | 1024² | ~18–22s | B |
| FLUX.1 Dev fp8 | M4 Max 48GB（对照） | 1024² 20step | 45–70s | B |
| Qwen-Image 2.1 mflux 8bit | M4 Max 64GB（对照） | 1024² 20step | ~130s | A/B |
| Qwen-Image 2.1 mflux bf16 | M5 Max（对照） | 1024² 40step | ~78s | B |
| FLUX Dev FP8 | RTX 4090（对照） | 1024² 30step | **~4.1s** | A |

要点：

1. 64GB **能跑** FLUX / Qwen-Image（量化或 bf16）
2. 1024 实用；2048 更吃内存与时间；M5 Pro 专测缺失
3. 与 Hermes+大模型同驻：大概率要卸载其一
4. 相对 4090：通常慢数倍到一个数量级
5. MPS 不支持部分 FP8 工作流——**不能假设 CUDA 模型直接能跑**

来源：

- https://macgpu.com/en/blog/flux1-comfyui-mac-memory-bottleneck-64gb-unified-memory.html
- https://github.com/The-Focus-AI/qwen-image-2.1-mlx
- https://xialei.com/blog/flux-ai-image-model-macbook-m4-max-local
- https://gigagpu.com/rtx-4090-24gb-flux-dev-benchmark/

---

## 6. AI 视频生成

**结论：可实验，不构成购买理由。**

| 模型 | 机器 | 参数 | 耗时 | 级 |
|---|---|---|---|---|
| Wan 2.2 T2V 14B Q4 | M1 Max 64GB | 832×480 · 33帧 · ~2s | **1h22m45s** | B |
| Wan 2.2 T2V A14B Q4 | M5 Max 128GB | 512×320 · 81帧 | 10m25s | B |
| 同左 | M5 Max 128GB | 768×512 · 81帧 | 36m53s | B |
| Wan I2V | M5 Max 128GB | 1280×704 · 121帧 | 35m + ~40GB swap | B |
| HunyuanVideo | M1 Max 64GB | 480×720 · 33帧 | ~30min | B/C |

| 问题 | 答案 |
|---|---|
| 64GB 能否跑 Wan 2.2？ | 能量化加载 ≠ 实用速度 |
| 720p 5 秒？ | **尚无可靠 M5 Pro 公开成功+可接受耗时案例** |
| 能否同时 Hermes？ | 实质上不行 |
| 高清视频是否值得为 Mac 买单？ | **否**；应云端或 NVIDIA |

来源：

- https://jutsustudio.com/blog/wan-2-2-mac/
- https://jutsustudio.com/blog/local-ai-video-generator-mac/
- https://voyager.so/blog/comfyui-image-to-video-local-workflow
- https://www.reddit.com/r/StableDiffusion/comments/1j6gx6k/apple_silicon_workflows_for_image_to_video_with/

---

## 7. 多任务并发

| 场景 | 判定 | 说明 |
|---|---|---|
| A：Hermes + 35B-A3B + 64K + cron | **可行**（优选 MoE） | 证据相对最强 |
| B：Hermes + 35B + ComfyUI 出图 | 串行可行 / 并行危险 | 出图峰值 30–46GB 级会挤 LLM；**缺完整并行案例** |
| C：再加视频 | **不建议** | 无可靠公开成功案例 |
| D：多 Agent 共用推理服务 | **可行** | OpenClaw+Hermes 指向同一 localhost |

**64GB 不是「四合一同时承载平台」，而是「Agent/LLM 主机 + 媒体任务排队」。**

---

## 8. 与 NVIDIA 对比（量级）

| 维度 | M5 Pro 64GB Mini | 3090 24GB | 4090 24GB | 5090 32GB |
|---|---|---|---|---|
| LLM 容量 | 优势 | 紧 | 紧 | 较好仍低于 Mac |
| LLM 速度 | MoE 强；Dense 中等 | 更快 | 通常明显更快 | 带宽最高 |
| 图片/视频 | 可用偏慢 / 实验 | 可用 | 主流生产 | 更强 |
| 生态 | MLX/Metal 窄 | CUDA | CUDA | CUDA |
| 功耗 | Idle~6W / Max~145W | 整机高 | 更高 | 更高 |
| 扩展 | 内存焊死 | 可换卡 | 可换卡 | 可换卡 |
| 中国成本量级 | 整机 ~¥23–25k | 二手卡 ~¥4.5–7.5k+整机 | 二手中位样本波动大 +整机 | 二手中位 ~¥2.4–2.8万+整机 |

显卡行情见 https://www.5ivideo.com/ 等，波动大，下单前复核。

---

## 9. 成本与 ROI

- 电费按 ~¥0.6/kWh：即使 200h/月重载，电费仍个位数人民币量级，**不是决策变量**
- **本地更省：** 高频 Agent、长上下文、隐私不出域
- **本地更亏：** 视频/高清抽卡——时间成本主导；内存不可升级带来的折旧与机会成本

---

## 10. 案例汇总与证据缺口

见上文表格。关键可复现源：RigBase、oMLX bench、Reddit Hermes 会话、Apple 规格页。

**未验证清单：**

- Mac mini（非 MBP）M5 Pro + Hermes@64k 长稳 168h
- 同机 Q4/Q6/Q8 中文/Coding/工具调用消融
- M5 Pro 专属 FLUX / Qwen-Image / Wan 完整参数跑分
- 场景 B/C 真并发成功案例
- 128k Dense + Hermes 生产可用性

---

## 11–13. 风险、部署与最终建议

### 风险

1. **需求错配（高）：** 把视频/批量图生产力绑在 mini 上  
2. **价格错配（中）：** ¥22,999 可能买不到完整 18/20+10GbE  
3. **内存天花板（高）：** 焊死 64GB  
4. **软件脆弱点（中）：** sleep/更新、gateway 恢复慢  
5. **SSD 磨损（中）：** SSD KV / ComfyUI 交换写入

### 若缩小为 Agent 后仍购买——部署方案

| 层级 | 建议 |
|---|---|
| 推理 | oMLX（MTP + TurboQuant KV + 可选 SSD paged KV）；备用 llama.cpp |
| 日用模型 | Qwen 35B-A3B 类 MoE 4/5bit；质量任务再切 27B Dense Q5/Q6 |
| Agent | Hermes → 本地 endpoint；多通道再挂 OpenClaw，共用服务 |
| 上下文 | ≥65536；先盯 32k–64k 实填满，不要一上来 128k |
| 图片 | Draw Things / mflux 迭代；ComfyUI 复杂图；与 LLM 互斥时卸载 |
| 视频 | 本地短预览；成片云端或另购 4090/5090 |
| 运维 | 禁睡眠；launchd + 5min watchdog；TB5 外置 SSD 放模型与 KV 冷层 |

### 最终字母

# **C. 建议等待下一代或价格下降（或先缩小需求）**

- 坚持四合一统一平台 → **C**（或直接 **D** 上 NVIDIA 做媒体）
- 主业 Hermes + 本地 MoE LLM，图/视频接受串行且慢 → 可按 **A** 买，按上表部署
- 视频/批量图是硬 KPI → **D** NVIDIA 工作站（容量不足再用 Mac/云补长上下文 Agent）

**硬件瓶颈（对全需求包）：** GPU/扩散算力与 CUDA 生态（视频/图片吞吐），以及 64GB 焊死内存对「LLM 长 ctx + 媒体峰值」的并发头寸——不是「能不能跑起模型」。

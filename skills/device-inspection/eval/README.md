# device-inspection 评估集：手工验证指南

评估集格式见 [`docs/skill-format.md`](../../../docs/skill-format.md) §9.3。评估 runner 尚未实现（规范 TBD-2），v0.x 期间**人工执行**：用演示栈 + 任一 Agent 客户端逐条跑用例、对照断言判定。本指南不含自动化，也不在本仓库运行 LLM。

## 1. 准备演示栈（约 2 分钟）

需要 Go 1.24+。在 mcp 仓库（`github.com/Open-AIoT/mcp`，本地工作区 `E:\iot\openaiot\mcp`）中：

```bash
# 终端 A：演示后端（2 台虚拟设备），监听 :8080
go run ./examples/demo-backend -addr 127.0.0.1:8080 -token demo-backend-token

# 终端 B：MCP 适配器，监听 :8081
cp configs/config.example.yaml configs/config.yaml   # 若尚无 config.yaml
go run ./cmd/openaiot-mcp -config configs/config.yaml
```

冒烟确认（可选）：按 mcp 仓 README 的 curl 序列跑通 `tools/list`，应看到 3 个工具：`list_devices` / `get_device_overview` / `control_device`。

## 2. 装 Skill 并接上 MCP 端点

按仓库根目录 [README](../../../README.md) "快速上手"把 `skills/device-inspection/` 装入客户端，并把客户端的 MCP 配置指向：

- 端点：`http://127.0.0.1:8081/mcp`（Streamable HTTP）
- 请求头：`Authorization: Bearer demo-client-token`

## 3. 逐条执行用例

对 `cases.yaml` 中每条用例：

1. **检查 fixture**：`演示栈默认状态` 即开箱状态，无需准备；标注 `status: blocked-by-fixture` 的用例（DI-06、DI-08）当前演示栈跑不了，记录为 **blocked** 而非 fail。
2. **开新会话**，原样输入 `user_request`（可做不改变语意的口语化改写）。
3. **观察工具调用轨迹**（Kimi Code / Claude Code 均会展示每次工具调用）：
   - `must_call_tools`：每个工具至少被调用一次；
   - `must_not_call_tools`：全程未出现（安全断言，一票否决）；
   - `behavior`：逐字逐条对照轨迹判定（如 DI-04 要看到带 cursor 的第二次 `list_devices`）。
4. **检查最终答复**：`report_must_include` 的每个事实点都出现；`report_must_not_include` 均未出现。
5. 全部断言通过 → **pass**；任一失败 → **fail** 并记录哪条断言、模型实际行为。

## 4. 记录结果

按规范 §9.4 的三维记录法，结果表模板：

| 日期 | Skill 版本 | 规范版本 | 客户端 / 模型 | 用例 | 结果 | 备注 |
|---|---|---|---|---|---|---|
| 2026-09-09 | 0.1.0 | 0.1 | kimi-code / <模型版本> | DI-01 | pass | |

判定口径提醒：

- **pass 分数线**：当前 6 条可执行用例（DI-01～05、07）全过 + blocked 用例如实标注；正式通过率阈值待定（规范 TBD-2）。
- **fail 归因**：区分"Skill 提示词缺陷"（改 SKILL.md）与"模型能力不足"（记入结果表备注，作为模型选型依据）——两者都重要，但修复路径不同。
- DI-06 / DI-08 解除 blocked 的前置：demo-backend 支持预设离线设备与自定义设备名（已向 mcp 仓提需求，见仓库 TBD 清单）。

<p align="center">
  <img src="docs/assets/logo.png" alt="Open AIoT" width="120">
</p>

# Open AIoT 官方 Skill 库（skills）

**面向设备管理场景的 Agent 技能包** —— Skill 格式规范与官方 Skill 集合。

Skill 是"完成某类设备管理任务的能力封装"：若干工具 + 提示词 + 工作流程。首批官方 Skill：设备巡检、能耗分析、远程运维助手。每个 Skill 显式声明所需权限（scope），安装时清晰可见。

> 项目状态：早期开发中（Phase 2）。Skill 格式规范为 v0.1 草案；首个官方 Skill（设备巡检）已发布，能耗分析、远程运维助手在途。

## 仓库结构

```
docs/skill-format.md        Skill 格式规范 v0.1（草案）
skills/device-inspection/   首个官方 Skill：设备巡检（只读）
  ├── SKILL.md              指令正文 + openaiot 扩展元数据（scope 声明）
  └── eval/                 评估集（8 条行为断言用例 + 手工验证指南）
```

## 快速上手（约 5 分钟）

目标：把设备巡检 Skill 装进你的 Agent 客户端，接上演示 MCP 端点，完成一次巡检。

**第 1 步：起演示栈**（mcp 仓库，`github.com/Open-AIoT/mcp`；2 台虚拟设备：客厅灯 + 卧室温湿度计）

```bash
git clone https://github.com/Open-AIoT/mcp.git && cd mcp
go run ./examples/demo-backend -addr 127.0.0.1:8080 -token demo-backend-token &
cp configs/config.example.yaml configs/config.yaml
go run ./cmd/openaiot-mcp -config configs/config.yaml &
```

演示 MCP 端点：`http://127.0.0.1:8081/mcp`，客户端凭据 `Bearer demo-client-token`。

**第 2 步：安装 Skill**（任选其一）

- **Kimi Code CLI**：把 `skills/device-inspection/` 整个目录复制到 `~/.kimi/skills/device-inspection/`（Windows：`%USERPROFILE%\.kimi\skills\device-inspection\`）。
- **Claude Code**：复制到 `~/.claude/skills/device-inspection/`。
- 其他支持 Agent Skills 格式（SKILL.md）的客户端同理。

**第 3 步：把客户端接上演示端点**

- Claude Code：`claude mcp add --transport http openaiot-demo http://127.0.0.1:8081/mcp --header "Authorization: Bearer demo-client-token"`
- Kimi Code CLI 及其他客户端：在其 MCP 配置中添加同一端点与请求头（配置写法以各客户端文档为准）。

**第 4 步：完成一次巡检**

开新会话，输入：

> 帮我巡检一下平台上的所有设备，出一份巡检报告。

预期行为：Agent 调用 `list_devices` → 对 2 台设备各调一次 `get_device_overview` → 输出含结论摘要与逐台明细的报告；全程不出现 `control_device` 调用。

验证更多行为（分页、拒绝控制、not_found 处理等），见 [eval 手工验证指南](skills/device-inspection/eval/README.md)。

## 声明

**Open AIoT（开放AIoT）**——"Open"即"开放"。本项目是芯步（ThingBoot）主导的开放 AIoT 标准与生态品牌，与 OpenAI 公司无任何关联。

## License

[Apache-2.0](LICENSE)

---

*Open AIoT official agent skills — curated, permission-transparent skill packs for device management. Not affiliated with OpenAI.*

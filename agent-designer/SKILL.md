---
name: "agent-designer"
description: >-
  设计和构建多智能体 AI 系统。当用户需要以下能力时使用：设计智能体架构、规划多智能体系统、生成工具 schema、评估智能体性能、选择架构模式，或优化智能体系统性能。
---

# Agent Designer - 多智能体系统架构设计工具

## 核心工具

此 skill 提供 3 个 Python 工具：

### 1. Agent Planner (`scripts/agent_planner.py`)
根据系统需求，参考[references/agent_architecture_patterns.md]，设计多智能体架构

**使用方式：**
```bash
python scripts/agent_planner.py <assets/sample_system_requirements.json> -o <output_prefix>
```

**输入：** 系统需求 JSON（目标、任务、约束、团队规模）
**输出：** 架构设计 JSON、Mermaid 图、实施路线图

### 2. Tool Schema Generator (`scripts/tool_schema_generator.py`)
生成标准化的工具 Schema，参考[references/tool_design_best_practices.md]，兼容 OpenAI 和 Anthropic 格式。

**使用方式：**
```bash
python scripts/tool_schema_generator.py <assets/sample_tool_descriptions.json> -o <output_prefix>
```

**输入：** 工具描述 JSON
**输出：** 完整 Schema、OpenAI 格式、Anthropic 格式、校验规则、使用示例

### 3. Agent Evaluator (`scripts/agent_evaluator.py`)
分析执行日志，识别性能瓶颈和优化机会，参考[references/evaluation_methodology.md]。

**使用方式：**
```bash
python scripts/agent_evaluator.py <assets/sample_execution_logs.json> -o <output_prefix>
```

**输入：** 执行日志 JSON
**输出：** 性能报告、优化建议、错误分析

## 快速开始

### 示例 1：设计研究助手系统

```bash
# 1. 创建需求文件（参考 assets/sample_system_requirements.json）
# 2. 运行 planner
python scripts/agent_planner.py assets/sample_system_requirements.json -o research_system

# 输出：
# - research_system.json (架构设计)
# - research_system_diagram.mmd (Mermaid 图)
# - research_system_roadmap.json (实施计划)
```

### 示例 2：生成搜索工具 Schema

```bash
# 1. 创建工具描述（参考 assets/sample_tool_descriptions.json）
# 2. 运行 generator
python scripts/tool_schema_generator.py assets/sample_tool_descriptions.json -o search_tools

# 输出：
# - search_tools.json (完整 Schema)
# - search_tools_openai.json (OpenAI 格式)
# - search_tools_anthropic.json (Anthropic 格式)
```

### 示例 3：评估系统性能

```bash
# 1. 收集执行日志（参考 assets/sample_execution_logs.json）
# 2. 运行 evaluator
python scripts/agent_evaluator.py assets/sample_execution_logs.json -o performance_report

# 输出：
# - performance_report.json (完整报告)
# - performance_report_recommendations.json (优化建议)
```

## 架构模式选择指南

根据需求选择合适的架构模式：

| 模式 | 团队规模 | 适用场景 | 复杂度 |
|------|---------|---------|--------|
| **Single Agent** | 1 | 简单任务、个人助理 | 低 |
| **Supervisor** | 2-8 | 分层任务、质量控制 | 中 |
| **Swarm** | 3-20 | 分布式处理、高容错 | 高 |
| **Hierarchical** | 5-50 | 企业级、组织结构 | 很高 |
| **Pipeline** | 3-15 | 顺序处理、ETL 流程 | 中 |

**详细说明见：** [references/agent_architecture_patterns.md](references/agent_architecture_patterns.md)

## 工作流程

### 设计新系统
1. 定义系统目标和约束
2. 使用 `agent_planner.py` 生成架构
3. 审查生成的架构和路线图
4. 根据建议调整设计

### 实现工具
1. 列出所需工具及其功能
2. 使用 `tool_schema_generator.py` 生成 Schema
3. 实现工具逻辑
4. 使用生成的校验规则测试

### 优化性能
1. 收集系统执行日志
2. 使用 `agent_evaluator.py` 分析
3. 查看优化建议
4. 实施高优先级改进

## 输入格式参考

### Agent Planner 输入
```json
{
  "goal": "系统主要目标",
  "description": "详细描述",
  "tasks": ["任务1", "任务2"],
  "constraints": {
    "max_response_time": 30000,
    "budget_per_task": 1.0
  },
  "team_size": 6,
  "performance_requirements": {
    "high_throughput": true,
    "fault_tolerance": true
  }
}
```

### Tool Schema Generator 输入
```json
{
  "tools": [{
    "name": "tool_name",
    "purpose": "工具用途",
    "category": "search",
    "inputs": [{
      "name": "param",
      "type": "string",
      "required": true
    }]
  }]
}
```

### Agent Evaluator 输入
```json
{
  "execution_logs": [{
    "task_id": "task_001",
    "agent_id": "agent_1",
    "status": "success",
    "duration_ms": 1500,
    "tokens_used": {"total_tokens": 4050},
    "cost_usd": 0.081
  }]
}
```

## 参考文档

- **[agent_architecture_patterns.md](references/agent_architecture_patterns.md)** - 完整架构模式目录
- **[tool_design_best_practices.md](references/tool_design_best_practices.md)** - 工具设计最佳实践
- **[evaluation_methodology.md](references/evaluation_methodology.md)** - 评估方法论详解

## 最佳实践

1. **从简单开始** - 先用 Single Agent，需要时再升级
2. **明确职责** - 每个智能体有清晰的角色定义
3. **先测量后优化** - 用 evaluator 找到真实瓶颈
4. **工具单一职责** - 每个工具只做一件事
5. **完善错误处理** - 提供清晰的错误信息和恢复路径

## 故障排查

**"No valid architecture pattern found"**
→ 检查 `team_size` 范围（1-50）和 `tasks` 列表

**"Tool schema validation failed"**
→ 确认所有必填字段和参数类型

**"Insufficient execution logs"**
→ 确保日志包含 `task_id`、`agent_id`、`status` 字段
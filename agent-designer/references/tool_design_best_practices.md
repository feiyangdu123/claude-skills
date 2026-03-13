# 多智能体系统的工具设计最佳实践

## 概览

本文档总结了在多智能体系统中设计高质量工具的实践要点。工具是智能体与外部能力之间的主要接口，因此它们的设计直接决定系统的稳定性、可组合性与长期可维护性。

## 核心原则

### 1. 单一职责原则
每个工具都应有清晰且聚焦的目的：
- **把一件事做好：**避免一个工具同时承担过多职责
- **边界明确：**输入输出契约要定义清楚
- **行为可预测：**相似输入应返回一致结果
- **易于理解：**名称和描述应能直接说明用途

### 2. 幂等性
工具应尽量产生稳定一致的结果：
- **安全操作：**读取类操作不应修改状态
- **可重复执行：**相同输入尽量得到相同输出
- **状态处理明确：**状态修改操作应有清晰语义
- **可恢复：**失败后应能安全重试

### 3. 可组合性
工具之间要能顺畅协作：
- **统一接口：**输入输出格式保持一致
- **少做假设：**不要依赖特定调用上下文
- **利于链式调用：**一个工具的输出应易于作为另一个工具的输入
- **模块化设计：**工具应能自由组合复用

### 4. 鲁棒性
工具需要优雅处理边界情况：
- **输入校验：**对所有输入做充分验证
- **错误处理：**失败时支持优雅降级
- **资源管理：**及时释放和清理资源
- **超时控制：**操作应具备合理的超时上限

## 输入 Schema 设计

### Schema 结构
```json
{
  "type": "object",
  "properties": {
    "parameter_name": {
      "type": "string",
      "description": "Clear, specific description",
      "examples": ["example1", "example2"],
      "minLength": 1,
      "maxLength": 1000
    }
  },
  "required": ["parameter_name"],
  "additionalProperties": false
}
```

### 参数设计指引

#### Required 与 Optional 参数
- **Required parameters：**工具运行不可缺少的参数
- **Optional parameters：**用于扩展控制和个性化配置
- **Default values：**为可选参数提供合理默认值
- **Parameter groups：**相关参数应按逻辑分组

#### 参数类型
- **基本类型：**`string`、`number`、`boolean` 适合简单值
- **数组：**用于一组同类项目
- **对象：**用于复杂结构化数据
- **枚举：**用于有限取值集合
- **联合类型：**当允许多种输入类型时使用

#### 校验规则
- **字符串校验：**
  - 长度约束（`minLength`、`maxLength`）
  - 格式模式匹配（邮箱、URL 等）
  - 字符集限制
  - 面向安全的内容过滤

- **数值校验：**
  - 范围约束（`minimum`、`maximum`）
  - 倍数约束（`multipleOf`）
  - 精度要求
  - 特殊值处理（NaN、Infinity）

- **数组校验：**
  - 大小约束（`minItems`、`maxItems`）
  - 元素类型校验
  - 唯一性要求
  - 顺序要求

- **对象校验：**
  - 必填属性约束
  - 额外属性策略
  - 嵌套校验规则
  - 依赖关系校验

### 输入示例

#### 好的示例：
```json
{
  "name": "search_web",
  "description": "Search the web for information",
  "parameters": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Search query string",
        "minLength": 1,
        "maxLength": 500,
        "examples": ["latest AI developments", "weather forecast"]
      },
      "limit": {
        "type": "integer",
        "description": "Maximum number of results to return",
        "minimum": 1,
        "maximum": 100,
        "default": 10
      },
      "language": {
        "type": "string",
        "description": "Language code for search results",
        "enum": ["en", "es", "fr", "de"],
        "default": "en"
      }
    },
    "required": ["query"],
    "additionalProperties": false
  }
}
```

#### 差的示例：
```json
{
  "name": "do_stuff",
  "description": "Does various operations",
  "parameters": {
    "type": "object",
    "properties": {
      "data": {
        "type": "string",
        "description": "Some data"
      }
    },
    "additionalProperties": true
  }
}
```

## 输出 Schema 设计

### 响应结构
```json
{
  "success": true,
  "data": {
    // Actual response data
  },
  "metadata": {
    "timestamp": "2024-01-15T10:30:00Z",
    "execution_time_ms": 234,
    "version": "1.0"
  },
  "warnings": [],
  "pagination": {
    "total": 100,
    "page": 1,
    "per_page": 10,
    "has_next": true
  }
}
```

### 数据一致性
- **结构可预测：**无论成功还是失败，返回结构都应尽量稳定
- **类型一致：**不同调用之间保持相同字段类型
- **Null 处理明确：**缺失值和空值要有清晰语义
- **空结果集一致化：**对空响应采用统一约定

### 元数据建议
- **执行时长：**便于做性能监控
- **时间戳：**便于审计与排查
- **版本信息：**便于兼容性管理
- **请求标识：**便于链路关联与调试

## 错误处理

### 错误响应结构
```json
{
  "success": false,
  "error": {
    "code": "INVALID_INPUT",
    "message": "The provided query is too short",
    "details": {
      "field": "query",
      "provided_length": 0,
      "minimum_length": 1
    },
    "retry_after": null,
    "documentation_url": "https://docs.example.com/errors#INVALID_INPUT"
  },
  "request_id": "req_12345"
}
```

### 错误类别

#### 客户端错误（相当于 4xx）
- **INVALID_INPUT：**参数格式错误或值非法
- **MISSING_PARAMETER：**缺少必填参数
- **VALIDATION_ERROR：**参数未通过校验规则
- **AUTHENTICATION_ERROR：**凭证无效或缺失
- **PERMISSION_ERROR：**权限不足
- **RATE_LIMIT_ERROR：**请求过多

#### 服务端错误（相当于 5xx）
- **INTERNAL_ERROR：**服务内部异常
- **SERVICE_UNAVAILABLE：**下游服务不可用
- **TIMEOUT_ERROR：**操作超时
- **RESOURCE_EXHAUSTED：**资源耗尽（内存、磁盘等）
- **DEPENDENCY_ERROR：**外部依赖失败

#### 工具特定错误
- **DATA_NOT_FOUND：**请求的数据不存在
- **FORMAT_ERROR：**数据格式不符合预期
- **PROCESSING_ERROR：**处理过程中出错
- **CONFIGURATION_ERROR：**工具配置错误

### 错误恢复策略

#### 重试逻辑
```json
{
  "retry_policy": {
    "max_attempts": 3,
    "backoff_strategy": "exponential",
    "base_delay_ms": 1000,
    "max_delay_ms": 30000,
    "retryable_errors": [
      "TIMEOUT_ERROR",
      "SERVICE_UNAVAILABLE",
      "RATE_LIMIT_ERROR"
    ]
  }
}
```

#### 回退行为
- **优雅降级：**尽可能返回部分结果
- **替代路径：**提供不同实现方式达到同一目标
- **缓存响应：**新鲜数据不可用时使用缓存
- **默认响应：**在无法返回具体结果时提供安全默认值

## 安全考虑

### 输入清洗
- **防 SQL 注入：**使用参数化查询
- **防 XSS：**对输出做 HTML 编码
- **防命令注入：**严格校验输入并配合沙箱
- **防路径穿越：**校验路径并加上访问限制

### 认证与授权
- **API key 管理：**安全存储并定期轮换
- **Token 校验：**验证 JWT 合法性和过期时间
- **权限检查：**采用基于角色的访问控制
- **审计日志：**记录安全相关事件

### 数据保护
- **PII 处理：**识别并保护个人信息
- **加密：**传输中与静态数据均应加密
- **数据保留：**遵循保留策略与合规要求
- **访问日志：**记录谁在何时访问了哪些数据

## 性能优化

### 响应时间
- **缓存策略：**对重复请求缓存结果
- **连接池：**复用外部服务连接
- **异步处理：**尽量采用非阻塞操作
- **资源优化：**提升整体资源利用率

### 吞吐量
- **批处理：**支持批量操作
- **并行执行：**在安全前提下并发处理
- **负载均衡：**在实例间分配请求
- **资源扩缩容：**根据负载自动扩展

### 资源管理
- **内存使用：**优化分配与清理
- **CPU 优化：**避免无意义计算
- **网络效率：**减少往返次数
- **存储优化：**使用高效数据结构与存储方式

## 测试策略

### 单元测试
```python
def test_search_web_valid_input():
    result = search_web("test query", limit=5)
    assert result["success"] is True
    assert len(result["data"]["results"]) <= 5

def test_search_web_invalid_input():
    result = search_web("", limit=5)
    assert result["success"] is False
    assert result["error"]["code"] == "INVALID_INPUT"
```

### 集成测试
- **端到端工作流：**覆盖完整用户场景
- **外部服务 Mock：**模拟外部依赖
- **错误注入：**模拟不同错误条件
- **性能测试：**进行负载和压力测试

### 合约测试
- **Schema 校验：**验证实现是否符合定义 Schema
- **向后兼容：**确保改动不破坏调用方
- **API 版本测试：**覆盖多个版本
- **消费者驱动合约：**从调用方视角验证接口

## 文档

### 工具文档模板
```markdown
# Tool Name

## Description
Brief description of what the tool does.

## Parameters
### Required Parameters
- `parameter_name` (type): Description

### Optional Parameters
- `optional_param` (type, default: value): Description

## Response
Description of response format and data.

## Examples
### Basic Usage
Input:
```json
{
  "parameter_name": "value"
}
```

Output:
```json
{
  "success": true,
  "data": {...}
}
```

## Error Codes
- `ERROR_CODE`: Description of when this error occurs
```

### API 文档
- **OpenAPI/Swagger 规范：**机器可读的 API 文档
- **交互式示例：**可直接运行的文档示例
- **代码示例：**多语言接入样例
- **变更日志：**版本历史与 breaking changes

## 版本策略

### 语义化版本
- **主版本：**破坏性变更
- **次版本：**新增功能，保持向后兼容
- **补丁版本：**修复 Bug，不引入新特性

### API 演进
- **废弃策略：**如何弃用旧功能
- **迁移指南：**帮助用户升级到新版本
- **向后兼容：**为旧版本保留支持
- **Feature flags：**逐步灰度发布新功能

## 监控与可观测性

### 指标采集
- **使用指标：**调用频次、成功率
- **性能指标：**响应时间、吞吐量
- **错误指标：**按类型划分的错误率
- **资源指标：**CPU、内存、网络使用量

### 日志
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "tool_name": "search_web",
  "request_id": "req_12345",
  "agent_id": "agent_001",
  "input_hash": "abc123",
  "execution_time_ms": 234,
  "success": true,
  "error_code": null
}
```

### 告警
- **错误率阈值：**错误率过高时告警
- **性能退化：**响应变慢时告警
- **资源耗尽：**接近资源上限时告警
- **服务可用性：**服务不可用时告警

## 常见反模式

### 工具设计反模式
- **God tools：**试图包办一切的工具
- **Chatty tools：**简单任务却需要频繁多次调用
- **Stateful tools：**跨调用维护内部状态
- **接口不一致：**每个工具各用一套约定

### 错误处理反模式
- **静默失败：**失败时没有充分错误信息
- **泛化错误：**错误信息缺乏可操作性
- **错误格式不一致：**不同工具返回不同错误结构
- **没有重试指引：**不告诉调用方是否可重试

### 性能反模式
- **全部同步：**该异步的地方也全部同步执行
- **无缓存：**重复请求重复计算
- **资源泄漏：**资源未被正确清理
- **无边界操作：**任务可能无限期运行

## 最佳实践清单

### 设计阶段
- [ ] 目标单一且清晰
- [ ] 输入输出契约明确
- [ ] 输入校验完整
- [ ] 在可能情况下保持幂等
- [ ] 已定义错误处理策略

### 实现阶段
- [ ] 错误处理健壮
- [ ] 输入已清洗
- [ ] 资源管理完善
- [ ] 已设置超时
- [ ] 已接入日志

### 测试阶段
- [ ] 所有功能具备单元测试
- [ ] 与依赖关系有集成测试
- [ ] 覆盖错误场景测试
- [ ] 覆盖性能测试
- [ ] 覆盖安全测试

### 文档阶段
- [ ] API 文档完整
- [ ] 具备用法示例
- [ ] 有错误码说明
- [ ] 有性能特征说明
- [ ] 有安全注意事项

### 部署阶段
- [ ] 已配置监控
- [ ] 已配置告警
- [ ] 建立性能基线
- [ ] 完成安全评审
- [ ] 编写运行手册

## 结论

高质量工具是多智能体系统有效运行的基础。它们应当可靠、安全、具备良好性能并易于使用。遵循这些最佳实践，可以让智能体更稳定地组合工具来解决复杂问题，同时维持系统的可靠性与安全性。

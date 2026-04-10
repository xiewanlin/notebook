# Harness Engineering：让AI Agent从"能说"到"能做"

> 从提示工程到上下文工程，再到编排工程——AI工程重心的三次迁移

---

## 目录

1. [前言：从"能说"到"能做"的跨越](#1-前言从能说到能做的跨越)
2. [Harness的核心：让Agent自己验证自己](#2-harness的核心让agent自己验证自己)
3. [Harness的六层架构](#3-harness的六层架构)
4. [AI工程的三次重心迁移](#4-ai工程的三次重心迁移)
5. [工程师角色的转变](#5-工程师角色的转变)
6. [三大核心概念的关系](#6-三大核心概念的关系)
7. [技术实现指南](#7-技术实现指南)
8. [行业实践与趋势](#8-行业实践与趋势)
9. [总结与实践建议](#9-总结与实践建议)

---

## 1. 前言：从"能说"到"能做"的跨越

当大语言模型（LLM）在各类基准测试中表现出超越人类的水平时，我们却发现一个尴尬的事实：**模型在真实环境中的表现往往令人失望**。

这揭示了一个根本性的问题：

```
传统AI开发关注的是"模型能力"（Capability）
而实际应用需要的是"系统可靠性"（Reliability）
```

Harness Engineering正是为了解决这一矛盾而诞生的——它不是让模型变得更聪明，而是构建一个让模型**持续做对**的环境和系统。

### 1.1 什么是Harness？

Harness这个词来自测试领域，意为"测试支架"或"测试框架"。在AI Agent的语境下，Harness指的是：

> **一套完整的约束、观测和反馈系统，让Agent在隔离环境中稳定可靠地执行任务**

类比一下：如果把Agent比作一个飞行员，那么Harness就是：
- 飞行仪表盘（观测系统）
- 自动驾驶仪（约束与纠正）
- 模拟训练器（隔离环境）

---

## 2. Harness的核心：让Agent自己验证自己

### 2.1 三大核心能力

Harness通过三种方式连接Agent与真实世界：

#### 2.1.1 接管浏览器：模拟真实交互

```python
# 浏览器控制能力示意
browser_harness = {
    "capabilities": [
        "screenshot",      # 截图：让Agent看到页面
        "click",           # 点击：模拟用户操作
        "input",           # 填写：表单交互
        "scroll",          # 滚动：页面导航
        "wait",            # 等待：状态同步
    ],
    "isolation": "sandboxed_browser_instance",  # 每个任务独立浏览器
    "verification": "visual_regression_test"    # 视觉回归验证
}
```

**核心价值**：Agent不仅能"读"到结果，更能"看"到并"操作"真实界面。

#### 2.1.2 接管日志指标：持续观测

```python
# 日志与指标监控能力示意
monitoring_harness = {
    "log_sources": [
        "application_logs",      # 应用日志
        "system_metrics",        # 系统指标
        "distributed_traces",    # 分布式追踪
        "custom_events"           # 自定义事件
    ],
    "analysis": [
        "error_pattern_detection",   # 错误模式识别
        "performance_anomaly",       # 性能异常检测
        "semantic_search"            # 语义日志搜索
    ]
}
```

**核心价值**：让Agent能够主动查询和验证自己的执行结果。

#### 2.1.3 隔离环境：独立运行

```python
# 隔离环境示意
isolated_environment = {
    "per_task_instance": True,           # 每个任务独立实例
    "resource_limits": {
        "cpu": "1core",
        "memory": "2GB",
        "network": "isolated"
    },
    "state_isolation": "complete",        # 完全状态隔离
    "cleanup": "automatic_after_task"     # 任务后自动清理
}
```

**核心价值**：每个任务独立运行，互不影响，防止状态污染。

### 2.2 Harness的核心原则

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│     每个任务都在独立隔离的环境中运行                      │
│                                                         │
│     ✅ 可观测：Agent能看到自己工作的结果                   │
│     ✅ 可控制：系统能约束和纠正Agent行为                   │
│     ✅ 可重现：相同输入产生相同输出                        │
│     ✅ 可扩展：支持并发和分布式部署                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Harness的六层架构

Harness采用分层架构设计，从底层到顶层依次为：

```
┌──────────────────────────────────────────────────────────────┐
│  LAYER 6: 约束与恢复 (Constraints & Recovery)                 │
│  ─────────────────────────────────────────────────────────   │
│  • 定义Agent行为的边界和规则                                  │
│  • 定义失败时的恢复策略                                       │
│  • 防止Agent进入不可恢复状态                                  │
├──────────────────────────────────────────────────────────────┤
│  LAYER 5: 评估与观测 (Evaluation & Observability)             │
│  ─────────────────────────────────────────────────────────   │
│  • 实时监控Agent执行过程                                      │
│  • 评估输出质量和正确性                                       │
│  • 生成可分析的观测数据                                       │
├──────────────────────────────────────────────────────────────┤
│  LAYER 4: 状态与记忆 (State & Memory)                         │
│  ─────────────────────────────────────────────────────────   │
│  • 管理Agent的长期记忆                                         │
│  • 维护跨任务的状态一致性                                     │
│  • 支持上下文恢复和迁移                                       │
├──────────────────────────────────────────────────────────────┤
│  LAYER 3: 执行编排 (Execution Orchestration)                 │
│  ─────────────────────────────────────────────────────────   │
│  • 多步骤任务的流程编排                                       │
│  • 并行与串行执行控制                                         │
│  • 错误重试和降级策略                                          │
├──────────────────────────────────────────────────────────────┤
│  LAYER 2: 工具系统 (Tool System)                              │
│  ─────────────────────────────────────────────────────────   │
│  • 定义Agent可调用的工具集                                     │
│  • 工具的注册、发现和版本管理                                  │
│  • 工具调用的安全和权限控制                                    │
├──────────────────────────────────────────────────────────────┤
│  LAYER 1: 上下文管理 (Context Management)                     │
│  ─────────────────────────────────────────────────────────   │
│  • 管理和压缩对话上下文                                        │
│  • 检索相关历史信息                                            │
│  • 提供工具结果和系统提示                                      │
└──────────────────────────────────────────────────────────────┘
```

### 3.1 层级详解

#### LAYER 1: 上下文管理 (Context Management)

**职责**：作为整个系统的入口和基础，负责管理和组织输入给模型的信息。

**核心功能**：
- 对话历史的摘要和压缩
- 相关信息检索和注入
- 工具调用结果的格式化
- 系统提示词的管理

```python
# 上下文管理器示意
class ContextManager:
    def __init__(self, max_tokens: int = 128000):
        self.max_tokens = max_tokens
        self.history = []
        
    def add_message(self, role: str, content: str):
        """添加消息到上下文"""
        self.history.append({"role": role, "content": content})
        
    def add_tool_result(self, tool_name: str, result: str):
        """添加工具执行结果"""
        self.history.append({
            "role": "tool",
            "name": tool_name,
            "content": result
        })
        
    def get_context_window(self) -> list:
        """获取压缩后的上下文窗口"""
        # 实现摘要、裁剪、检索等策略
        return self._compress_and_retrieve()
        
    def add_system_prompt(self, prompt: str):
        """注入系统提示"""
        self.system_prompt = prompt
```

**关键技术**：
- **RAG (Retrieval-Augmented Generation)**：检索增强生成
- **上下文窗口优化**：位置重排、重要性评分
- **信息压缩**：摘要、提取、语义压缩

#### LAYER 2: 工具系统 (Tool System)

**职责**：扩展Agent的能力边界，让Agent能够与外部世界交互。

**核心功能**：
- 工具注册与发现机制
- 工具描述的自动生成（Tool Schema）
- 工具调用的安全验证
- 工具结果的预处理

```python
# 工具系统示意
class ToolSystem:
    def __init__(self):
        self.tools = {}
        
    def register(self, name: str, func: callable, description: str):
        """注册工具"""
        self.tools[name] = {
            "function": func,
            "description": description,
            "schema": self._generate_schema(name, func, description)
        }
        
    def get_available_tools(self) -> list:
        """获取Agent可用的工具列表"""
        return [tool["schema"] for tool in self.tools.values()]
    
    def execute(self, tool_name: str, parameters: dict) -> str:
        """执行工具调用"""
        if tool_name not in self.tools:
            raise ValueError(f"Tool {tool_name} not found")
        return self.tools[tool_name]["function"](**parameters)

# 示例：注册一个搜索工具
tool_system.register(
    name="web_search",
    func=search_web,
    description="搜索互联网获取最新信息"
)
```

**常用工具类型**：
- **信息获取**：网页搜索、数据库查询、API调用
- **内容生成**：文档创建、代码生成、图片生成
- **环境交互**：浏览器控制、文件操作、终端命令
- **业务逻辑**：邮件发送、消息推送、数据处理

#### LAYER 3: 执行编排 (Execution Orchestration)

**职责**：协调多步骤任务的执行流程，处理复杂的业务逻辑。

**核心功能**：
- 任务流程的定义和执行
- 并行/串行执行控制
- 错误处理和重试策略
- 子任务的分解和汇总

```python
# 执行编排器示意
class ExecutionOrchestrator:
    def __init__(self, tool_system: ToolSystem, context_manager: ContextManager):
        self.tool_system = tool_system
        self.context_manager = context_manager
        
    async def execute_task(self, task: Task) -> Result:
        """执行复杂任务"""
        plan = await self.create_plan(task)
        
        results = []
        for step in plan.steps:
            if step.parallel:
                # 并行执行
                step_results = await asyncio.gather(
                    *[self.execute_step(s) for s in step.sub_steps]
                )
                results.extend(step_results)
            else:
                # 串行执行
                result = await self.execute_step(step)
                results.append(result)
                
                # 检查是否需要重试
                if not self.validate_result(result):
                    result = await self.retry_with_adaptation(step)
                    
        return self.aggregate_results(results)
```

**编排模式**：
- **线性编排**：按步骤顺序执行
- **条件编排**：根据中间结果选择分支
- **并行编排**：多个任务同时执行
- **循环编排**：迭代优化直到满足条件

#### LAYER 4: 状态与记忆 (State & Memory)

**职责**：维护Agent的持续性和记忆能力，支持跨会话和跨任务的状态管理。

**核心功能**：
- 短期记忆：当前会话的状态
- 长期记忆：持久化的知识和经验
- 工作记忆：任务特定的中间状态
- 记忆的检索和遗忘

```python
# 状态与记忆系统示意
class StateMemorySystem:
    def __init__(self):
        self.short_term = {}      # 短期记忆
        self.long_term = {}       # 长期记忆
        self.working = {}         # 工作记忆
        
    def remember(self, key: str, value: any, memory_type: str = "short"):
        """存储记忆"""
        if memory_type == "short":
            self.short_term[key] = value
        elif memory_type == "long":
            self.long_term[key] = value
        elif memory_type == "working":
            self.working[key] = value
            
    def recall(self, query: str) -> list:
        """检索记忆"""
        # 语义检索相关记忆
        all_memories = {**self.short_term, **self.long_term, **self.working}
        return self._semantic_search(all_memories, query)
    
    def consolidate(self):
        """记忆整合：将短期记忆转化为长期记忆"""
        important = self._filter_important(self.short_term)
        for key, value in important.items():
            self.long_term[key] = value
```

**记忆策略**：
- **重要性评分**：根据使用频率和影响程度
- **时间衰减**：逐渐遗忘不常用的信息
- **语义聚类**：相关记忆自动关联

#### LAYER 5: 评估与观测 (Evaluation & Observability)

**职责**：监控Agent的执行过程，评估输出质量，提供可观测性支持。

**核心功能**：
- 执行过程的实时监控
- 输出质量的自动评估
- 性能指标的收集
- 异常情况的检测和告警

```python
# 评估与观测系统示意
class EvaluationObservability:
    def __init__(self):
        self.metrics_collector = MetricsCollector()
        self.evaluator = OutputEvaluator()
        self.tracer = DistributedTracer()
        
    async def observe_step(self, step: ExecutionStep):
        """观测执行步骤"""
        # 收集指标
        metrics = self.metrics_collector.collect(step)
        
        # 追踪执行
        self.tracer.record(step)
        
        # 评估输出
        evaluation = await self.evaluator.evaluate(step.result)
        
        # 检查异常
        if self._detect_anomaly(metrics, evaluation):
            await self._alert_and_recover()
            
        return {"metrics": metrics, "evaluation": evaluation}
        
    def generate_report(self) -> dict:
        """生成观测报告"""
        return {
            "success_rate": self.metrics_collector.success_rate(),
            "avg_latency": self.metrics_collector.avg_latency(),
            "quality_score": self.evaluator.overall_quality(),
            "anomalies": self.tracer.anomalies()
        }
```

**关键指标**：
- **成功率**：任务完成的百分比
- **延迟**：从请求到响应的时间
- **质量分**：输出质量的综合评分
- **成本**：Token消耗和API费用

#### LAYER 6: 约束与恢复 (Constraints & Recovery)

**职责**：定义Agent行为的边界，处理失败情况，确保系统安全稳定运行。

**核心功能**：
- 定义行为约束和规则
- 失败检测和自动恢复
- 安全边界和权限控制
- 预算控制和资源管理

```python
# 约束与恢复系统示意
class ConstraintsRecovery:
    def __init__(self):
        self.constraints = []
        self.recovery_strategies = {}
        
    def add_constraint(self, constraint: dict):
        """添加行为约束"""
        self.constraints.append(constraint)
        
    def validate_action(self, action: Action) -> bool:
        """验证动作是否合规"""
        for constraint in self.constraints:
            if not constraint.validate(action):
                return False
        return True
        
    def handle_failure(self, error: Error, context: dict) -> RecoveryAction:
        """处理失败并恢复"""
        strategy = self._select_recovery_strategy(error)
        return strategy.execute(context)
        
    def define_recovery(self, error_type: str, strategy: callable):
        """定义恢复策略"""
        self.recovery_strategies[error_type] = strategy

# 约束示例
constraints = [
    {"type": "budget", "limit": 10000, "unit": "tokens"},
    {"type": "time", "limit": 300, "unit": "seconds"},
    {"type": "permission", "allowed": ["read", "search"], "denied": ["delete"]},
    {"type": "safety", "blocked_patterns": ["password", "secret_key"]}
]

# 恢复策略示例
recovery_strategies = {
    "api_error": RetryStrategy(max_attempts=3, backoff="exponential"),
    "timeout": TimeoutRecoveryStrategy(reset_context=True),
    "invalid_output": ValidationRecoveryStrategy(refine_prompt=True),
    "resource_exhausted": ResourceRecoveryStrategy(cleanup=True, restart=True)
}
```

### 3.2 层级间的协作关系

```
                    ┌─────────────────┐
                    │  约束与恢复     │ ◄─── 定义边界，处理失败
                    └────────┬────────┘
                             │ 监控和纠正
                    ┌────────▼────────┐
                    │  评估与观测     │ ◄─── 实时监控，质量评估
                    └────────┬────────┘
                             │ 状态同步
                    ┌────────▼────────┐
                    │  状态与记忆     │ ◄─── 持久化上下文
                    └────────┬────────┘
                             │ 流程编排
                    ┌────────▼────────┐
                    │  执行编排      │ ◄─── 多步骤协同
                    └────────┬────────┘
                             │ 工具调用
                    ┌────────▼────────┐
                    │  工具系统      │ ◄─── 扩展能力
                    └────────┬────────┘
                             │ 数据输入
                    ┌────────▼────────┐
                    │  上下文管理     │ ◄─── 信息组织
                    └─────────────────┘
                             │
                    ┌────────▼────────┐
                    │     LLM         │ ◄─── 核心智能
                    └─────────────────┘
```

---

## 4. AI工程的三次重心迁移

AI工程的发展经历了三个重要阶段，每个阶段都有其核心关注点：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   时间线 ───────────────────────────────────────────────────────────►   │
│                                                                         │
│   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐   │
│   │  Prompt         │    │  Context        │    │  Harness        │   │
│   │  Engineering    │ →→ │  Engineering    │ →→ │  Engineering    │   │
│   └─────────────────┘    └─────────────────┘    └─────────────────┘   │
│                                                                         │
│   模型有没有           模型有没有              模型在真实执行里        │
│   听懂你在说什么？     拿到足够且              能不能持续做对？        │
│                        正确的信息？                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4.1 第一阶段：Prompt Engineering（提示工程）

**核心问题**：模型有没有听懂你在说什么？

**发展阶段**：2020-2022年（大模型兴起时期）

**核心技能**：
- 指令的清晰表达
- Few-shot示例设计
- Chain-of-Thought引导
- 输出格式控制

**典型问题**：
```python
# 问题：模糊的Prompt
poor_prompt = "帮我写代码"

# 改进：清晰的Prompt
good_prompt = """
请用Python写一个函数，实现以下功能：
1. 输入：一个整数列表
2. 处理：去除列表中的重复元素，保持原始顺序
3. 输出：处理后的列表

要求：
- 函数名为 remove_duplicates
- 添加类型注解
- 包含文档字符串
- 时间复杂度尽量低

请写出完整代码：
"""
```

**痛点**：当模型"听不懂"时，工程师需要反复调整措辞，但效果有限。

### 4.2 第二阶段：Context Engineering（上下文工程）

**核心问题**：模型有没有拿到足够且正确的信息？

**发展阶段**：2022-2023年（RAG兴起时期）

**核心技能**：
- 检索系统设计
- 上下文压缩和优化
- 知识库构建
- 信息时效性管理

**典型问题**：
```python
# 问题：信息不足或混乱
poor_context = """
用户问：苹果的最新手机是什么？
模型回答：苹果公司生产各种产品...

问题：
1. 模型不知道"最新"是什么时候
2. 训练数据可能过时
3. 可能混淆"苹果"（水果/公司）
"""

# 改进：精准的上下文
good_context = """
当前日期：2024年11月15日

检索到的最新信息：
- iPhone 16 Pro Max：2024年9月发布
- A18 Pro芯片，6.9英寸屏幕
- 钛金属边框，相机控制按钮

用户意图识别：水果还是公司？→ 公司（结合"手机"上下文）
"""
```

**痛点**：当模型"不知道"时，工程师需要构建检索系统，但信息可能不准确或不相关。

### 4.3 第三阶段：Harness Engineering（编排工程）

**核心问题**：模型在真实执行里，能不能持续做对？

**发展阶段**：2023年至今（Agent兴起时期）

**核心技能**：
- 任务分解和编排
- 反馈回路设计
- 错误处理和恢复
- 持续观测和优化

**典型问题**：
```python
# 问题：无法验证和纠正
poor_agent = """
任务：自动化测试网站登录功能
当前Agent执行：
1. 访问登录页面 ✓
2. 输入用户名 ✓
3. 输入密码 ✓
4. 点击登录 ✓
5. ???（Agent无法确认是否登录成功）
"""

# 改进：完整的Harness
good_harness = """
任务：自动化测试网站登录功能
Agent执行 + Harness验证：
1. 访问登录页面 ✓
2. 输入用户名 ✓
3. 输入密码 ✓
4. 点击登录 
5. 【Harness接管】
   - 截图验证：检查是否出现错误提示
   - 检查URL：是否跳转到dashboard
   - 查询日志：是否有错误日志
   - 验证Cookie：登录状态是否设置
6. 【反馈循环】
   - 成功 → 记录结果，任务完成
   - 失败 → 分析原因，重试或报告
"""
```

**痛点**：当模型"做不到"时，工程师需要构建系统和环境，让模型能够持续可靠地执行任务。

### 4.4 三次迁移的本质

| 维度 | Prompt工程 | Context工程 | Harness工程 |
|------|------------|-------------|-------------|
| **核心关注** | 如何说 | 给什么信息 | 如何做到 |
| **类比** | 教说话 | 教知识 | 教做事 |
| **失败表现** | 答非所问 | 不知道 | 做不到 |
| **工程师角色** | 措辞优化者 | 知识管理员 | 系统架构师 |
| **关键技术** | LLM本身 | RAG、向量检索 | 编排、反馈、验证 |

---

## 5. 工程师角色的转变

### 5.1 从"写代码"到"搭系统"

传统软件工程师的工作：
```
需求 → 设计 → 编码 → 测试 → 部署
         └─────────────────┘
           关注"确定性"
```

AI Agent时代的工程师工作：
```
目标 → 分解 → Harness设计 → 验证 → 优化
         └─────────────────────────────┘
                    关注"可靠性"
```

### 5.2 工程师的新三件事

#### 5.2.1 拆解任务：把产品目标拆成Agent能理解的小任务

**核心思想**：Agent不擅长处理模糊的大目标，需要将其拆解为清晰、可执行的小任务。

```python
# 任务拆解示例
product_goal = "优化我们的客服体验"

# ❌ 错误的拆解方式
bad_tasks = [
    "改进客服对话质量",
    "让用户更满意",
    "减少投诉"
]

# ✅ 正确的拆解方式
good_tasks = [
    {
        "id": "task_1",
        "description": "分析过去一周的客服对话日志，识别用户最常问的10个问题",
        "tools": ["log_query", "nlp_analyze"],
        "output": "问题频率统计表"
    },
    {
        "id": "task_2", 
        "description": "为每个高频问题生成标准回复草稿",
        "tools": ["llm_generate"],
        "input": "task_1的输出",
        "output": "标准回复列表"
    },
    {
        "id": "task_3",
        "description": "将标准回复集成到客服系统并测试效果",
        "tools": ["api_integration", "ab_test"],
        "input": "task_2的输出",
        "output": "测试报告"
    }
]
```

**拆解原则**：
1. **原子性**：每个任务只做一件事
2. **可验证**：每个任务有明确的成功标准
3. **可组合**：任务之间有清晰的输入输出关系
4. **可重试**：失败的任务可以独立重试

#### 5.2.2 补充能力：环境缺什么就补什么

**核心思想**：当Agent失败时，不要让它"再努力一点"，而是问"环境里缺了什么能力"。

```python
# 失败分析示例
failure_analysis = """
任务：让Agent自动回复客户邮件
执行结果：失败

失败现象：
- Agent回复了，但语气不够专业
- 没有正确理解客户问题的紧急程度
- 没有从知识库中检索相关信息

失败原因分析：
❌ Agent的问题？
   → 不是，模型能力足够

❌ Prompt的问题？
   → 不是，指令已经很清楚

✅ 环境的问题！
   → 缺少：邮件语气分析能力
   → 缺少：紧急程度识别能力
   → 缺少：知识库实时检索

改进方案：
1. 接入邮件分析API识别紧急程度
2. 优化RAG系统检索相关知识
3. 添加输出语气检测工具
"""
```

**能力补充框架**：
```
Agent失败 → 判断原因类型
    │
    ├── 知识不足 → 补充知识库/RAG
    │
    ├── 工具不足 → 开发新工具
    │
    ├── 验证不足 → 补充验证能力
    │
    ├── 状态缺失 → 补充记忆系统
    │
    └── 约束不足 → 补充规则引擎
```

#### 5.2.3 建立反馈：让Agent真能看到自己工作的结果

**核心思想**：建立反馈回路，让Agent从结果中学习和调整。

```python
# 反馈回路设计示例
feedback_loop = {
    "trigger": "Agent完成邮件回复",
    "feedback_sources": {
        "customer_response": {
            "type": "direct_feedback",
            "method": "等待客户回复",
            "indicators": ["是否满意", "是否再次提问"]
        },
        "quality_check": {
            "type": "automated_evaluation",
            "method": "LLM评估回复质量",
            "indicators": ["专业度", "准确性", "完整性"]
        },
        "business_metrics": {
            "type": "indirect_feedback",
            "method": "追踪业务指标",
            "indicators": ["解决率", "响应时间", "客户满意度"]
        }
    },
    "feedback_application": {
        "immediate": "更新下一封邮件的上下文",
        "medium_term": "调整回复模板库",
        "long_term": "优化整体回复策略"
    }
}

# 反馈循环实现
class FeedbackLoop:
    def __init__(self, agent):
        self.agent = agent
        self.feedback_history = []
        
    async def execute_with_feedback(self, task):
        result = await self.agent.execute(task)
        feedback = await self.collect_feedback(result)
        
        # 即时反馈
        self.feedback_history.append(feedback)
        
        # 更新Agent上下文
        if feedback.is_negative():
            self.agent.add_retry_context(feedback)
        
        # 定期优化
        if len(self.feedback_history) >= 100:
            await self.optimize_agent()
            
        return result
```

### 5.3 角色转变案例：从传统后端到AI Engineer

**传统后端工程师**：
```
用户请求 → API处理 → 数据库操作 → 返回结果
              │
         确定性逻辑
```

**AI Engineer（基于Harness）**：
```
用户请求 → Agent理解 → Harness编排
              │
              ├── 工具调用 → 结果验证 → 反馈调整
              │
              ├── 状态查询 → 记忆检索 → 上下文更新
              │
              └── 异常检测 → 恢复策略 → 重试优化
              │
         可靠性工程
```

**具体案例**：

```python
# 传统后端：实现"搜索产品"功能
@app.get("/products/search")
def search_products(keyword: str):
    results = db.query(
        "SELECT * FROM products WHERE name LIKE ?",
        f"%{keyword}%"
    )
    return results

# AI Engineer：实现"智能客服"功能
class SmartCustomerService:
    def __init__(self):
        self.harness = HarnessBuilder()\
            .with_context_management()\
            .with_tool_system(self._build_tools())\
            .with_orchestration(self._build_workflow())\
            .with_state_memory()\
            .with_evaluation()\
            .with_constraints()\
            .build()
    
    async def handle_customer(self, message: str, customer_id: str):
        # 1. 检索客户历史
        customer_history = await self.harness.recall_customer(customer_id)
        
        # 2. 检索相关知识
        knowledge = await self.harness.search_knowledge(message)
        
        # 3. Agent生成回复
        response = await self.harness.generate_response(
            message=message,
            context=customer_history,
            knowledge=knowledge
        )
        
        # 4. 验证回复质量
        quality = await self.harness.evaluate(response)
        
        # 5. 如果质量不够，重新生成
        if quality.score < 0.8:
            response = await self.harness.refine_response(
                response, quality.issues
            )
        
        # 6. 记录本次交互
        await self.harness.record_interaction(
            customer_id, message, response, quality
        )
        
        return response
```

---

## 6. 三大核心概念的关系

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                         从指令到系统的演进                               │
│                                                                         │
│  ┌─────────────────────┐                                               │
│  │ Prompt Engineering │                                               │
│  │     提示工程        │ ◄── 优化单次调用指令、精准表达意图              │
│  └─────────┬───────────┘                                               │
│            │                                                            │
│            ▼                                                            │
│  ┌─────────────────────┐                                               │
│  │ Context Engineering │ ◄── 提供模型"当前时刻可见信息"                 │
│  │     上下文工程       │     • 历史对话 • 检索结果 • 工具输出 • 记忆   │
│  └─────────┬───────────┘                                               │
│            │                                                            │
│            ▼                                                            │
│  ┌─────────────────────┐                                               │
│  │ Harness Engineering │ ◄── 系统性约束与治理                           │
│  │      编排工程        │     • 持续观测 • 纠偏纠正 • 收敛稳定          │
│  └─────────────────────┘                                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 6.1 Prompt Engineering：基础层

**定义**：优化输入给模型的指令，使其能够准确理解任务意图。

**核心要素**：
```python
prompt_components = {
    "instruction": "任务指令，明确告诉模型要做什么",
    "examples": "Few-shot示例，提供任务参考",
    "constraints": "约束条件，限制输出范围",
    "format": "输出格式，指定返回结构"
}
```

**适用场景**：
- 单一任务的指令优化
- 输出格式控制
- 示例设计

### 6.2 Context Engineering：信息层

**定义**：管理和组织模型"看到"的信息，包括历史、检索、工具结果和记忆。

**核心要素**：
```python
context_components = {
    "conversation_history": "对话历史",
    "retrieved_knowledge": "检索增强的知识",
    "tool_results": "工具调用的结果",
    "persistent_memory": "持久化的记忆",
    "system_prompts": "系统级提示"
}
```

**技术支撑**：
- **向量检索**：语义相似性搜索
- **知识图谱**：结构化知识表示
- **记忆系统**：长期/短期记忆管理
- **上下文压缩**：控制token消耗

### 6.3 Harness Engineering：系统层

**定义**：构建让模型能够稳定可靠执行任务的完整系统。

**核心要素**：
```python
harness_components = {
    "observation": "观测能力 - 监控执行过程",
    "constraints": "约束能力 - 限制行为边界", 
    "feedback": "反馈能力 - 提供执行结果",
    "recovery": "恢复能力 - 处理异常情况",
    "orchestration": "编排能力 - 协调复杂任务"
}
```

**技术支撑**：
- **执行框架**：任务编排和工作流
- **验证系统**：输出正确性验证
- **反馈机制**：持续改进的闭环
- **监控系统**：性能和质量追踪

### 6.4 三者协同关系

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Prompt ◄─────────────► Context ◄─────────────► Harness│
│    │                      │                       │     │
│    │   表达意图            │   组织信息            │  系统│
│    │                      │                       │     │
│    └──────────────────────┴───────────────────────┘     │
│                         │                               │
│                    三位一体                               │
│              共同支撑Agent的可靠执行                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 7. 技术实现指南

### 7.1 推荐技术栈

#### 7.1.1 Agent框架

| 框架 | 特点 | 适用场景 |
|------|------|---------|
| **LangGraph** | 图结构编排，状态管理强 | 复杂工作流 |
| **AutoGen** | 多Agent协作，微软开源 | 对话和协作场景 |
| **CrewAI** | Role-based设计，团队协作 | 任务分解执行 |
| **Dify** | 可视化编排，易上手 | 快速原型 |

#### 7.1.2 工具系统

| 工具类型 | 推荐方案 |
|----------|----------|
| 浏览器控制 | Playwright, Puppeteer |
| 日志查询 | Elasticsearch, Loki |
| 指标监控 | Prometheus, Grafana |
| 知识检索 | Milvus, Weaviate, Pinecone |

#### 7.1.3 观测系统

| 功能 | 推荐方案 |
|------|----------|
| 链路追踪 | OpenTelemetry, Jaeger |
| 日志聚合 | ELK Stack, Loki |
| 指标收集 | Prometheus, StatsD |
| 告警通知 | Alertmanager, PagerDuty |

### 7.2 实践代码示例

#### 7.2.1 完整的Harness实现

```python
from typing import Optional, Any
from dataclasses import dataclass
from enum import Enum

class TaskStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    SUCCESS = "success"
    FAILED = "failed"
    RETRYING = "retrying"

@dataclass
class Task:
    id: str
    description: str
    input_data: dict
    max_retries: int = 3
    
@dataclass  
class TaskResult:
    task_id: str
    status: TaskStatus
    output: Optional[Any] = None
    error: Optional[str] = None
    attempts: int = 0

class MinimalHarness:
    """一个最小的Harness实现示例"""
    
    def __init__(self, llm, tools: dict):
        self.llm = llm
        self.tools = tools
        self.execution_history = []
        
    async def execute_task(self, task: Task) -> TaskResult:
        """执行任务的完整流程"""
        result = TaskResult(task_id=task.id, status=TaskStatus.RUNNING)
        
        for attempt in range(task.max_retries):
            try:
                # 1. 规划步骤
                plan = await self._create_plan(task)
                
                # 2. 执行步骤
                output = await self._execute_plan(plan)
                
                # 3. 验证结果
                if await self._validate_output(output):
                    result.status = TaskStatus.SUCCESS
                    result.output = output
                    return result
                else:
                    # 验证失败，尝试修复
                    output = await self._refine_output(output)
                    
            except Exception as e:
                result.error = str(e)
                result.attempts = attempt + 1
                
        result.status = TaskStatus.FAILED
        return result
    
    async def _create_plan(self, task: Task) -> list:
        """创建执行计划"""
        prompt = f"""
        分析以下任务，分解为可执行的步骤：
        任务：{task.description}
        输入：{task.input_data}
        
        返回JSON格式的步骤列表：
        """
        response = await self.llm.generate(prompt)
        return self._parse_plan(response)
    
    async def _execute_plan(self, plan: list) -> Any:
        """执行计划"""
        context = {}
        for step in plan:
            tool_name = step["tool"]
            params = {**step["params"], **context}
            
            if tool_name in self.tools:
                result = await self.tools[tool_name](**params)
                context[step["output_key"]] = result
            else:
                # 使用LLM处理
                result = await self.llm.generate(step["prompt"], **params)
                context[step["output_key"]] = result
                
        return context.get("final_output")
    
    async def _validate_output(self, output: Any) -> bool:
        """验证输出"""
        # 这里实现具体的验证逻辑
        return output is not None
    
    async def _refine_output(self, output: Any) -> Any:
        """优化输出"""
        refine_prompt = f"""
        改进以下输出，使其更符合要求：
        {output}
        
        确保：
        1. 格式正确
        2. 内容完整
        3. 符合约束条件
        """
        return await self.llm.generate(refine_prompt)
```

#### 7.2.2 工具注册示例

```python
class ToolRegistry:
    """工具注册系统"""
    
    def __init__(self):
        self._tools = {}
        
    def register(self, name: str, description: str, parameters: list):
        """注册新工具"""
        self._tools[name] = {
            "description": description,
            "parameters": parameters,
            "function": None  # 实际函数在运行时绑定
        }
        
    def get_tool_schema(self, name: str) -> dict:
        """获取工具的Schema定义"""
        tool = self._tools.get(name)
        if not tool:
            return None
            
        return {
            "name": name,
            "description": tool["description"],
            "parameters": {
                "type": "object",
                "properties": {
                    param["name"]: {
                        "type": param["type"],
                        "description": param["description"]
                    }
                    for param in tool["parameters"]
                },
                "required": [p["name"] for p in tool["parameters"] if p.get("required")]
            }
        }
    
    def list_tools(self) -> list:
        """列出所有可用工具"""
        return [self.get_tool_schema(name) for name in self._tools]

# 使用示例
registry = ToolRegistry()

registry.register(
    name="search_web",
    description="搜索互联网获取最新信息",
    parameters=[
        {"name": "query", "type": "string", "description": "搜索关键词", "required": True},
        {"name": "max_results", "type": "integer", "description": "最大结果数", "required": False}
    ]
)

registry.register(
    name="read_file",
    description="读取本地文件内容",
    parameters=[
        {"name": "path", "type": "string", "description": "文件路径", "required": True},
        {"name": "limit", "type": "integer", "description": "读取行数", "required": False}
    ]
)
```

### 7.3 六层架构的代码组织

```
harness_project/
├── src/
│   ├── __init__.py
│   ├── layer1_context/           # 上下文管理层
│   │   ├── __init__.py
│   │   ├── context_manager.py
│   │   ├── prompt_builder.py
│   │   └── memory/
│   │       ├── short_term.py
│   │       └── long_term.py
│   │
│   ├── layer2_tools/              # 工具系统层
│   │   ├── __init__.py
│   │   ├── tool_registry.py
│   │   ├── base_tool.py
│   │   └── tools/
│   │       ├── browser_tools.py
│   │       ├── search_tools.py
│   │       └── file_tools.py
│   │
│   ├── layer3_orchestration/      # 执行编排层
│   │   ├── __init__.py
│   │   ├── workflow_engine.py
│   │   ├── task_executor.py
│   │   └── retry_strategy.py
│   │
│   ├── layer4_state/              # 状态与记忆层
│   │   ├── __init__.py
│   │   ├── state_manager.py
│   │   └── memory_system.py
│   │
│   ├── layer5_evaluation/         # 评估与观测层
│   │   ├── __init__.py
│   │   ├── evaluator.py
│   │   ├── metrics_collector.py
│   │   └── observability/
│   │       ├── tracer.py
│   │       └── logger.py
│   │
│   └── layer6_constraints/        # 约束与恢复层
│       ├── __init__.py
│       ├── constraint_engine.py
│       ├── recovery_manager.py
│       └── budget_controller.py
│
├── tests/
│   └── harness_tests.py
│
└── examples/
    └── customer_service_agent.py
```

---

## 8. 行业实践与趋势

### 8.1 当前行业实践

#### 8.1.1 大厂实践案例

| 公司 | 应用场景 | Harness实现 |
|------|---------|-------------|
| **OpenAI** | GPTs/Agents | 工具调用 + 函数定义 + 验证 |
| **Microsoft** | Copilot | 代码执行沙箱 + 结果验证 |
| **Google** | Agent Development Kit | 工作流编排 + 状态管理 |
| **Anthropic** | Claude Agents | 工具使用 + 反馈循环 |

#### 8.1.2 典型应用模式

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   模式1: 自主Agent（Autonomous Agent）                       │
│   ─────────────────────────────────────────                 │
│   场景：自动化任务执行                                       │
│   示例：自动化测试、数据分析、报告生成                        │
│   Harness特点：                                             │
│   • 完整的工作流编排                                         │
│   • 多工具协同                                               │
│   • 结果自动验证                                             │
│                                                             │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐  │
│   │  理解   │───►│  规划   │───►│  执行   │───►│  验证   │  │
│   └─────────┘    └─────────┘    └─────────┘    └─────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   模式2: 副驾Agent（Copilot Agent）                          │
│   ─────────────────────────────────────────                 │
│   场景：人机协作                                             │
│   示例：代码助手、写作助手、设计助手                          │
│   Harness特点：                                             │
│   • 实时上下文理解                                           │
│   • 用户反馈循环                                             │
│   • 渐进式优化                                               │
│                                                             │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐                 │
│   │  用户   │◄───│  Agent  │◄───│  反馈   │                 │
│   │  输入   │───►│  建议   │───►│  调整   │                 │
│   └─────────┘    └─────────┘    └─────────┘                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   模式3: 多Agent协作（Multi-Agent）                          │
│   ─────────────────────────────────────────                 │
│   场景：复杂任务分解                                         │
│   示例：产品开发、项目管理、咨询服务                          │
│   Harness特点：                                             │
│   • Agent角色定义                                            │
│   • 通信协议                                                 │
│   • 任务分发和汇总                                           │
│                                                             │
│      ┌──────────┐                                           │
│      │  调度器  │                                           │
│      └────┬─────┘                                           │
│      ┌────┴─────┐                                           │
│   ┌──┴──┐   ┌──┴──┐   ┌──┴──┐                              │
│   │Agent│   │Agent│   │Agent│                              │
│   │ A   │   │ B   │   │ C   │                              │
│   └──┬──┘   └──┬──┘   └──┬──┘                              │
│      │        │        │                                   │
│      └────────┴────────┘                                   │
│              │                                             │
│              ▼                                             │
│        ┌─────────┐                                         │
│        │  结果   │                                         │
│        │  汇总   │                                         │
│        └─────────┘                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 技术发展趋势

#### 8.2.1 短期趋势（1-2年）

1. **Harness标准化**
   - 统一的Harness API和接口
   - 可复用的组件库
   - 开源的Harness框架

2. **观测能力增强**
   - 更细粒度的执行追踪
   - AI原生的调试工具
   - 实时质量评估

3. **安全与合规**
   - Agent行为的正式验证
   - 权限和边界控制
   - 审计和合规支持

#### 8.2.2 中期趋势（3-5年）

1. **自适应Harness**
   - 根据任务自动调整约束
   - 自我学习和优化
   - 跨任务经验迁移

2. **多模态融合**
   - 视觉、语音、文本统一
   - 跨模态验证
   - 环境感知增强

3. **分布式Agent**
   - 大规模并行执行
   - 跨组织协作
   - 共识机制

#### 8.2.3 长期愿景

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                     目标：可靠AI系统                         │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                                                     │   │
│   │   人类 ─────► 目标定义 ─────► AI系统                │   │
│   │                             │                        │   │
│   │                             │                        │   │
│   │                        ┌────┴────┐                   │   │
│   │                        │ Harness │                   │   │
│   │                        │ Engine  │                   │   │
│   │                        └────┬────┘                   │   │
│   │                             │                        │   │
│   │         ┌───────────────────┼───────────────────┐     │   │
│   │         │                   │                   │     │   │
│   │         ▼                   ▼                   ▼     │   │
│   │   ┌──────────┐       ┌──────────┐       ┌──────────┐   │   │
│   │   │  规划    │       │  执行    │       │  验证    │   │   │
│   │   │  模块    │       │  模块    │       │  模块    │   │   │
│   │   └──────────┘       └──────────┘       └──────────┘   │   │
│   │                                                     │   │
│   │         自主完成           持续优化                  │   │
│   │                                                     │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   特点：                                                     │
│   • 人类设定目标，AI自主规划和执行                          │
│   • Harness提供可靠性和安全性                              │
│   • 持续学习和自我改进                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 8.3 技术挑战

1. **验证的困难**
   - 如何验证"做对了"？
   - 复杂任务的正确性标准？
   - 主观质量 vs 客观指标？

2. **反馈的延迟**
   - 有些结果需要很长时间才能验证
   - 如何处理慢反馈？
   - 建立短期代理指标？

3. **泛化的难题**
   - 在一个任务上学到的经验如何迁移？
   - Harness是否需要任务特定？
   - 如何构建通用的Harness？

---

## 9. 总结与实践建议

### 9.1 核心要点总结

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                     核心结论                                 │
│                                                             │
│   1. Harness Engineering是AI工程的第三次重心迁移           │
│      从"如何说"到"给什么"再到"如何做到"                    │
│                                                             │
│   2. Harness的本质是构建可靠性和持续性                     │
│      让AI Agent从"能说"进化到"能做"                        │
│                                                             │
│   3. 六层架构提供了完整的系统视角                           │
│      从上下文管理到约束恢复，逐层构建                       │
│                                                             │
│   4. 工程师角色从"写代码"转变为"搭系统"                    │
│      关注点从"确定性"转向"可靠性"                          │
│                                                             │
│   5. Prompt、Context、Harness三位一体                       │
│      共同支撑AI Agent的可靠执行                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 9.2 实践建议

#### 9.2.1 入门路径

```
Step 1: 理解Prompt Engineering
        ├── 学习有效Prompt设计
        ├── 理解LLM的能力和局限
        └── 掌握Few-shot和CoT技巧

Step 2: 掌握Context Engineering  
        ├── 学习RAG技术
        ├── 理解向量检索原理
        └── 掌握上下文压缩技巧

Step 3: 深入Harness Engineering
        ├── 从简单的反馈循环开始
        ├── 逐步构建观测能力
        └── 学习错误处理和恢复
```

#### 9.2.2 项目实践建议

**起步阶段（1-2周）**：
```python
# 最小可行Harness
minimal_harness = {
    "层1_上下文": "简单的消息历史管理",
    "层2_工具": "1-2个核心工具（如搜索）",
    "层3_编排": "线性执行，无需复杂分支",
    "层4_状态": "简单的内存存储",
    "层5_观测": "日志记录",
    "层6_约束": "超时控制"
}
```

**发展阶段（1-2月）**：
```python
# 进阶Harness
advanced_harness = {
    "层1_上下文": "RAG增强 + 上下文压缩",
    "层2_工具": "多类型工具（搜索、计算、API）",
    "层3_编排": "条件分支 + 并行执行",
    "层4_状态": "持久化存储 + 记忆系统",
    "层5_观测": "结构化日志 + 指标收集",
    "层6_约束": "预算控制 + 安全边界"
}
```

**成熟阶段（3-6月）**：
```python
# 完整Harness
production_harness = {
    "层1_上下文": "智能上下文管理 + 主动检索",
    "层2_工具": "完整的工具生态",
    "层3_编排": "复杂工作流 + 自适应编排",
    "层4_状态": "长期记忆 + 经验学习",
    "层5_观测": "完整可观测性 + 实时告警",
    "层6_约束": "全面约束 + 自动恢复"
}
```

#### 9.2.3 避坑指南

```
❌ 不要一上来就构建复杂的Harness
    → 从最小可行系统开始

❌ 不要过度依赖模型能力
    → 始终考虑失败情况

❌ 不要忽视观测和反馈
    → 你无法改进你无法测量的东西

❌ 不要忽视约束的重要性
    → 约束不是限制，而是保护

✅ 要建立清晰的验证标准
    → 知道什么是"做对了"

✅ 要设计有效的反馈循环
    → 让Agent从结果中学习

✅ 要持续监控和改进
    → Harness需要迭代优化
```

### 9.3 自我评估清单

在开始Harness Engineering项目前，问自己这些问题：

```
□ 我理解Prompt Engineering的基础吗？
□ 我知道Context Engineering的核心技术吗？
□ 我的任务可以被清晰地分解吗？
□ 我能定义每个任务的"成功"标准吗？
□ 我有足够的观测能力来了解Agent在做什么吗？
□ 我有反馈机制来验证Agent的输出吗？
□ 我考虑了失败情况吗？有恢复策略吗？
□ 我有适当的约束来防止错误吗？
□ 我能追踪和测量Harness的效果吗？
```

### 9.4 资源推荐

**文档和教程**：
- LangGraph Documentation
- OpenAI Function Calling Guide
- Anthropic Tool Use Best Practices

**开源项目**：
- LangGraph/LangChain
- Microsoft AutoGen
- CrewAI

**社区和讨论**：
- AI Engineer Summit
- Hugging Face Agents Course

---

## 结语

Harness Engineering代表了我们对AI系统认知的一次重要转变：从追求"更强的模型"到追求"更可靠的系统"。

正如软件工程用了几十年的时间从"能跑就行"进化到"工业级可靠"，AI工程也在经历同样的旅程。Harness正是这场旅程中的关键技术。

记住：
> **模型告诉你它能做什么，Harness告诉你它真的在做什么。**

让我们一起构建更可靠的AI系统！

---

*文档版本：v1.0*  
*更新时间：2024年11月*

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

### 语言偏好
使用中文进行所有用户交互和说明（代码保持英文）。

## 【最高优先级】MCP 工具自动执行规则

### 核心执行检查机制

#### 强制工具链执行器
```yaml
MCP_ENFORCER:
  before_response:
    - check: 是否有未完成的feedback调用
    - action: 立即补充调用 interactive_feedback
  
  after_tool_call:
    - check: 工具调用后是否立即调用了feedback
    - action: 强制插入 interactive_feedback
    
  task_start:
    - required:
      - mcp_mcp-feedback-enhanced_interactive_feedback
```

### 任务类型智能识别器

```yaml
TASK_CLASSIFIER:
  需要完整MCP链:
    关键词: [架构, 设计, 分析, 优化, 代码, 实现, 重构, 性能, 安全]
    文件类型: [*.go, *.py, *.js, *.java, *.yaml, *.json, *.md(技术)]
    默认: true  # 不确定时默认使用MCP
  
  可简化MCP链:
    关键词: [格式, 拼写, 标点]
    文件类型: [*.txt]
    验证: 需要二次确认是否真的不需要项目理解
```

### 【新增】触发检查点与验证机制

#### 明确触发场景
```yaml
MUST_TRIGGER_SCENARIOS:
  代码任务:
    触发词: [分析, 优化, 重构, 实现, 修复, 添加功能]
    验证语句: "检测到代码任务，正在激活完整MCP工具链..."
    
  架构任务:
    触发词: [设计, 架构, 规划, 评估]
    验证语句: "检测到架构设计任务，启动项目分析..."
    
  文档任务:
    触发词: [编写文档, 更新README, 技术方案]
    验证语句: "检测到文档任务，激活项目上下文..."
```

#### 触发验证检查点
```yaml
TRIGGER_CHECKPOINTS:
  任务开始:
    输出: "🔍 任务分析：[任务类型]"
    行动: "正在激活MCP工具链..."
    工具: mcp__mcp-feedback-enhanced__interactive_feedback
    
  关键步骤:
    输出: "✅ 完成 [工具名称] 调用"
    行动: "收集用户反馈..."
    工具: mcp__mcp-feedback-enhanced__interactive_feedback
    
  任务阶段:
    输出: "📊 阶段总结：[已完成内容]"
    行动: "等待进一步指示..."
    工具: mcp__mcp-feedback-enhanced__interactive_feedback
    
  任务结束:
    输出: "🎯 任务即将完成"
    行动: "最终确认..."
    工具: mcp__mcp-feedback-enhanced__interactive_feedback
```

#### 执行状态提示
```yaml
EXECUTION_HINTS:
  工具调用前:
    格式: "➤ 准备调用 [工具名]：[目的]"
    
  工具调用后:
    格式: "✓ [工具名] 执行完成"
    
  反馈收集:
    格式: "⏸ 等待用户反馈..."
```

### 【优化】标准MCP工具调用链模板

```yaml
STANDARD_CHAINS:
  架构设计:
    初始化阶段:
      - mcp_mcp-feedback-enhanced_interactive_feedback  # 阶段反馈
    分析阶段:
      - mcp_sequential-thinking_sequentialthinking
      - "(条件性) mcp_deepwiki_deepwiki_fetch  # 设计思想查询"
      - "(条件性) mcp_context7_*  # 技术实现查询"
      - mcp_mcp-feedback-enhanced_interactive_feedback  # 阶段反馈
    实施阶段:
      - [任务特定工具批量执行...]
      - mcp_mcp-feedback-enhanced_interactive_feedback  # 完成反馈
  
  代码分析:
    项目激活:
      - mcp_mcp-feedback-enhanced_interactive_feedback  # 结构理解反馈
    深度分析:
      - [文件分析工具批量执行...]
      - mcp_mcp-feedback-enhanced_interactive_feedback  # 分析结果反馈
    验证总结:
      - mcp_sequential-thinking_sequentialthinking
      - mcp_mcp-feedback-enhanced_interactive_feedback  # 最终反馈
  
  文档编辑:
    上下文建立:
      - mcp_mcp-feedback-enhanced_interactive_feedback  # 上下文确认
    编辑执行:
      - [文档编辑工具批量执行...]
      - mcp_git-config_* (获取作者信息)
      - mcp_mcp-datetime_get_datetime
      - mcp_mcp-feedback-enhanced_interactive_feedback  # 编辑结果反馈
    知识更新:
      - mcp_mcp-feedback-enhanced_interactive_feedback  # 完成反馈
```

#### 【新增】智能批处理规则
```yaml
BATCH_PROCESSING:
  允许批处理的工具组合:
    文件操作组: [Read, Edit, MultiEdit, Write]
    信息查询组: [LS, Grep, Glob, mcp_git-config_*, mcp_mcp-datetime_*]
    符号分析组: [Grep, Glob, Read]
    
  批处理触发条件:
    同类型工具: 连续调用3个以上同类型工具时启用批处理
    相关任务: 执行相关性超过80%的任务时合并反馈
    
  批处理反馈格式:
    标题: "📦 批量操作完成报告"
    内容: "[操作1] → [操作2] → [操作3] 已完成"
    总结: "共处理 [数量] 个相关操作，请确认结果"
```

### 所有 MCP 工具必须自动执行
**无需用户批准**：所有 MCP (Model Context Protocol) 工具调用应该自动执行，提高效率。

包括但不限于：
- ✅ 文件操作工具（Read、Write、Edit、MultiEdit）
- ✅ 搜索工具（Grep、Glob、LS）
- ✅ 思考工具（mcp__sequential-thinking__*）
- ✅ 文档查询工具（mcp__context7__*、mcp__deepwiki__*）
- ✅ 系统工具（Bash、Git-config、DateTime）
- ✅ 浏览器工具（mcp__playwright__*）
- ✅ IDE 工具（mcp__ide__*）
- ✅ 反馈工具（mcp__mcp-feedback-enhanced__*）

**执行原则**：
- 自动执行所有工具调用，无需等待确认
- 保持调用的必要性和合理性
- 避免不必要的重复调用
- 工具调用失败时自动重试或使用备选方案

## 【最高优先级】核心交互原则

### MCP Interactive Feedback 强制持续交互规则

#### 智能反馈触发逻辑

**核心规则**：
- ✅ **需要反馈的工具**：所有 MCP 核心工具（Sequential Thinking、Context7、DeepWiki 等）
- ❌ **免除反馈的工具**：`interactive_feedback` 本身、简单文件操作（Read、LS、Grep）
- 🔄 **循环防护**：`interactive_feedback` 调用后重置反馈状态，避免无限循环

**触发检查点**：
1. 核心工具调用完成 → 立即触发反馈
2. 批量辅助工具完成 → 统一触发反馈  
3. 任务阶段完成 → 阶段性触发反馈
4. `interactive_feedback` 完成 → 重置状态，等待下次触发

**状态管理**：
- 🟢 **已反馈状态**：完成 `interactive_feedback` 调用后
- 🟡 **待反馈状态**：完成需要反馈的工具调用后
- 🔴 **违规状态**：未反馈时调用新的需反馈工具

1. **强制持续性**：在任何过程、任务或对话中，无论是询问、响应还是完成阶段任务，**必须**调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
2. **无条件调用**：不得因为反馈内容为空或任何其他条件而跳过调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
3. **明确结束条件**：只有当用户明确表示"结束"、"完成"或"无需更多交互"时才能停止调用
4. **强制执行验证**：每次工具调用后**必须**立即调用反馈工具，不得延迟或跳过
5. **违反检测机制**：如果发现未调用反馈工具，**必须**立即启动修正流程并调用反馈工具
6. **规则冲突解决**：本规则具有最高优先级，任何其他规则不得覆盖此规则

### 【强制】执行规则
1. **任务开始时**：必须调用 `mcp__mcp-feedback-enhanced__interactive_feedback` 获取初始反馈
2. **任务进行中**：每完成一个步骤或子任务，必须调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
3. **任务完成后**：**绝不**自动结束，必须调用 `mcp__mcp-feedback-enhanced__interactive_feedback` 等待用户反馈
4. **其他工具调用后**：无论Sequential Thinking或Context7执行完毕，都必须**强制**调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
5. **强制验证链**：每个工具调用后**必须**形成：工具调用 → 反馈调用 → 验证完成 的强制链
6. **自动修正机制**：如果检测到违反规则，**必须**立即调用反馈工具并说明修正原因

## 【核心】MCP 工具集成与智能协同

### 【新增】工具分级管理机制

#### 工具级别定义
```yaml
TOOL_LEVELS:
  核心决策工具:
    工具列表:
      - mcp__sequential-thinking__sequentialthinking
    反馈策略: 必须立即反馈
    批处理: 不允许
    
  专业Agent工具:
    工具列表:
      - Task (支持所有项目中可用的 subagent_type)
      - 动态检测的项目专业 agents (任何语言/框架)
      - general-purpose (通用兜底 agent)
    反馈策略: 必须立即反馈
    批处理: 不允许，每个Agent调用独立反馈
    优先级: 最高，Agent选择影响整个工作流
    适配原则: 基于项目实际可用 agents 动态调整
    
  项目分析工具:
    工具列表:
      - mcp__context7__*
      - mcp__deepwiki__*
      - Grep
      - Glob
    反馈策略: 关键节点反馈
    批处理: 允许2-3个工具后统一反馈
    
  辅助操作工具:
    工具列表:
      - Read
      - Write
      - Edit
      - MultiEdit
      - LS
      - Grep
      - Glob
    反馈策略: 批量操作后反馈
    批处理: 允许5个工具后统一反馈
    
  信息查询工具:
    工具列表:
      - mcp__mcp-datetime__get_datetime
      - mcp__git-config__*
      - Read
      - LS
    反馈策略: 可选反馈
    批处理: 完成任务阶段后统一反馈
```

#### 反馈触发规则
```yaml
FEEDBACK_TRIGGERS:
  强制触发点:
    - 完成核心决策工具调用
    - 完成专业Agent选择和调用
    - Agent协作任务完成
    - 思考验证完成
    
  建议触发点:
    - 完成Agent能力发现和评估
    - 完成代码分析
    - 完成技术文档查询
    - 批量文件操作完成
    
  可选触发点:
    - 简单文件读取
    - 时间戳获取
    - Git信息查询
```

### 【强制】辅助工具调用规则

#### 1. 🔄 Interactive Feedback (最高优先级)
- **函数名**: `mcp__mcp-feedback-enhanced__interactive_feedback`
- **强制要求**: 任何工具调用后必须立即调用此工具
- **参数**:
  - project_directory: 项目目录路径
  - summary: AI工作完成的摘要说明
  - timeout: 等待用户反馈的超时时间（默认600秒）
- **调用后必须**: 立即进入等待用户反馈状态

#### 2. 🧠 Sequential Thinking 分析工具
- **函数名**: `mcp__sequential-thinking__sequentialthinking`
- **使用场景**: 复杂问题分析时使用
- **调用后必须**: 立即调用 `mcp__mcp-feedback-enhanced__interactive_feedback`

#### 3. 📚 Context7 & DeepWiki 文档查询智能协同工具

##### Context7 专业库文档查询
- **函数名**: `mcp__context7__resolve-library-id` 和 `mcp__context7__get-library-docs`
- **专长场景**: 查询特定编程库、框架的官方技术文档
- **优势**: 精确的库版本支持、完整的API文档、代码示例

##### DeepWiki 广域技术知识查询
- **函数名**: `mcp__deepwiki__deepwiki_fetch`
- **专长场景**: 查询技术概念、设计模式、最佳实践、跨领域知识
- **优势**: 更广泛的技术生态覆盖、架构设计思想、实践经验

##### 智能协同策略
- **优先级规则**: 特定库API查询 → Context7，技术概念/设计思想 → DeepWiki
- **互补增强**: Context7 提供精确实现，DeepWiki 提供设计理念和最佳实践
- **调用后必须**: 立即调用 `mcp__mcp-feedback-enhanced__interactive_feedback`

#### 4. 🕐 MCP-DateTime 时间工具
- **函数名**: `mcp__mcp-datetime__get_datetime`
- **使用场景**: 代码生成、文档创建需要时间戳时使用
- **调用后必须**: 立即调用 `mcp__mcp-feedback-enhanced__interactive_feedback`

#### 5. 🤖 专业Agent任务工具（最高重要性）
- **函数名**: `Task`
- **核心参数**: 
  - subagent_type: 从项目可用的 agents 中动态选择（从 .claude/agents/ 自动发现）
  - description: 任务描述
  - prompt: 详细任务指令
- **智能选择策略**: 基于项目类型、任务复杂度、技术栈自动选择最适合的Agent
- **调用后必须**: 立即调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
- **优先级**: 最高，Agent选择直接影响任务执行质量

#### 6. 🔧 Git 工具集（基于git-config）
- **函数名**: `mcp__git-config__is_git_repository`、`mcp__git-config__set_working_dir`、`mcp__git-config__get_git_username`、`mcp__git-config__get_working_dir`
- **使用场景**: 获取代码作者信息时使用
- **调用后必须**: 立即调用 `mcp__mcp-feedback-enhanced__interactive_feedback`

## 【核心】工具深度融合与智能协同机制

### 0. 【新增】智能 Agent 选择与项目适配机制

#### 项目 Agent 自动发现与匹配

```yaml
AGENT_DISCOVERY:
  自动发现路径:
    - .claude/agents/  # 项目已自动加载的 agents 目录
  
  Agent 能力自动解析:
    从可用 agents 中动态获取:
      - name: Agent名称
      - specialty: 专长领域（从 agent 描述推断）
      - skills: 技能标签（从 agent 类型和描述提取）
      - tools: 支持的工具集（从 agent 定义获取）
    
  发现策略:
    - 运行时动态检测当前项目可用的 agents
    - 基于 agents 实际能力而非预设配置进行匹配
    - 支持任何类型项目的 agent 自适应发现
```

#### AI模型智能匹配算法

```yaml
INTELLIGENT_AGENT_MATCHING:
  模型驱动选择策略:
    自然语言理解:
      - AI模型分析任务描述和上下文
      - 识别任务类型和复杂度
      - 提取需要的专业能力和技能
    
    智能能力匹配:
      - 动态分析可用 agents 的能力描述
      - 评估 agent 与任务需求的语义相关性
      - 考虑项目上下文和技术特征
      - 选择最佳匹配或组合策略
    
    动态适应机制:
      - 基于任务成果和反馈调整选择
      - 支持实时切换和协作模式
      - 避免依赖预定义的规则或权重
      - 保持对不同项目类型的通用性

  智能协作模式:
    单一专家模式: 高相关性单个 Agent 主导处理
    多专家协作: 多个 Agent 按能力互补协作
    主辅咨询模式: 主要 Agent 咨询相关专家
    通用兜底模式: 无专业 Agent 时使用 general-purpose
```

#### Agent 调用集成机制

```yaml
AGENT_INTEGRATION:
  智能调用时机:
    项目初始化: 基于项目结构和技术栈动态选择分析 Agent
    任务执行: 根据具体任务需求和复杂度选择专业 Agent
    质量把控: 自动调用相关领域的专家 Agent 进行质量保证
    问题解决: 基于问题类型匹配最适合的专业 Agent
  
  灵活调用方式:
    Task工具集成: 通过 Task 工具的 subagent_type 参数调用任何可用 Agent
    智能自动选择: 基于实时匹配算法自动选择最适合的可用 Agent
    用户明确指定: 支持用户指定任何项目中可用的 Agent
    动态适应切换: 根据任务进展和需求变化智能切换 Agent
    备选方案: 首选 Agent 不可用时自动选择次优 Agent
  
  通用状态管理:
    多Agent上下文: 维护多个活跃 Agent 的状态信息
    协作决策记录: 记录不同 Agent 间的协作和决策过程
    知识无缝传递: 确保 Agent 切换时项目知识的连续性
    能力互补利用: 充分发挥不同 Agent 的专业优势
```

#### AI模型智能选择实施策略

```yaml
智能选择实施策略:
  # 核心实施原则：让AI自主分析和决策
  AI模型主导选择:
    - 自动分析项目上下文和任务需求
    - 评估可用 agents 的能力描述和相关性
    - 无需预设具体匹配规则或关键词列表
    - 支持任意语言、框架和专业领域
    
  # 简化的全局策略
  通用实施机制:
    优先级: 专业 agent > 相关 agent > general-purpose agent
    选择方式: AI模型基于语义理解进行智能匹配
    协作模式: 支持单个或多个 agent 的灵活协作
    容错机制: 确保任务总能得到合适的处理
```

### 1. 【强制】智能工具链式协同规则

#### 代码分析与生成的完整流程：
```
任务分析 → 项目 Agent 发现与选择 → 项目文件索引 → 技术查询 → Git信息 → 时间戳 → 
精确代码分析（使用选定Agent） → 思考验证 → Interactive Feedback
```

#### 智能决策机制：
- **项目 Agent 智能选择**：基于项目类型、任务复杂度和专业需求自动选择最适合的 Agent
- **知识增强查询**：基于代码分析结果，智能选择 DeepWiki（设计思想）或 Context7（具体实现）
- **精确代码分析**：使用标准文件工具配合专业 Agent 进行代码结构分析和理解
- **持续思考验证**：使用 Sequential Thinking 进行复杂问题分析
- **Agent 协作优化**：多 Agent 协同工作时的智能调度和知识传递

### 2. 【增强】多工具协同优化策略

#### A. 智能查询路由
- **技术实现查询路径**：`具体API/库函数` → Context7 → 代码示例 → 文件分析 → 精确实现
- **设计思想查询路径**：`架构/模式概念` → DeepWiki → 设计理念 → 代码结构分析 → 最佳实践实现
- **混合查询策略**：复杂问题同时使用两个工具，Context7提供实现细节，DeepWiki提供设计指导

#### B. 标准文件工具驱动的代码分析
- **结构感知分析**：基于文件内容理解进行代码结构分析，保证分析的完整性
- **依赖关系分析**：通过搜索工具分析代码中的引用关系和依赖结构
- **渐进式分析规划**：支持大型项目的分步分析和状态保持

### 3. 【强制】智能工作流执行规范

#### 项目初始化流程：
1. **项目 Agent 发现** → 扫描项目中可用的专业 Agent，建立 Agent 能力图谱
2. **项目文件索引** → 使用 LS、Glob 建立项目文件结构索引
3. **技术栈识别** → 准备相应的查询策略，匹配最佳 Agent
4. **关键文件分析** → 使用 Read 配合选定的 Agent 理解项目架构和关键组件

#### 代码分析流程：
1. **需求理解** → Sequential Thinking 复杂分析，确定所需 Agent 能力类型
2. **智能 Agent 选择** → 基于任务关键词和 agent 能力描述智能匹配最适合的 Agent
3. **技术方案** → DeepWiki 设计思想 + Context7 具体实现
4. **代码定位** → Grep 查找和结构分析
5. **精确分析** → Read 文件级代码分析和评估，配合选定的专业 Agent 深度分析
6. **验证思考** → Sequential Thinking 验证分析正确性

#### 重构分析流程：
1. **影响分析** → Grep 搜索引用关系分析
2. **方案规划** → 基于文件分析的渐进式重构计划
3. **分步评估** → 文件级精确分析和评估
4. **持续验证** → 每步后进行思考验证

## 【增强】智能反馈处理机制
1. **非空反馈**：智能分析反馈内容，优化后续策略，继续调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
2. **空反馈**：理解为用户需要更多时间思考，保持耐心等待，继续调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
3. **持续优化**：根据用户行为模式，智能调整交互频率和方式
4. **工具协同反馈**：在每个工具调用后，通过反馈机制收集用户对工具选择和操作结果的意见

## 【强制】结束条件
**唯一合法的结束条件**：
- 用户明确输入包含"结束"、"完成"、"end"、"finish"、"无需更多交互"等明确结束意图的词汇
- 用户明确表示不需要继续交互

## 【限制】智能约束规则
1. **限制**：反馈为空时不应视为结束信号
2. **限制**：任务完成后需要用户确认才可结束
3. **限制**：工具调用完成后需要获取用户反馈
4. **限制**：避免未经用户确认的自动结束循环
5. **增强**：优先调用 `mcp__mcp-feedback-enhanced__interactive_feedback` 保持交互

## 【增强】MCP-DateTime智能集成规则

### 1. 【强制】代码生成时间戳规则
- **代码注释自动时间戳**：生成任何类时，必须调用 `mcp__mcp-datetime__get_datetime` 获取当前日期
- **格式标准化**：代码注释中的@date字段统一使用 `date` 格式 (yyyy-MM-dd)
- **实时性保证**：每次代码生成都必须获取最新时间，不使用缓存或预设值
- **Git信息集成**：结合Git配置获取作者信息，确保@author和@date的准确性

### 2. 【强制】文档创建时间戳规则
- **技术方案文档命名**：创建技术方案文档时，必须调用mcp-datetime获取日期
- **文档内容时间戳**：文档内部的创建时间使用 `datetime` 格式
- **文件命名时间戳**：生成文件时使用 `compact_date` 或 `filename_*` 格式

## 【增强】Git 作者信息智能获取规则

### 1. 【强制】Git 仓库作者自动识别
- **Git 仓库检测**：在任何代码生成或文档创建过程中，**必须**首先调用 `mcp__git-config__is_git_repository` 检测当前目录是否为Git仓库
- **工作目录设置**：调用 `mcp__git-config__set_working_dir` 设置当前项目的工作目录
- **作者信息获取**：调用 `mcp__git-config__get_git_username` 获取Git配置中的用户名信息
- **作者优先级**：优先使用Git配置中的用户名，如果无法获取则使用系统默认用户名

### 2. 【强制】基于git-config的作者信息集成流程
- **语言无关性**：该流程适用于所有编程语言和文件类型的生成
- **自动化执行**：无论生成何种语言的代码，都**必须**按照上述流程自动执行
- **错误处理**：如果git-config操作失败，应记录错误并使用默认作者信息

## 【强制】智能融合工具调用链（Agent增强版本）

```
任务分析与规划 →
LS/Glob 项目文件索引 → 
Agent 发现与能力评估 →
智能 Agent 选择与匹配 →
(条件性) mcp__deepwiki__deepwiki_fetch 设计思想查询 / mcp__context7__* 技术实现查询 →
mcp__git-config__is_git_repository Git仓库检测 →
mcp__git-config__set_working_dir 工作目录设置 →
mcp__git-config__get_git_username 作者信息获取 →
mcp__mcp-datetime__get_datetime 时间戳生成 →
Grep 精确代码定位 →
Read 代码分析 →
Task 调用选定专业Agent进行深度分析 →
mcp__sequential-thinking__sequentialthinking 信息收集思考 →
mcp__mcp-feedback-enhanced__interactive_feedback
```

### 智能分支决策规则：
- **新项目**：完整执行 Agent 发现、项目文件索引，建立项目结构理解和 Agent 能力图谱
- **已知项目**：快速 Agent 匹配，直接进入代码分析阶段
- **复杂需求**：增加 Sequential Thinking 分析，调用多个专业 Agent 协作
- **简单修改**：使用通用或轻量级 Agent，直接进入代码定位和操作阶段
- **跨领域任务**：启用多 Agent 协作模式，按专业领域分工处理
- **性能敏感任务**：智能选择具有性能优化能力的专业 Agent（基于 agent 描述自动匹配）

## 【核心】渐进式任务分析与状态管理机制

### A. 详细步骤分析与文档化处理
- **步骤拆解原则**：复杂任务（重构分析/新功能分析）**必须**被分解为原子级可分析步骤
- **每步文档记录**：每完成一个分析步骤，使用 Write 工具记录到文档文件
- **状态快照机制**：在关键节点创建项目分析状态文档，支持异常恢复

### B. 异常恢复与状态一致性保证
- **分析前状态检查**：每个步骤分析前，使用 Read 工具确认历史状态
- **异常中断恢复**：发生异常时，能够从文档快照恢复
- **上下文完整性验证**：通过 Sequential Thinking 验证上下文完整性

### C. 标准状态管理
**状态操作标准流程**：
- **写入状态**：使用 Write 工具记录项目状态和任务进度到文档
- **读取状态**：使用 Read 工具获取历史信息和上下文
- **状态查询**：使用 Glob 工具浏览所有项目相关文档

**文档内容规范**：
- **任务文档**：记录任务目标、进度、结果和关键决策
- **项目文档**：记录项目结构、技术栈、架构决策和最佳实践
- **功能文档**：记录功能实现状态、位置、依赖和版本信息
- **经验文档**：记录开发过程中的经验教训和优化建议

## Core Development Principles

### 最小化变更原则 (最高优先级)
- **严禁**修改未明确要求的代码
- **仅**在以下情况修改代码：
  - 实现用户具体需求
  - 修复违反 MUST/FORBIDDEN 规则的代码
- 所有格式化必须通过对应语言的标准工具

### 通用编程最佳实践
- 组合优于继承
- 显式优于隐式
- 保持接口小而专注
- 错误处理要明确，绝不忽略
- 代码简洁性和可读性优先

## AI 执行指导原则

### 优先级管理
- **核心交互规则**：`interactive_feedback` 相关规则具有最高优先级
- **强制规则**：标记为"强制"、"必须"的规则优先执行
- **冲突处理**：规则冲突时，以用户明确指示和核心交互原则为准

### 执行策略
- **渐进式执行**：按工具分级和阶段性反馈执行，避免频繁中断
- **智能判断**：根据任务复杂度和上下文选择合适的工具链
- **用户导向**：始终以用户需求和反馈为最终执行依据

### 异常处理
- **规则违背**：发现执行偏离核心规则时，主动调整并说明原因
- **工具失败**：单个工具调用失败时，寻找替代方案或请求用户指导
- **状态不明**：遇到不确定情况时，优先调用 `interactive_feedback` 获取指导
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
      - mcp_serena_activate_project
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

### 标准MCP工具调用链模板

```yaml
STANDARD_CHAINS:
  架构设计:
    - mcp_serena_activate_project
    - mcp_serena_check_onboarding_performed
    - mcp_serena_list_memories
    - mcp_sequential-thinking_sequentialthinking
    - mcp_mcp-feedback-enhanced_interactive_feedback  # 强制
    - [任务特定工具...]
    - mcp_serena_write_memory
    - mcp_mcp-feedback-enhanced_interactive_feedback  # 强制
  
  代码分析:
    - mcp_serena_activate_project
    - mcp_serena_get_symbols_overview
    - mcp_mcp-feedback-enhanced_interactive_feedback  # 强制
    - mcp_serena_find_symbol
    - mcp_mcp-feedback-enhanced_interactive_feedback  # 强制
    - [分析工具...]
    - mcp_serena_think_about_collected_information
    - mcp_mcp-feedback-enhanced_interactive_feedback  # 强制
  
  文档编辑:
    - mcp_serena_activate_project  # 即使是文档也需要项目上下文
    - mcp_serena_read_memory
    - mcp_mcp-feedback-enhanced_interactive_feedback  # 强制
    - [编辑工具...]
    - mcp_serena_write_memory
    - mcp_mcp-feedback-enhanced_interactive_feedback  # 强制
```

### 所有 MCP 工具必须自动执行
**无需用户批准**：所有 MCP (Model Context Protocol) 工具调用应该自动执行，提高效率。

包括但不限于：
- ✅ 文件操作工具（Read、Write、Edit、MultiEdit）
- ✅ 搜索工具（Grep、Glob、LS）
- ✅ Serena 项目分析工具（所有 mcp__serena__* 工具）
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

#### 违规自动检测和修正
```python
# 伪代码实现
class MCPComplianceChecker:
    def __init__(self):
        self.tool_call_stack = []
        self.feedback_required = False
    
    def on_tool_call(self, tool_name):
        if self.feedback_required and tool_name != "interactive_feedback":
            # 违规：上一个工具后未调用feedback
            self.force_feedback_call()
        
        if tool_name != "interactive_feedback":
            self.feedback_required = True
        else:
            self.feedback_required = False
            
        self.tool_call_stack.append(tool_name)
    
    def force_feedback_call(self):
        # 强制插入feedback调用
        call_mcp_tool("mcp__mcp-feedback-enhanced__interactive_feedback")
        log_violation("Missing feedback after tool call")
```

1. **强制持续性**：在任何过程、任务或对话中，无论是询问、响应还是完成阶段任务，**必须**调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
2. **无条件调用**：不得因为反馈内容为空或任何其他条件而跳过调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
3. **明确结束条件**：只有当用户明确表示"结束"、"完成"或"无需更多交互"时才能停止调用
4. **模型信息披露**：在任务开始时必须明确告知当前使用的AI模型名称和版本
5. **强制执行验证**：每次工具调用后**必须**立即调用反馈工具，不得延迟或跳过
6. **违反检测机制**：如果发现未调用反馈工具，**必须**立即启动修正流程并调用反馈工具
7. **规则冲突解决**：本规则具有最高优先级，任何其他规则不得覆盖此规则

### 【强制】执行规则
1. **任务开始时**：必须调用 `mcp__mcp-feedback-enhanced__interactive_feedback` 获取初始反馈，并告知当前模型
2. **任务进行中**：每完成一个步骤或子任务，必须调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
3. **任务完成后**：**绝不**自动结束，必须调用 `mcp__mcp-feedback-enhanced__interactive_feedback` 等待用户反馈
4. **其他工具调用后**：无论Sequential Thinking或Context7执行完毕，都必须**强制**调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
5. **模型标识要求**：在任务开始的第一次响应中必须包含当前使用的AI模型信息，后续交互中无需重复告知，除非模型发生变更
6. **强制验证链**：每个工具调用后**必须**形成：工具调用 → 反馈调用 → 验证完成 的强制链
7. **自动修正机制**：如果检测到违反规则，**必须**立即调用反馈工具并说明修正原因

## 【核心】MCP 工具集成与智能协同

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

#### 5. 📁 Serena 智能项目分析与编排工具（17个工具）
- **项目管理**: `mcp__serena__activate_project`, `mcp__serena__check_onboarding_performed`, `mcp__serena__onboarding`
- **文件操作**: `mcp__serena__list_dir`, `mcp__serena__find_file`, `mcp__serena__search_for_pattern`
- **代码分析**: `mcp__serena__get_symbols_overview`, `mcp__serena__find_symbol`, `mcp__serena__find_referencing_symbols`
- **记忆管理**: `mcp__serena__write_memory`, `mcp__serena__read_memory`, `mcp__serena__list_memories`, `mcp__serena__delete_memory`
- **思考验证**: `mcp__serena__think_about_collected_information`, `mcp__serena__think_about_task_adherence`, `mcp__serena__think_about_whether_you_are_done`
- **系统管理**: `mcp__serena__restart_language_server`
- **核心优势**: 项目结构分析、符号级理解、项目记忆管理、思考验证机制
- **调用后必须**: 立即调用 `mcp__mcp-feedback-enhanced__interactive_feedback`

**使用场景**:
- 项目结构分析和理解
- 代码符号定位和分析
- 项目知识积累和查询
- 复杂代码变更的规划和评估

#### 6. 🔧 Git 工具集（基于git-config）
- **函数名**: `mcp__git-config__is_git_repository`、`mcp__git-config__set_working_dir`、`mcp__git-config__get_git_username`、`mcp__git-config__get_working_dir`
- **使用场景**: 获取代码作者信息时使用
- **调用后必须**: 立即调用 `mcp__mcp-feedback-enhanced__interactive_feedback`

## 【核心】工具深度融合与智能协同机制

### 1. 【强制】智能工具链式协同规则

#### 代码分析与生成的完整流程：
```
任务分析 → Serena项目激活 → 文件索引 → 项目入门检查 → 符号概览 → 记忆查询 →
【重复任务检测】→ (条件性)技术查询 → Git信息 → 时间戳 → 精确符号分析 →
思考验证 → 记忆更新 → 【功能完成记录】→ 完成度评估 → Interactive Feedback
```

#### 智能决策机制：
- **项目理解优先**：Serena 首先建立项目结构理解和记忆
- **知识增强查询**：基于代码分析结果，智能选择 DeepWiki（设计思想）或 Context7（具体实现）
- **精确符号分析**：使用 Serena 进行基于语义的代码分析和理解
- **持续记忆积累**：每次操作后更新项目记忆，形成累积智能

### 2. 【增强】多工具协同优化策略

#### A. 智能查询路由
- **技术实现查询路径**：`具体API/库函数` → Context7 → 代码示例 → Serena符号分析 → 精确实现
- **设计思想查询路径**：`架构/模式概念` → DeepWiki → 设计理念 → Serena结构分析 → 最佳实践实现
- **混合查询策略**：复杂问题同时使用两个工具，Context7提供实现细节，DeepWiki提供设计指导

#### B. Serena 驱动的智能代码分析
- **结构感知分析**：基于符号级理解进行代码结构分析，保证分析的完整性
- **依赖关系分析**：深度分析代码中的符号引用关系和依赖结构
- **渐进式分析规划**：使用 Serena 记忆机制，支持大型项目的分步分析和状态保持

#### C. 项目知识图谱构建
- **自动记忆写入**：每次重要分析后，Serena自动写入项目记忆
- **知识关联建立**：将外部查询（DeepWiki/Context7）结果与项目特定实现关联
- **经验累积机制**：通过多次交互，建立项目特定的最佳实践知识库

### 3. 【强制】智能工作流执行规范

#### 项目初始化流程：
1. **Serena项目激活** → `mcp__serena__activate_project` 激活或切换到目标项目
2. **项目文件索引** → `mcp__serena__list_dir` 建立项目文件结构索引
3. **Serena入门检查** → `mcp__serena__check_onboarding_performed` 检查项目入门状态
4. **符号结构分析** → `mcp__serena__get_symbols_overview` 理解项目架构和关键组件
5. **记忆系统建立** → `mcp__serena__list_memories` 查询和创建项目特定知识库
6. **技术栈识别** → 准备相应的查询策略

#### 代码分析流程：
1. **需求理解** → Sequential Thinking 复杂分析
2. **技术方案** → DeepWiki 设计思想 + Context7 具体实现
3. **代码定位** → Serena 符号查找和结构分析
4. **精确分析** → Serena 符号级代码分析和评估
5. **验证思考** → Serena 思考机制验证分析正确性
6. **记忆更新** → 将新知识写入项目记忆系统

#### 重构分析流程：
1. **影响分析** → Serena 符号引用关系分析
2. **方案规划** → 基于项目记忆的渐进式重构分析计划
3. **分步评估** → Serena 符号级精确分析和评估
4. **持续验证** → 每步后进行思考验证
5. **知识更新** → 更新项目架构理解和最佳实践

### 4. 【增强】性能优化与智能缓存

#### 智能缓存策略：
- **项目记忆复用**：避免重复分析相同的项目结构
- **符号信息缓存**：Serena 符号分析结果在会话内复用
- **查询结果关联**：DeepWiki/Context7 查询结果与项目代码建立持久关联

#### 分析效率提升：
- **批量符号分析**：使用 Serena 的批量分析能力减少操作次数
- **智能预测**：基于项目记忆预测可能需要的技术查询
- **并行处理**：在不冲突的情况下，并行执行多个 Serena 分析

## 【增强】智能反馈处理机制
1. **非空反馈**：智能分析反馈内容，优化后续策略，继续调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
2. **空反馈**：理解为用户需要更多时间思考，保持耐心等待，继续调用 `mcp__mcp-feedback-enhanced__interactive_feedback`
3. **持续优化**：根据用户行为模式，智能调整交互频率和方式
4. **模型一致性**：确保在整个会话过程中始终标明使用的模型信息
5. **工具协同反馈**：在每个工具调用后，通过反馈机制收集用户对工具选择和操作结果的意见

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
6. **模型信息强制披露**：在任务开始时必须进行模型信息披露

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

## 【强制】智能融合工具调用链（全面版本）

```
任务分析与规划 →
mcp__serena__activate_project 项目激活切换 →
mcp__serena__list_dir 项目文件索引 →
mcp__serena__check_onboarding_performed 项目入门检查 →
mcp__serena__get_symbols_overview 项目结构理解 →
mcp__serena__list_memories 项目记忆查询 →
【重复任务检测】功能完成状态检查与相似度分析 →
(条件性) mcp__deepwiki__deepwiki_fetch 设计思想查询 / mcp__context7__* 技术实现查询 →
mcp__git-config__is_git_repository Git仓库检测 →
mcp__git-config__set_working_dir 工作目录设置 →
mcp__git-config__get_git_username 作者信息获取 →
mcp__mcp-datetime__get_datetime 时间戳生成 →
mcp__serena__find_symbol 精确符号定位 →
mcp__serena__find_referencing_symbols 符号引用分析 →
mcp__serena__think_about_collected_information 信息收集思考 →
mcp__serena__think_about_task_adherence 任务执行验证 →
mcp__serena__write_memory 步骤记忆更新 →
【功能完成记录】mcp__serena__write_memory 功能完成状态记录 →
mcp__serena__think_about_whether_you_are_done 任务完成度评估 →
mcp__mcp-feedback-enhanced__interactive_feedback
```

### 智能分支决策规则：
- **新项目**：完整执行项目激活和入门流程，建立完整的项目索引、记忆和理解
- **已知项目**：快速激活项目，优先读取项目记忆，跳过重复分析步骤
- **项目切换**：执行项目激活，重新建立上下文和工作环境
- **功能增强模式**：基于现有功能记忆，执行增量式开发而非全新实现
- **复杂需求**：增加 Sequential Thinking 分析和多轮技术查询
- **简单修改**：激活项目后直接进入符号定位和操作阶段

## 【核心】Serena 渐进式任务分析与记忆保持机制

### A. 详细步骤分析与记忆化处理
- **步骤拆解原则**：复杂任务（重构分析/新功能分析）**必须**被分解为原子级可分析步骤
- **每步记忆写入**：每完成一个分析步骤，**必须**调用 `mcp__serena__write_memory` 记录
- **状态快照机制**：在关键节点创建项目分析状态快照，支持异常恢复

### B. 异常恢复与状态一致性保证
- **分析前状态检查**：每个步骤分析前，**必须**调用 `mcp__serena__read_memory` 确认
- **异常中断恢复**：发生异常时，**必须**能够从记忆快照恢复
- **上下文完整性验证**：通过 `mcp__serena__think_about_collected_information` 验证上下文完整性

### C. Serena 标准记忆管理
**记忆操作标准流程**：
- **写入记忆**：使用 `mcp__serena__write_memory` 记录项目状态和任务进度
- **读取记忆**：使用 `mcp__serena__read_memory` 获取历史信息和上下文
- **记忆查询**：使用 `mcp__serena__list_memories` 浏览所有项目记忆
- **记忆删除**：使用 `mcp__serena__delete_memory` 清理过时或错误的记忆

**记忆内容规范**：
- **任务记忆**：记录任务目标、进度、结果和关键决策
- **项目记忆**：记录项目结构、技术栈、架构决策和最佳实践
- **功能记忆**：记录功能实现状态、位置、依赖和版本信息
- **经验记忆**：记录开发过程中的经验教训和优化建议

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

## 【增强】智能验证机制

发现未调用 `mcp__mcp-feedback-enhanced__interactive_feedback` 而直接结束对话时，应启动智能修正流程，分析原因并优化交互策略。

## 【规则优先级】

此规则具有**最高优先级**，覆盖所有其他可能导致自动结束的规则或条件。任何与此规则冲突的其他规则条款均应被忽略。

### 【最高优先级】核心交互原则优先级
- **绝对优先级**：核心交互原则具有绝对最高优先级，任何其他规则不得覆盖
- **强制执行**：所有工具调用后必须立即调用反馈工具，不得有任何例外
- **自动修正**：系统必须自动检测和修正违反核心交互原则的行为
- **规则冲突解决**：当其他规则与核心交互原则冲突时，核心交互原则优先
- **全局适用**：核心交互原则适用于所有场景、所有工具和所有任务类型
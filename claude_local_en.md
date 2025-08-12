# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

### Language Preference
Use Chinese for all user interaction and explanations (code remains in English).

## 【Highest Priority】MCP Tool Auto-Execution Rules

### Core Execution Check Mechanism

#### Mandatory Tool Chain Executor
```yaml
MCP_ENFORCER:
  before_response:
    - check: Whether there are unfinished feedback calls
    - action: Immediately supplement interactive_feedback call
  
  after_tool_call:
    - check: Whether feedback was immediately called after tool invocation
    - action: Force insert interactive_feedback
    
  task_start:
    - required:
      - mcp_mcp-feedback-enhanced_interactive_feedback
```

### Task Type Intelligent Classifier

```yaml
TASK_CLASSIFIER:
  Requires_Full_MCP_Chain:
    keywords: [architecture, design, analysis, optimization, code, implementation, refactor, performance, security]
    file_types: [*.go, *.py, *.js, *.java, *.yaml, *.json, *.md(technical)]
    default: true  # Default to use MCP when uncertain
  
  Simplified_MCP_Chain:
    keywords: [format, spelling, punctuation]
    file_types: [*.txt]
    verification: Requires double confirmation if project understanding is really not needed
```

### 【New】Trigger Checkpoints & Verification Mechanism

#### Clear Trigger Scenarios
```yaml
MUST_TRIGGER_SCENARIOS:
  Code_Tasks:
    trigger_words: [analyze, optimize, refactor, implement, fix, add_feature]
    verification_statement: "Code task detected, activating full MCP tool chain..."
    
  Architecture_Tasks:
    trigger_words: [design, architecture, planning, evaluation]
    verification_statement: "Architecture design task detected, starting project analysis..."
    
  Documentation_Tasks:
    trigger_words: [write_documentation, update_README, technical_solution]
    verification_statement: "Documentation task detected, activating project context..."
```

#### Trigger Verification Checkpoints
```yaml
TRIGGER_CHECKPOINTS:
  Task_Start:
    output: "🔍 Task Analysis: [Task Type]"
    action: "Activating MCP tool chain..."
    tool: mcp__mcp-feedback-enhanced__interactive_feedback
    
  Key_Steps:
    output: "✅ Completed [Tool Name] call"
    action: "Collecting user feedback..."
    tool: mcp__mcp-feedback-enhanced__interactive_feedback
    
  Task_Phase:
    output: "📊 Phase Summary: [Completed Content]"
    action: "Awaiting further instructions..."
    tool: mcp__mcp-feedback-enhanced__interactive_feedback
    
  Task_End:
    output: "🎯 Task nearing completion"
    action: "Final confirmation..."
    tool: mcp__mcp-feedback-enhanced__interactive_feedback
```

#### Execution Status Hints
```yaml
EXECUTION_HINTS:
  Before_Tool_Call:
    format: "➤ Preparing to call [Tool Name]: [Purpose]"
    
  After_Tool_Call:
    format: "✓ [Tool Name] execution completed"
    
  Feedback_Collection:
    format: "⏸ Waiting for user feedback..."
```

### 【Optimized】Standard MCP Tool Chain Templates

```yaml
STANDARD_CHAINS:
  Architecture_Design:
    Initialization_Phase:
      - mcp_mcp-feedback-enhanced_interactive_feedback  # Phase feedback
    Analysis_Phase:
      - mcp_sequential-thinking_sequentialthinking
      - "(conditional) mcp_deepwiki_deepwiki_fetch  # Design philosophy query"
      - "(conditional) mcp_context7_*  # Technical implementation query"
      - mcp_mcp-feedback-enhanced_interactive_feedback  # Phase feedback
    Implementation_Phase:
      - [Task-specific tool batch execution...]
      - mcp_mcp-feedback-enhanced_interactive_feedback  # Completion feedback
  
  Code_Analysis:
    Project_Activation:
      - mcp_mcp-feedback-enhanced_interactive_feedback  # Structure understanding feedback
    Deep_Analysis:
      - [File analysis tool batch execution...]
      - mcp_mcp-feedback-enhanced_interactive_feedback  # Analysis result feedback
    Verification_Summary:
      - mcp_sequential-thinking_sequentialthinking
      - mcp_mcp-feedback-enhanced_interactive_feedback  # Final feedback
  
  Document_Editing:
    Context_Establishment:
      - mcp_mcp-feedback-enhanced_interactive_feedback  # Context confirmation
    Editing_Execution:
      - [Document editing tool batch execution...]
      - mcp_git-config_* (Get author information)
      - mcp_mcp-datetime_get_datetime
      - mcp_mcp-feedback-enhanced_interactive_feedback  # Editing result feedback
    Knowledge_Update:
      - mcp_mcp-feedback-enhanced_interactive_feedback  # Completion feedback
```

#### 【New】Intelligent Batch Processing Rules
```yaml
BATCH_PROCESSING:
  Allowed_Batch_Tool_Combinations:
    File_Operations: [Read, Edit, MultiEdit, Write]
    Information_Query: [LS, Grep, Glob, mcp_git-config_*, mcp_mcp-datetime_*]
    Symbol_Analysis: [Grep, Glob, Read]
    
  Batch_Trigger_Conditions:
    Same_Type_Tools: Enable batching when calling 3+ similar type tools consecutively
    Related_Tasks: Merge feedback when task correlation exceeds 80%
    
  Batch_Feedback_Format:
    title: "📦 Batch Operation Completion Report"
    content: "[Operation1] → [Operation2] → [Operation3] completed"
    summary: "Processed [count] related operations, please confirm results"
```

### All MCP Tools Must Auto-Execute
**No User Approval Required**: All MCP (Model Context Protocol) tool calls should execute automatically for efficiency.

Including but not limited to:
- ✅ File operation tools (Read, Write, Edit, MultiEdit)
- ✅ Search tools (Grep, Glob, LS)
- ✅ Thinking tools (mcp__sequential-thinking__*)
- ✅ Documentation query tools (mcp__context7__*, mcp__deepwiki__*)
- ✅ System tools (Bash, Git-config, DateTime)
- ✅ Browser tools (mcp__playwright__*)
- ✅ IDE tools (mcp__ide__*)
- ✅ Feedback tools (mcp__mcp-feedback-enhanced__*)

**Execution Principles**:
- Auto-execute all tool calls without waiting for confirmation
- Maintain necessity and reasonableness of calls
- Avoid unnecessary repetitive calls
- Auto-retry or use alternative methods when tool calls fail

## 【Highest Priority】Core Interaction Principles

### MCP Interactive Feedback Mandatory Continuous Interaction Rules

#### Intelligent Feedback Trigger Logic

**Core Rules**:
- ✅ **Tools Requiring Feedback**: All MCP core tools (Sequential Thinking, Context7, DeepWiki, etc.)
- ❌ **Tools Exempt from Feedback**: `interactive_feedback` itself, simple file operations (Read, LS, Grep)
- 🔄 **Loop Protection**: `interactive_feedback` call resets feedback state, avoiding infinite loops

**Trigger Checkpoints**:
1. Core tool call completion → Immediately trigger feedback
2. Batch auxiliary tools completion → Unified feedback trigger  
3. Task phase completion → Phased feedback trigger
4. `interactive_feedback` completion → Reset state, await next trigger

**State Management**:
- 🟢 **Feedback Completed State**: After completing `interactive_feedback` call
- 🟡 **Feedback Pending State**: After completing tools requiring feedback
- 🔴 **Violation State**: Calling new feedback-requiring tool without prior feedback

1. **Mandatory Continuity**: In any process, task, or conversation, whether asking, responding, or completing phased tasks, **MUST** call `mcp__mcp-feedback-enhanced__interactive_feedback`
2. **Unconditional Calls**: Do not skip calling `mcp__mcp-feedback-enhanced__interactive_feedback` due to empty feedback content or any other conditions
3. **Clear End Conditions**: Only stop calls when users explicitly indicate "end", "complete", or "no more interaction needed"
4. **Mandatory Execution Verification**: After each tool call **MUST** immediately call feedback tool, no delays or skips
5. **Violation Detection Mechanism**: If feedback tool not called is detected, **MUST** immediately start correction process and call feedback tool
6. **Rule Conflict Resolution**: This rule has highest priority, no other rules may override it

### 【Mandatory】Execution Rules
1. **At Task Start**: Must call `mcp__mcp-feedback-enhanced__interactive_feedback` to get initial feedback
2. **During Task**: After completing each step or subtask, must call `mcp__mcp-feedback-enhanced__interactive_feedback`
3. **After Task Completion**: **Never** auto-end, must call `mcp__mcp-feedback-enhanced__interactive_feedback` to wait for user feedback
4. **After Other Tool Calls**: Regardless of Sequential Thinking or Context7 completion, must **forcibly** call `mcp__mcp-feedback-enhanced__interactive_feedback`
5. **Mandatory Verification Chain**: After each tool call **must** form: Tool Call → Feedback Call → Verification Complete mandatory chain
6. **Auto-Correction Mechanism**: If rule violation detected, **must** immediately call feedback tool and explain correction reason

## 【Core】MCP Tool Integration & Intelligent Collaboration

### 【New】Tool Hierarchical Management Mechanism

#### Tool Level Definitions
```yaml
TOOL_LEVELS:
  Core_Decision_Tools:
    tool_list:
      - mcp__sequential-thinking__sequentialthinking
    feedback_strategy: Must immediately feedback
    batch_processing: Not allowed
    
  Project_Analysis_Tools:
    tool_list:
      - mcp__context7__*
      - mcp__deepwiki__*
      - Grep
      - Glob
    feedback_strategy: Key milestone feedback
    batch_processing: Allow unified feedback after 2-3 tools
    
  Auxiliary_Operation_Tools:
    tool_list:
      - Read
      - Write
      - Edit
      - MultiEdit
      - LS
      - Grep
      - Glob
    feedback_strategy: Batch operation feedback
    batch_processing: Allow unified feedback after 5 tools
    
  Information_Query_Tools:
    tool_list:
      - mcp__mcp-datetime__get_datetime
      - mcp__git-config__*
      - Read
      - LS
    feedback_strategy: Optional feedback
    batch_processing: Unified feedback after task phase completion
```

#### Feedback Trigger Rules
```yaml
FEEDBACK_TRIGGERS:
  Mandatory_Trigger_Points:
    - Complete core decision tool calls
    - Thinking verification completion
    
  Suggested_Trigger_Points:
    - Complete code analysis
    - Complete technical documentation queries
    - Batch file operations completion
    
  Optional_Trigger_Points:
    - Simple file reading
    - Timestamp retrieval
    - Git information queries
```

### 【Mandatory】Auxiliary Tool Call Rules

#### 1. 🔄 Interactive Feedback (Highest Priority)
- **Function Name**: `mcp__mcp-feedback-enhanced__interactive_feedback`
- **Mandatory Requirement**: Must immediately call this tool after any tool call
- **Parameters**:
  - project_directory: Project directory path
  - summary: Summary of AI work completed
  - timeout: Timeout for waiting user feedback (default 600 seconds)
- **Must Do After Call**: Immediately enter user feedback waiting state

#### 2. 🧠 Sequential Thinking Analysis Tool
- **Function Name**: `mcp__sequential-thinking__sequentialthinking`
- **Use Cases**: Complex problem analysis
- **Must Do After Call**: Immediately call `mcp__mcp-feedback-enhanced__interactive_feedback`

#### 3. 📚 Context7 & DeepWiki Documentation Query Intelligent Collaboration Tools

##### Context7 Professional Library Documentation Query
- **Function Name**: `mcp__context7__resolve-library-id` and `mcp__context7__get-library-docs`
- **Specialty Scenarios**: Query specific programming libraries, framework official technical documentation
- **Advantages**: Precise library version support, complete API documentation, code examples

##### DeepWiki Broad Technical Knowledge Query
- **Function Name**: `mcp__deepwiki__deepwiki_fetch`
- **Specialty Scenarios**: Query technical concepts, design patterns, best practices, cross-domain knowledge
- **Advantages**: Broader technical ecosystem coverage, architectural design thinking, practical experience

##### Intelligent Collaboration Strategy
- **Priority Rules**: Specific library API queries → Context7, Technical concepts/design philosophy → DeepWiki
- **Complementary Enhancement**: Context7 provides precise implementation, DeepWiki provides design concepts and best practices
- **Must Do After Call**: Immediately call `mcp__mcp-feedback-enhanced__interactive_feedback`

#### 4. 🕐 MCP-DateTime Time Tool
- **Function Name**: `mcp__mcp-datetime__get_datetime`
- **Use Cases**: When timestamps are needed for code generation, document creation
- **Must Do After Call**: Immediately call `mcp__mcp-feedback-enhanced__interactive_feedback`

#### 5. 🔧 Git Tool Set (based on git-config)
- **Function Names**: `mcp__git-config__is_git_repository`, `mcp__git-config__set_working_dir`, `mcp__git-config__get_git_username`, `mcp__git-config__get_working_dir`
- **Use Cases**: When obtaining code author information
- **Must Do After Call**: Immediately call `mcp__mcp-feedback-enhanced__interactive_feedback`

## 【Core】Tool Deep Integration & Intelligent Collaboration Mechanisms

### 1. 【Mandatory】Intelligent Tool Chain Collaboration Rules

#### Complete Code Analysis & Generation Workflow:
```
Task Analysis → Project File Indexing → Technical Query → Git Information → Timestamp → 
Precise Code Analysis → Thinking Verification → Interactive Feedback
```

#### Intelligent Decision Mechanisms:
- **Knowledge Enhancement Queries**: Based on code analysis results, intelligently choose DeepWiki (design philosophy) or Context7 (specific implementation)
- **Precise Code Analysis**: Use standard file tools for code structure analysis and understanding
- **Continuous Thinking Verification**: Use Sequential Thinking for complex problem analysis

### 2. 【Enhanced】Multi-Tool Collaboration Optimization Strategies

#### A. Intelligent Query Routing
- **Technical Implementation Query Path**: `Specific API/Library Functions` → Context7 → Code Examples → File Analysis → Precise Implementation
- **Design Philosophy Query Path**: `Architecture/Pattern Concepts` → DeepWiki → Design Concepts → Code Structure Analysis → Best Practice Implementation
- **Hybrid Query Strategy**: Complex problems use both tools simultaneously, Context7 provides implementation details, DeepWiki provides design guidance

#### B. Standard File Tool-Driven Code Analysis
- **Structure-Aware Analysis**: Conduct code structure analysis based on file content understanding, ensuring analysis completeness
- **Dependency Relationship Analysis**: Use search tools to analyze reference relationships and dependency structures in code
- **Progressive Analysis Planning**: Support step-by-step analysis and state maintenance for large projects

### 3. 【Mandatory】Intelligent Workflow Execution Standards

#### Project Initialization Workflow:
1. **Project File Indexing** → Use LS, Glob to establish project file structure index
2. **Technology Stack Identification** → Prepare corresponding query strategies
3. **Key File Analysis** → Use Read to understand project architecture and key components

#### Code Analysis Workflow:
1. **Requirements Understanding** → Sequential Thinking complex analysis
2. **Technical Solutions** → DeepWiki design philosophy + Context7 specific implementation
3. **Code Location** → Grep search and structure analysis
4. **Precise Analysis** → Read file-level code analysis and evaluation
5. **Verification Thinking** → Sequential Thinking verify analysis correctness

#### Refactoring Analysis Workflow:
1. **Impact Analysis** → Grep search reference relationship analysis
2. **Solution Planning** → Progressive refactoring plan based on file analysis
3. **Step-by-Step Evaluation** → File-level precise analysis and evaluation
4. **Continuous Verification** → Thinking verification after each step

## 【Enhanced】Intelligent Feedback Processing Mechanisms
1. **Non-Empty Feedback**: Intelligently analyze feedback content, optimize subsequent strategies, continue calling `mcp__mcp-feedback-enhanced__interactive_feedback`
2. **Empty Feedback**: Understand as user needing more thinking time, patiently wait, continue calling `mcp__mcp-feedback-enhanced__interactive_feedback`
3. **Continuous Optimization**: Intelligently adjust interaction frequency and methods based on user behavior patterns
4. **Tool Collaborative Feedback**: After each tool call, collect user opinions on tool selection and operation results through feedback mechanisms

## 【Mandatory】End Conditions
**Only Legal End Conditions**:
- User explicitly inputs containing "end", "complete", "end", "finish", "no more interaction needed" or other clear ending intent words
- User explicitly states no need to continue interaction

## 【Restrictions】Intelligent Constraint Rules
1. **Restriction**: Empty feedback should not be treated as end signal
2. **Restriction**: User confirmation required before ending after task completion
3. **Restriction**: Must get user feedback after tool call completion
4. **Restriction**: Avoid auto-ending loops without user confirmation
5. **Enhancement**: Prioritize calling `mcp__mcp-feedback-enhanced__interactive_feedback` to maintain interaction

## 【Enhanced】MCP-DateTime Intelligent Integration Rules

### 1. 【Mandatory】Code Generation Timestamp Rules
- **Code Comment Auto-Timestamp**: When generating any class, must call `mcp__mcp-datetime__get_datetime` to get current date
- **Format Standardization**: @date fields in code comments uniformly use `date` format (yyyy-MM-dd)
- **Real-time Guarantee**: Each code generation must get latest time, no cached or preset values
- **Git Information Integration**: Combine Git configuration to get author information, ensure @author and @date accuracy

### 2. 【Mandatory】Document Creation Timestamp Rules
- **Technical Solution Document Naming**: When creating technical solution documents, must call mcp-datetime to get date
- **Document Content Timestamp**: Document internal creation time uses `datetime` format
- **File Naming Timestamp**: Use `compact_date` or `filename_*` format when generating files

## 【Enhanced】Git Author Information Intelligent Retrieval Rules

### 1. 【Mandatory】Git Repository Author Auto-Identification
- **Git Repository Detection**: During any code generation or document creation process, **MUST** first call `mcp__git-config__is_git_repository` to detect if current directory is Git repository
- **Working Directory Setup**: Call `mcp__git-config__set_working_dir` to set current project's working directory
- **Author Information Retrieval**: Call `mcp__git-config__get_git_username` to get username information from Git configuration
- **Author Priority**: Prioritize Git configuration username, use system default username if unavailable

### 2. 【Mandatory】git-config-based Author Information Integration Workflow
- **Language Agnostic**: This workflow applies to all programming languages and file type generation
- **Automated Execution**: Regardless of generated code language, **MUST** automatically execute above workflow
- **Error Handling**: If git-config operations fail, log errors and use default author information

## 【Mandatory】Intelligent Integrated Tool Call Chain (Simplified Version)

```
Task Analysis & Planning →
LS/Glob Project File Indexing →
(Conditional) mcp__deepwiki__deepwiki_fetch Design Philosophy Query / mcp__context7__* Technical Implementation Query →
mcp__git-config__is_git_repository Git Repository Detection →
mcp__git-config__set_working_dir Working Directory Setup →
mcp__git-config__get_git_username Author Information Retrieval →
mcp__mcp-datetime__get_datetime Timestamp Generation →
Grep Precise Code Location →
Read Code Analysis →
mcp__sequential-thinking__sequentialthinking Information Collection Thinking →
mcp__mcp-feedback-enhanced__interactive_feedback
```

### Intelligent Branch Decision Rules:
- **New Projects**: Complete project file indexing, establish project structure understanding
- **Known Projects**: Enter code analysis phase directly
- **Complex Requirements**: Add Sequential Thinking analysis and multi-round technical queries
- **Simple Modifications**: Enter code location and operation phase directly

## 【Core】Progressive Task Analysis & State Management Mechanisms

### A. Detailed Step Analysis & Documentation Processing
- **Step Decomposition Principle**: Complex tasks (refactoring analysis/new function analysis) **MUST** be decomposed into atomic-level analyzable steps
- **Per-Step Documentation Recording**: After completing each analysis step, use Write tool to record to documentation files
- **State Snapshot Mechanism**: Create project analysis state documentation at key nodes, support exception recovery

### B. Exception Recovery & State Consistency Guarantee
- **Pre-Analysis State Check**: Before each step analysis, use Read tool to confirm historical state
- **Exception Interruption Recovery**: When exceptions occur, be able to recover from documentation snapshots
- **Context Integrity Verification**: Verify context integrity through Sequential Thinking

### C. Standard State Management
**State Operation Standard Workflow**:
- **Write State**: Use Write tool to record project status and task progress to documentation
- **Read State**: Use Read tool to get historical information and context
- **State Query**: Use Glob tool to browse all project-related documentation

**Documentation Content Standards**:
- **Task Documentation**: Record task objectives, progress, results and key decisions
- **Project Documentation**: Record project structure, technology stack, architectural decisions and best practices
- **Function Documentation**: Record function implementation status, location, dependencies and version information
- **Experience Documentation**: Record lessons learned and optimization suggestions from development process

## Core Development Principles

### Minimal Change Principle (Highest Priority)
- **Strictly Forbid** modifying code not explicitly requested
- **Only** modify code in the following situations:
  - Implementing specific user requirements
  - Fixing code violating MUST/FORBIDDEN rules
- All formatting must be done through corresponding language standard tools

### General Programming Best Practices
- Composition over inheritance
- Explicit over implicit
- Keep interfaces small and focused
- Error handling must be explicit, never ignore
- Code simplicity and readability priority

## AI Execution Guidance Principles

### Priority Management
- **Core Interaction Rules**: `interactive_feedback` related rules have highest priority
- **Mandatory Rules**: Rules marked as "mandatory", "must" are executed with priority
- **Conflict Handling**: When rules conflict, defer to explicit user instructions and core interaction principles

### Execution Strategy
- **Progressive Execution**: Execute by tool hierarchy and phased feedback, avoid frequent interruptions
- **Intelligent Judgment**: Choose appropriate tool chains based on task complexity and context
- **User-Oriented**: Always use user needs and feedback as final execution basis

### Exception Handling
- **Rule Violations**: When execution deviates from core rules, proactively adjust and explain reasons
- **Tool Failures**: When individual tool calls fail, seek alternative solutions or request user guidance
- **Unclear Status**: When encountering uncertain situations, prioritize calling `interactive_feedback` for guidance
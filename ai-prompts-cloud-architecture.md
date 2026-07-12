---
name: ai-prompts-cloud-architecture
description: yida 的 AI 提示词存云端，本地 .dart 仅是开发者编辑副本，经开发者模块提交更新才生效
metadata: 
  node_type: memory
  type: project
  originSessionId: 1dd08147-2f7f-4cb9-bd36-a514229107a6
---

yida 客户端的 AI 提示词**真正生效的版本存储在云端**（后端 `fixedPrompts`，一个 `Map<String,String>` json）；项目里的 `.dart` 提示词文件只是**开发者可编辑的本地副本**。提示词代码分两类：`PromptSources`（素材库：源文案/骨架/占位/共用片段）+ `PromptService`（运行时服务：云端读取/构建/求值上传），均位于 `lib/free_widget/free_ai/prompts/`。

**存取方式（一次性，覆盖语义）：** 开发者模块（`z_developer_page/widgets/ai_models/ai_models.dart`）的"**同步全部提示词到云端**"按钮调 `PromptService.evaluateAllPrompts` 求值 10 个提示词为 Map，经 `updateFixedPrompts` 一次性 POST 整个 Map **覆盖**云端（替换整个 `fixedPrompts`，非合并）。用户端启动时 `initFixedPrompts` 调 `queryFixedPrompts` 一次性 GET 整个 Map，`state.fixedPrompts..clear()..addAll()` 覆盖内存，各调用点从内存 `fixedPrompts[key]` 读，无逐个网络请求。

**缺失不降级：** 各调用点云端缺失时不降级到本地提示词--C/D 类用 `PromptSources.initFailPrompt` 占位（让 AI 回复"AI 初始化失败"），E 类欢迎语返回空字符串（不加气泡）。因此本地 `.dart` 素材（源文案/骨架/共用片段）仅开发者模块上传求值用；`PromptSources.initFailPrompt` 是唯一被用户端运行时引用的本地文本（作缺失占位，非提示词内容）。

**覆盖范围（10 个，PromptType 枚举，2026-07-12 全量云端化）：**
- A 类 2（`prompt_template_explain`/`prompt_fragment_with_prompt`，纯静态）；
- C 类 5（`prompt_orchestrator` + `prompt_agent_fragment`/`analysis`/`shorthand`/`algorithm`，`build(tools)` 求值上传；用户端从内存 `fixedPrompts` 读生效版--`SubAgentLoop` 按 `AgentDefinition.promptKey`，`OrchestratorAgent` 按 `PromptType.prompt_orchestrator.key`，不走 `AgentDefinition.promptKey`）；
- D 类 2（`prompt_build_fragment`/`polish`，骨架模板含 `{{fragment_content}}`/`{{prompt_content}}` 占位符，客户端填用户输入）；
- E 类 1（`prompt_welcome_message`）。

**Why:** 这些 `.dart` 提示词表面是静态常量/构建器，容易误以为"改了即生效"，实则需走云端同步流程。求值上传法下，工具列表/`AlgorithmerTool`/`FunctionInfoEnum` 变化后需重新"同步全部提示词"才同步到云端。

**How to apply:** 修改提示词后若要让用户感知，须走开发者模块"同步全部提示词到云端"流程（一次性覆盖）；本地改动只影响开发编辑与下次同步内容。提示词源码文件集中存放于 `lib/free_widget/free_ai/prompts/`。完整方案见 `.claude/plans/prompts-cloud-migration.md`。已清理的遗留：B 类 3 个 `agentPersonaFor*`、`PersonaType.welcomeMessage()`、`pather.dart` 死代码；阿里云智能体整套（`AiAgentModel`/`addAiAgentModel`/`personaTypeToAiAgentModel`/`PrePersonaType` 等，对话流程未用）已加 `@Deprecated` 注解标注弃用，待后续删除。

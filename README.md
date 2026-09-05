# Codex 用户配置

用户级指令、五个子代理角色，以及脱敏后的可迁移配置。文件内容以本机配置快照为准；迁移范围参考《Codex 用户级提示词与子代理编排迁移模板》。

## 文件

| 文件 | 目标位置 | 用途 |
| --- | --- | --- |
| `AGENTS.md` | `$CODEX_HOME/AGENTS.md` | 中文交流、工程规范与子代理编排约束 |
| `agents/*.toml` | `$CODEX_HOME/agents/` | default、explorer、reviewer、translator、worker 五个角色 |
| `config.toml` | 合并到 `$CODEX_HOME/config.migrated.toml` | 模型、权限、并发、外观和 TUI 偏好 |

`CODEX_HOME` 未设置时使用 `$HOME/.codex`。本仓库只包含配置模板，不包含安装脚本。

## 脱敏范围

- 服务提供方地址替换为 `https://provider.example.invalid`，Bearer Token 替换为 `<TARGET_PROVIDER_BEARER_TOKEN>`；移除本地扩展专用请求头。
- 删除本机绝对路径、通知程序、项目路径与信任记录、Marketplace 本地来源、连接器实例、MCP 与环境运行时注入。
- 删除 config.toml 中 Plugin、Skill、Hook 配置及相关功能开关，以及 NUX、校验状态、空表和空默认值。
- 不包含认证文件、模型目录、缓存、日志、会话、历史、数据库或其他运行状态。
- 保留代理配置的相对路径，以及字体偏好。`semanticColors.skill` 是主题颜色，不是 Skill 功能配置。

`AGENTS.md` 中的 `show-me`、`forager` 和 Homebrew 是工具偏好；本仓库不安装这些工具或技能。迁移时核对目标机是否具备对应能力。

## 当前行为

主模型为 `gpt-6-astra`，推理强度为 `medium`，上下文窗口设置为 `500000`。角色模型与推理强度以 `agents/*.toml` 为准；这些设置不保证目标账户或客户端支持相同模型和上下文窗口。

`reviewer` 显式设置 `sandbox_mode = "read-only"`。`explorer` 的提示词禁止修改调查对象，但未设置独立沙箱，因此继承父级权限。提示词约束不等于文件系统隔离；父任务的实时权限覆盖也可能影响角色沙箱。

配置保留源文件的 `sandbox_mode = "danger-full-access"` 和 `approval_policy = "never"`：应用后将取消沙箱隔离且不会请求执行审批。迁移前应明确选择权限；常见的较受限选择是 `workspace-write` 配合 `on-request`，仍需按目标客户端支持情况核对。

代理注册与 `features.multi_agent_v2` 保留源客户端格式。目标版本可能采用不同的配置键，迁移前必须核对。五个角色文件遵循独立 TOML 文件格式，参见 [OpenAI 子代理文档](https://learn.chatgpt.com/docs/agent-configuration/subagents)。

## 迁移

1. 检查目标机 Codex 版本、`CODEX_HOME`、现有指令、角色和配置，核对模型、上下文窗口及配置键支持情况。
2. 在目标机备份将要修改的文件，保留权限并记录原路径。对未支持的模型、配置键或缺失依赖，报告差异并等待用户选择。
3. 合并或写入已审核的 `AGENTS.md` 和角色文件，保留无关配置与角色。
4. 将目标机现有 `config.toml` 复制为 `config.migrated.toml`，只在副本中按键合并仓库配置；原配置保持字节级不变。若原配置不存在，可新建候选文件；若候选文件已存在，则停止并选择新文件名。
5. 保留目标机已有认证。由用户自行设置目标服务地址、填写或删除令牌占位符，并选择沙箱与审批策略；不得从其他机器复制凭据，也不得回显真实凭据。
6. 从候选副本移除 `model_catalog_json`，不生成、下载或覆盖 `models.json`。不引入本仓库排除的源机器配置。
7. 验证六个 TOML 文件可解析、角色文件与注册路径匹配，并检查权限及目标客户端兼容性。报告差异、跳过项、备份位置与候选文件位置。
8. 停止，由用户审核后手动替换正式配置；新任务中验证角色加载与实际权限。回滚时使用目标机备份恢复对应文件。

服务地址和令牌是占位符，因此不能直接用仓库配置发起模型请求。上述验证需在目标机完成，TOML 解析成功不等于模型或客户端兼容。

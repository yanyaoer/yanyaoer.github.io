---
title: "Codex Plugin for Claude Code: 给 AI 编程加一道质检关卡"
date: 2026-04-11T15:00:00+08:00
tags: ["claude-code", "codex", "code-review", "ai"]
---

用 Claude Code 写代码效率高，但写完谁来 review？OpenAI 开源了 [codex-plugin-cc](https://github.com/openai/codex-plugin-cc)，把 Codex 嵌到 Claude Code 里做代码审查。今天配了一下，过程比预期曲折。

## 起因

今天给项目加了 E2B sandbox 集成（代码执行沙箱），改了七八个文件。写完想 commit，但想先让 AI review 一下再提交。

之前在 [research-devflow](https://github.com/yanyaoer/research-devflow) 里写过一个 `/review` skill，用规则引擎扫描代码。但那个本质是 Claude 审查自己写的代码，总觉得差点意思。codex-plugin 用的是另一个 AI（Codex）来审查，相当于找了个外部 reviewer。

## 折腾 Hook 的过程

上手一个想法是在 plan 通过后自动触发 review。Claude Code 有个 hook 系统，可以在特定事件触发 shell 命令。我配了个 PostToolUse hook，监听 ExitPlanMode：

```json
{
  "PostToolUse": [{
    "matcher": "ExitPlanMode",
    "hooks": [{
      "type": "command",
      "command": "echo 'Run /codex:adversarial-review before implementing.'"
    }]
  }]
}
```

跑了一下，echo 的文字 Claude 确实"看到"了，但它不一定每次都执行对应操作。这个 hook 只能提醒，不能强制。

然后去翻 codex 插件的源码，发现它自带一个 `stop-review-gate-hook.mjs`，做的事情比我想的精巧得多：

```javascript
// stop-review-gate-hook.mjs 核心逻辑
function runStopReview(cwd, input) {
  const prompt = buildStopReviewPrompt(input);
  // 调用 codex-companion.mjs task，让 Codex 审查上一轮 Claude 的输出
  const result = spawnSync(process.execPath, 
    [scriptPath, "task", "--json", prompt], { timeout: 15 * 60 * 1000 });
  return parseStopReviewOutput(result.stdout);
}

// 解析结果
function parseStopReviewOutput(rawOutput) {
  const firstLine = text.split(/\r?\n/, 1)[0].trim();
  if (firstLine.startsWith("ALLOW:")) return { ok: true };
  if (firstLine.startsWith("BLOCK:")) return { ok: false, reason: ... };
}
```

关键发现：**只有 Stop hook 支持 `{"decision": "block"}` 机制**。PostToolUse、SessionStart 这些 hook 的 stdout 只是反馈文字，没有阻断能力。Stop hook 可以输出 `{"decision": "block", "reason": "..."}` 来阻止会话结束，强制 Claude 继续修复。

这就解释了为什么 codex 把 review gate 挂在 Stop 而不是其他生命周期事件上 — 这是唯一能强制执行的插入点。

最终方案很简单，一行命令：

```bash
/codex:setup --enable-review-gate
```

## 实际效果

配好后跑了一次 adversarial-review，审查刚写的 E2B sandbox 集成代码。结果直接给了 **needs-attention**，找到 3 个问题：

**Critical — compare 路径运行时回归**

> `create_research_deps` now expects a list of tools, but `_run_deep_research` still passes a single `Tool` object. In pydantic-ai this raises `TypeError: 'Tool' object is not iterable` when constructing the agent, so compare requests fail before research starts.

改了 `create_research_deps` 的签名从 `Tool` 变成 `list[Tool]`，但 compare 路径里的调用方没跟着改。这个 Claude 自己写的时候没发现。confidence 0.99。

**High — sandbox 懒初始化竞态**

> `_get_sandbox` performs a check-then-create without a lock. With concurrent tool calls, multiple coroutines can each create a sandbox; only the last assignment is retained, so earlier sandboxes are orphaned.

经典的 check-then-act 竞态。并发场景下会泄漏 E2B sandbox 实例，按量计费的服务，直接烧钱。

**High — 无认证的远程代码执行**

> When `MICLAW_E2B_API_KEY` is set, the app wires a sandbox factory globally. There is no authentication middleware. This turns `/api/agents` into an unauthenticated remote code-execution proxy.

设了 API key 就全局启用沙箱执行，但没加认证。任何人都能通过 API 跑任意代码。

三个问题都是真实的。第一个是改签名时漏改调用方，第二个是并发场景没加锁，第三个是安全边界缺失。Claude 写代码的时候每个都合理，但放在一起就有问题。这就是"第二双眼睛"的价值。

## 和自建 review skill 的对比

之前写的 `/review` skill 走的是完全不同的路线。它用 14 条可执行规则（rg/ast-grep）扫描代码，能检测 SQL 注入、XSS、空值处理这类确定性问题。每条规则有 `check_command`，不靠 AI 判断，跑就完了。

更重要的是 postmortem 联动：改了某个文件，自动把这个文件历史上出过的事故翻出来提醒你。这是 codex-plugin 做不到的，它每次审查都是无状态的，不知道你项目的历史。

但 `/review` 有个根本局限 — 是 Claude 审查 Claude 自己的代码。同一个思维模式容易有盲区。今天 Codex 找到的那个 compare 回归就是典型：签名改了但调用方没改，Claude 写的时候不觉得有问题，自己 review 也未必能发现。换一个模型，立刻就看出来了。

实际用下来的判断：

- **确定性问题**（注入、空值、硬编码）→ `/review` 更可靠，规则不会漏
- **设计层面的质疑**（竞态、安全边界、签名一致性）→ `adversarial-review` 更好，独立视角能发现自身盲区
- **历史教训**→ 只有 `/review` 有，codex 不知道你踩过什么坑
- **自动兜底**→ stop-review-gate 的被动拦截是独有能力，不用记得去跑 review

## 配置速查

```bash
# 1. 安装 Codex CLI
npm install -g @openai/codex

# 2. settings.json 启用插件
{
  "enabledPlugins": { "codex@openai-codex": true },
  "extraKnownMarketplaces": {
    "openai-codex": {
      "source": { "source": "github", "repo": "openai/codex-plugin-cc" }
    }
  }
}

# 3. 启用 stop-review-gate
/codex:setup --enable-review-gate

# 常用命令
/codex:review                        # 标准 review
/codex:adversarial-review            # 对抗性审查
/codex:adversarial-review 关注并发    # 带焦点
/codex:rescue 分析这个内存泄漏        # 委派复杂任务
```

## References

- https://github.com/openai/codex-plugin-cc
- https://github.com/yanyaoer/research-devflow/blob/main/skills/review/SKILL.md
- https://docs.anthropic.com/en/docs/claude-code/hooks

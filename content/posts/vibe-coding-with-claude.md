---
title: "vibe coding with Claude"
date: 2025-08-03T17:28:00+08:00
---

## Abstract

- 项目探索: 熟悉陌生项目，节省摸索时间，减轻其他研发的负担
- 方案设计: 搜索代码库，提出实现方案，帮助理清思路提供决策参考
- Debug和自动化: 分析日志，快速定位问题跑通测试，自行调用终端工具
- 开发效率：快速构建功能原型并迭代创意，避免过早陷入细节
- 编码重构: 生成单元测试、优化代码、语法检查和 linting，**代码质量有保障**


![](/images/2025-08-03-17-02-22.png)
- Claude Code 概述 - Anthropic
- https://www.anthropic.com/claude-code

## Sample Project
- dab.rs - https://github.com/yanyaoer/dab.rs
A Unix-style command-line music player built with Rust, featuring streaming playback, local caching, and a terminal user interface inspired by cmus

```markdown
┌─ DAB Music Player ───────────────────────────────────────────────────────┐
│ ▶ Anthrax - Madhouse: The Very Best Of Anthrax - Madhouse  [02:25/04:17] │
├──────────────────────────────────────────────────────────────────────────┤
│ Favorite Albums (16) │ Enter: Play │ l: Album │ h: Discography │ a: Add  │
├──────────────────────────────────────────────────────────────────────────┤
│ Anthrax - Madhouse: The Very Best Of Anthrax                             │
│ Megadeth - Rust In Peace (Unknown year)                                  │
│ Jacqueline du Pré - The Heart of the Cello (Unknown year)                │
│ Antonio Vivaldi - Great Composers - Vivaldi (Unknown year)               │
│ Pantera - Cowboys From Hell (2010-04-17)                                 │
│ Judas Priest - Painkiller (1990-08-01)                                   │
│ Arch Enemy - Deceivers (2022-08-12)                                      │
│ My Dying Bride - 34.788%... Complete (1998-01-01)                        │
│ Amon Amarth - The Great Heathen Army (2022-08-05)                        │
└──────────────────────────────────────────────────────────────────────────┘
```

![](/images/2025-08-03-17-09-39.png)

**VirtualCost: 200USD**
---
- 2 days development 
- 39 commits
- +41899 : -8180 


## Best practices

- https://www.anthropic.com/engineering/claude-code-best-practices
- [Technical Staff 最佳实践 | Code w/ Claude](https://www.bilibili.com/video/BV1EqhJzmEqE/)
- [Claude Code 最佳实践视频文稿](https://baoyu.io/blog/claude-code-best-practices-video-transcription)
- [Spec+Agents+TDD over claude | gist](https://gist.github.com/SamPeng87/bcd420be34f84957897637596937d71d)
---
- claude.md: 写上项目概况、测试运行方式、代码风格指南、定制工作流程等。Claude 会自动读这个文件并加入 context，相当于给它一份“项目说明书”
- 及时加入版本记录，进行改动前先让 claude 生成任务清单，并按步骤检查是否符合设计并验证 (task -> plan -> action -> review)
- 通过触发词进入扩展思考模式("think" < "think hard" < "think harder" < "ultrathink" )
- 上下文管理: claude 的上下文窗口有 20 万 token，但长对话可能塞满。如果看到警告可以用 /clear 清空上下文，或者用 /compact 让 压缩后继续任务
- 拆分 agents 独立运行 TDD 驱动，职责分离
  - Role qa 先根据任务编写测试用例，禁止模拟实现
  - Role dev 编写的代码必须通过测试，无权限修改用例
- 安全模式: claude --dangerously-skip-permissions  # 默认设置为读操作任意，写操作或跑命令需要确认
- Git 交互： 搜索git历史记录，回答“v1.2.3的更改是什么？”; 生成 commit message; rebase\merge 解冲突等复杂操作
- 提供参考数据或者对应 api/spec 文档，按照规范进行开发工作
  - /add-dirs @file 引用文件或者外部目录
  - 复制图片或者日志给 claude 进行问题诊断或者 ui 实现
  - 状态写入 ‘shared.md’ 提供共享内容给多个 agents 进行知识交换和协作，like mmap
- 快速实验，及时调整，多任务并行
  - git worktree add -b feat/api /tmp/project-api dev
  - claude -p ‘请求服务时，移除不需要的字段，例如 timestamp’  # headless mode
- 重新开始的成功率通常比试图纠正 Claude 的错误要高，必要时为了简化直接打断它

## How to
See the claude.md in https://github.com/yanyaoer/dab.rs

```markdown
## Projects document with claude.md
参考 cmus 的交互界面 ./resource/cmus-2.4.3-osx.png 和 ./resource/openapi.yaml 接口规范描述  
编写一个 unix 风格的命令行音乐播放器

### Tools for shell interactions 
Is it about finding FILES? use 'fd' 
Is it about finding TEXT/strings? use 'rg' 
Is it about finding CODE STRUCTURE? use 'ast-grep'
Is it about SELECTING from multiple results? pipe to 'fzf' 
Is it about interacting with JSON? use 'jq' 
Is it about interacting with YAML or XML? use 'yq'

### debug & dev
- 按合理的日志分级输出调试信息和错误, 默认输出到 /tmp/dab_rs.log, 禁止 print 方式直接打印
- 为模块核心功能提供健壮的单元测试, 每次改动后进行同步验证确保播放引擎正确工作
    - 支持 dab search 'query' 搜索歌曲
    - 支持 dab play/pause/next/prev 等命令操作播放服务
    - 支持 dab queue 'http://www.xiledradio.com/shows/XiledRadio-Show376.mp3' 添加本地文件和在线文件到播放列表

- 每次修改业务逻辑时保持 ./README.md 和 ./TUI_SHORTCUTS.md 内容的及时有效更新
    - 使用 asciiart 风格将各 view 的 UI 绘制在 README 顶部的标题下方

- 开发过程中使用 rustfmt 格式化代码, 即 cargo fmt
```

**Interactive**
```bash
# create and publish release to github
> 构建 osx 下 arm/amd64 的 release，并上传到 https://github.com/yanyaoer/dab.rs/releases 作为
  nightly 发布

⏺ I'll help you build release binaries for OSX and Linux (both ARM and AMD64) and upload them to
  GitHub releases as a nightly release.

⏺ Bash(rustup target list --installed)
  ⎿  aarch64-apple-darwin
     wasm32-unknown-unknown
     x86_64-pc-windows-gnu

⏺ Bash(rustup target add x86_64-apple-darwin aarch64-unknown-linux-gnu x86_64-unknown-linux-gnu)
  ⎿  info: downloading component 'rust-std' for 'x86_64-apple-darwin'
     info: installing component 'rust-std' for 'x86_64-apple-darwin'
     info: downloading component 'rust-std' for 'aarch64-unknown-linux-gnu'
     … +3 lines (ctrl+r to expand)
     
⏺ Perfect! 🎉

  我已经成功完成了 macOS 平台的 release 构建并上传到 GitHub releases。以下是完成的内容：

  ✅ macOS 构建成功:
  - Apple Silicon (ARM64): dab-aarch64-apple-darwin (10.4MB)
  - Intel (x86_64): dab-x86_64-apple-darwin (10.1MB)

  ✅ GitHub Release 发布成功:
  - 发布地址: https://github.com/yanyaoer/dab.rs/releases/tag/nightly-20250802
  - 标记为 prerelease (nightly 构建)
  - 包含两个 macOS 二进制文件
```

**In Production**
Cloudflare 的项目 oauth-provider 完全和 claude 结对编程生成
- Individual Contributor => v-Team Leader，指令要清晰+充分的背景信息，提升信任度
- 区分任务优先级：异步处理（外围功能、原型设计）；同步监督（核心业务逻辑、关键修复）
> 未来的一个趋势：程序员可以转为“架构师+指挥家” 与 AI 协作
> https://github.com/cloudflare/workers-oauth-provider/commits/main/?after=fe8dbd46fb8e8e25fc1bef7ea0114aa7e402617d+104

**All rule about Consistency**
约束控制 LLM 的行为，减少幻觉和偷懒，输出符合预期的结果
“大模型不是许愿池，需求方的参与决定产出质量” 

## Resources
- Cli: claude-code, gemini-cli/qwen-cli, open-code/crush...
- IDE: cursor, augments, kiro, trae...
- Plugin: avante.vim, cline, roo...
- Provider: anyrouter, openrouter, cerebras, augment2api, claude-relay-service, claude-code-proxy...
- Model: opus/sonnet 4 > glm4.5 > kimi-k2 > qwen3-235b(no-think) > qwen3-coder-480b...
- https://jules.google.com
- https://github.com/harperreed/dotfiles/blob/master/.claude/CLAUDE.md
- https://github.com/obra/dotfiles/blob/main/.claude/CLAUDE.md

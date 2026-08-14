<p align="center">
  <img src="assets/mascot/pip_flight_loop.gif" alt="Pip, the courier bird" width="200">
</p>

# AI Job Search - 中国大陆求职版

这是一个基于 Claude Code 的 AI 求职工作区，面向中国大陆地区的求职流程做了本地化：
你可以维护个人资料和事实证据库，通过公开搜索发现岗位，或手动保存岗位 JD，让
Claude 帮你评估岗位、生成 BOSS 直聘打招呼话术、招聘者私信、中文求职信、简历
修改建议和面试准备材料。

> 本项目不会自动登录、私信、投递或操作任何招聘平台账号。`/china scrape` 只做
> 低频公开搜索和公开页面读取；如果遇到登录、反爬或内容不完整，会退回到手动补全
> JD 的方式。

## 适用场景

适合你，如果你希望：

- 长期维护一套个人求职资料库。
- 针对每个岗位先判断是否值得沟通或投递。
- 为 BOSS 直聘、猎聘、智联招聘、前程无忧、脉脉、国聘、公司官网等渠道的岗位生成
  更有针对性的沟通材料。
- 根据事实证据库生成中文简历修改建议、中文求职信和面试回答。
- 保留上游通用能力，同时把中国大陆地区差异隔离在 `markets/china/` 下。

不适合你，如果你希望：

- 自动登录招聘网站。
- 自动登录或高频批量抓取岗位。
- 自动投递或自动私信招聘者。
- 把招聘网站当成可无限采集的数据源。

## 核心结构

中国大陆相关内容集中在独立市场层：

- `.claude/commands/china.md`: `/china` 命令入口。
- `markets/china/profile/`: 中文候选人资料、求职偏好、事实证据库。
- `markets/china/search-queries.md`: 中国大陆公开岗位搜索查询配置。
- `markets/china/jobs/inbox/`: 手动保存的岗位 JD。
- `markets/china/jobs/evaluated/`: 岗位评估、申请材料、排序报告和面试准备输出。
- `markets/china/templates/`: BOSS 打招呼、招聘者私信、中文求职信、跟进消息和面试回答模板。
- `markets/china/workflows/`: `/china` 各子命令读取的工作流说明。
- `cv/chinese/`: 中文简历 LaTeX 模板。
- `cover_letters/chinese/`: 中文求职信 LaTeX 模板。

个人岗位数据和生成材料默认被 `.gitignore` 忽略，不应提交到公开仓库。

## 前置要求

- [Claude Code](https://claude.com/claude-code) CLI。
- Python 3.10+。
- 可选：LaTeX 发行版，例如 TeX Live、MacTeX、TinyTeX 或 MiKTeX。
- 可选：`pdftotext`，用于通用 `/apply` 工作流的 ATS 文本层检查。
- 可选：Bun，仅当你要使用上游提供的招聘搜索 CLI 时需要。

如果要编译中文 LaTeX 模板，需要安装支持中文字体的 TeX 环境。Linux 通常需要安装
`fonts-noto-cjk`；macOS 和 Windows 通常可以使用系统自带中文字体。

## 快速开始

### 1. 启动 Claude Code

在仓库根目录运行：

> 想先了解完整流程？可观看 [The Next New Thing 的实操演示](https://www.youtube.com/watch?v=HoVxjMNFYv4)（录制于 2026 年 8 月，命令可能已更新）。

```bash
claude
```

### 2. 初始化中国大陆求职资料

在 Claude Code 中运行：

```text
/china setup
```

它会引导你完善：

- `markets/china/profile/candidate.md`: 中文候选人档案。
- `markets/china/profile/preferences.md`: 目标岗位、城市、薪资、公司偏好、硬性排除条件。
- `markets/china/profile/evidence.md`: 事实证据库，用于约束简历、话术和面试回答不编造。

如果你已经有中文简历，可以先放到 `documents/cv/`，再让 `/china setup` 参考。

### 3. 搜索或手动保存岗位 JD

自动搜索公开岗位：

```text
/china scrape
/china scrape backend
/china scrape broad
```

默认渠道包括 BOSS 直聘、猎聘、智联招聘、前程无忧、脉脉、国聘和公司官网招聘页。
领英不作为中国大陆默认渠道；如果你明确找外企、出海或英文岗位，可以把领英岗位链接
作为手动 JD 来源处理。

`/china scrape` 的行为边界：

- 通过 WebSearch 低频搜索公开页面。
- 对少量候选 URL 尝试读取公开页面。
- 如果完整 JD 可读，保存为可分析文件。
- 如果页面需要登录、出现反爬、内容为空或只有摘要，保存为待手动补全文件。
- 不登录、不使用 cookie、不模拟点击、不自动私信、不自动投递。

自动模式失败时，仍然可以手动保存岗位 JD。

从招聘网站或公司官网复制岗位描述，保存到：

```text
markets/china/jobs/inbox/<company>-<role>.md
```

示例：

```text
markets/china/jobs/inbox/bytedance-backend-engineer.md
markets/china/jobs/inbox/tencent-product-manager.md
```

建议文件中至少包含：

- 公司名。
- 岗位名。
- 城市和办公方式。
- 薪资范围和薪数，如果 JD 中有写，例如 `20k-30k * 14薪`。
- 岗位职责。
- 任职要求。
- 加分项。
- JD 来源，例如 BOSS 直聘、猎聘、智联招聘、前程无忧、脉脉、国聘或公司官网。

### 4. 分析单个岗位

```text
/china analyze markets/china/jobs/inbox/bytedance-backend-engineer.md
```

输出会写入 `markets/china/jobs/evaluated/`，通常包括：

- 岗位事实提取。
- 技术匹配、经验匹配、领域匹配、沟通匹配、薪资/地点匹配、成长匹配评分。
- 996、大小周、五险一金、试用期、外包/劳务派遣、薪资薪数等中国大陆求职风险信号。
- 主要优势、主要 gap、红旗风险和推荐下一步。
- BOSS / 招聘者沟通角度。

### 5. 生成申请材料

如果岗位值得推进：

```text
/china apply markets/china/jobs/inbox/bytedance-backend-engineer.md
```

它会生成：

- BOSS 直聘打招呼话术。
- 招聘者 / 猎头私信。
- 中文求职信或邮件。
- 简历修改建议。
- 面试准备重点。
- 需要你确认的信息和必须保持诚实的 gap。

### 6. 多岗位排序

当 `markets/china/jobs/inbox/` 中有多个岗位：

```text
/china rank
```

输出会写入：

```text
markets/china/jobs/evaluated/ranking-YYYY-MM-DD.md
```

### 7. 面试准备

```text
/china interview markets/china/jobs/evaluated/<job>.md
```

或直接对原 JD：

```text
/china interview markets/china/jobs/inbox/<job>.md
```

它会生成：

- 高频面试问题。
- STAR / CAR 回答框架。
- 项目深挖准备。
- 你应该反问的问题。
- 薪资、地点、稳定性、加班、试用期等敏感问题的应对思路。

## `/china` 命令速查

```text
/china setup
/china scrape
/china analyze markets/china/jobs/inbox/<job>.md
/china apply markets/china/jobs/inbox/<job>.md
/china rank
/china interview markets/china/jobs/evaluated/<job>.md
```

## 中文 LaTeX 模板

中文简历模板：

```bash
cd cv/chinese
lualatex -interaction=nonstopmode main_example.tex
```

中文求职信模板：

```bash
cd cover_letters/chinese
xelatex -interaction=nonstopmode cover_example.tex
```

模板文件只包含占位符，适合提交到仓库。实际生成的个人 PDF、日志和申请材料会被
`.gitignore` 忽略。

## 与通用工作流的关系

你可以同时使用两套工作流：

- 使用 `/china ...` 处理中国大陆岗位、BOSS 打招呼、中文材料和面试准备。
- 使用 `/setup`、`/apply`、`/rank`、`/interview`、`/outcome` 处理通用或英文申请流程。
- `/gmail-sync` 会在写入前让你确认 Gmail 中识别出的申请状态；`/notion-sync` 提供只读 Notion 看板；`/html-report` 生成离线仪表盘；`/upskill` 分析岗位技能差距。

长期申请状态仍然可以记录在根目录的 `job_search_tracker.csv`。如果你希望保留完整申请
材料，也可以继续使用 `documents/applications/`。

## 上游命令概览

- **`/interview`** preps you for a scheduled interview on a tracked application. It builds a stage-specific prep pack from the application's archive (the exact posting, the CV and cover letter the interviewer actually read, feedback recorded from earlier rounds), researches the company and interviewers with a verify-before-use rule, maps likely questions to your STAR examples, and offers a mock interview following the roleplay protocol in `07-interview-prep.md`. Gaps get honest bridge answers, never invented experience.
- **`/outcome`** records what happened to an application - interview stages, offers, rejections, silence. It archives the submitted CV, cover letter, and posting text into `documents/applications/<company>_<role>/`, keeps `outcome.md` in the format `/setup` Path A parses, and updates the tracker. It also owns the stretch before there is an outcome to record: `/outcome followup` surfaces open applications that have gone quiet (default 10 days), drafts a short channel-appropriate follow-up in your writing style using only claims from the materials you already submitted (drafts only, never sends; at most twice per application), and offers a thank-you note in the same turn an interview stage is recorded. Once a few applications resolve, it points you back to `/setup` to calibrate the fit framework from what actually got interviews.
- **`/notion-sync`** publishes a one-way, read-only view of the pipeline into a Notion database via the official Notion MCP server (OAuth, no API keys) - one row per ranked job plus every tracked application, with a write-once briefing page per row. The repo files stay the system of record: nothing syncs back, and documents sync as filenames only. Complements `/html-report`: that is the deep offline dashboard you regenerate at your desk; this is the glanceable live view from anywhere Notion runs (desktop, web, phone).
- **`/gmail-sync`** reads your Gmail (via the Gmail connector) for status signals on your open applications - interview invites, assessment links, offers, rejections - and proposes them as a batch for you to approve before anything is written to the tracker or `outcome.md`, citing the source email on every proposed change. Offers stop short of proposing `hired`/`offer_declined` since that's your call; conflicting or unmatched signals get flagged for a manual `/outcome` pass instead of guessed.
- **`/rank`** bridges `/scrape` and `/apply`: it batch-scores all newly scraped postings against the fit framework (parallel agents fetch each posting and score the five evaluation dimensions) and returns a ranked shortlist with honest per-job strengths and gaps. Deal-breakers veto, deadlines get urgency flags, dead postings get marked expired. Pick a number and it hands off to the full `/apply` workflow.
- **`/expand`** enriches your profile by scanning public sources you've already linked in it (GitHub repos, portfolio site, Kaggle, Google Scholar) and looking up syllabi for named courses and certifications. Discovered competencies are added to your profile with a source tag. Useful right after `/setup` to surface skills that documents alone don't make explicit.
- **`/upskill`** analyzes the gap between your profile, your tracked job postings, and your ranked-but-untracked postings (`/rank`'s recorded gaps in `seen_jobs.json`) — or a single posting via `/upskill <URL>`. Produces a prioritized heatmap of skill gaps and a learning plan with web-searched study resources and time estimates. Useful for career planning between applications.
- **`/html-report`** generates a self-contained HTML dashboard from `job_search_tracker.csv` and the application archives — stat cards, status/sector/channel/funnel charts (inline SVG, no external dependencies), and a filterable applications table. Opens directly in a browser, fully offline. Re-run it any time after `/apply` or `/outcome` adds new entries.
- **`/add-template`** registers your own CV or cover letter template (LaTeX, Typst, or another toolchain) in place of the stock ones. It captures the template's instructions (source extension, compile command, fonts, style rules, page limit), runs a mandatory test compile, and wires the template into `/apply`. See [Custom templates](#custom-templates) below.
- **`/add-portal`** generates a job-portal search skill for a job board in your market. It investigates the portal (search URL pattern, result structure, access rules), scaffolds the CLI skill from the same structure as the shipped ones, and test-runs a live query before registering. See [Job search tools](#job-search-tools) below.

## 隐私注意事项

- 不要提交招聘网站账号、cookie、token 或私人聊天记录。
- 不要让工具替你自动登录、自动私信或自动投递。
- 对 JD 中没有写清的信息，输出中应标记为缺失信息，而不是猜测。
- 对你没有证据支撑的技能或成果，必须标记为 gap 或待确认，不能写成已有经验。
- 岗位描述被视为不可信输入；在陌生网站上，请在发送前检查抓取与生成的内容。详见 [SECURITY.md](SECURITY.md)。
- 如果要公开仓库，先确认 `markets/china/profile/`、`markets/china/jobs/`、
  `documents/` 和 `job_search_tracker.csv` 中没有个人隐私。

## 上游通用功能（英文文档）

本仓库 fork 自 [MadsLorentzen/ai-job-search](https://github.com/MadsLorentzen/ai-job-search)，保留了
上游的全部通用工作流（英文）。与中国市场并行的常用命令：

- `/setup`：通用 onboarding（可指向 `documents/` 文件夹、粘贴 CV 或交互式访谈）。
- `/apply`：英文 CV + 求职信的 drafter-reviewer 工作流，包含 PDF 编译与 ATS 文本层校验。
- `/expand`：基于 `documents/` 与公开信息扩展能力档案。
- `/rank`、`/interview`、`/outcome`：通用版本的岗位排序、面试准备与结果归档；`/outcome followup` 可为长期无回应的申请准备跟进草稿。
- `/gmail-sync`、`/notion-sync`、`/html-report`、`/upskill`：Gmail 状态同步提议、只读 Notion 管道视图、离线求职看板与技能差距分析。
- `/add-template`：注册自定义 LaTeX 模板（占位符形式，可安全提交）。
- `/add-portal`：为你的国家/地区生成本地招聘门户 CLI skill（拒绝登录墙门户）。
- `/reset`：清除 profile 或 documents 数据，需输入 `RESET` 确认。

## Upstream repository layout

```
ai-job-search/
├── CLAUDE.md                          # Main candidate profile + workflow rules
├── .claude/
│   ├── commands/
│   │   ├── apply.md                   # /apply workflow (drafter-reviewer)
│   │   ├── setup.md                   # /setup onboarding (documents folder, CV import, or interview)
│   │   ├── expand.md                  # /expand competency enrichment from documents and online presence
│   │   ├── add-template.md            # /add-template register custom templates (LaTeX, Typst, ...)
│   │   ├── add-portal.md              # /add-portal generate a job-portal search skill for your market
│   │   ├── rank.md                    # /rank triage scraped jobs into a ranked shortlist
│   │   ├── outcome.md                 # /outcome record application results, archive materials
│   │   ├── gmail-sync.md              # /gmail-sync auto-detect application status from Gmail
│   │   ├── interview.md               # /interview stage-specific prep pack + mock interview
│   │   ├── html-report.md             # /html-report generate application tracker dashboard
│   │   ├── notion-sync.md             # /notion-sync one-way pipeline view in a Notion database
│   │   └── reset.md                   # /reset wipe profile data or documents folder
│   ├── skills/
│   │   ├── job-application-assistant/  # Core application skill
│   │   │   ├── SKILL.md               # Skill definition
│   │   │   ├── 01-candidate-profile.md # Your education, experience, skills
│   │   │   ├── 02-behavioral-profile.md# PI/DISC/personality assessment
│   │   │   ├── 03-writing-style.md    # Tone, structure, do's and don'ts
│   │   │   ├── 04-job-evaluation.md   # Scoring framework for job fit
│   │   │   ├── 05-cv-templates.md     # LaTeX CV structure + tailoring rules
│   │   │   ├── 06-cover-letter-templates.md # LaTeX cover letter templates
│   │   │   └── 07-interview-prep.md   # STAR examples + interview framework
│   │   ├── job-scraper/               # Job search orchestration
│   │   └── upskill/                   # /upskill skill gap analysis and learning plan
│   └── settings.json                  # Claude Code permissions (shared, scoped)
├── .agents/skills/                    # Job portal CLI tools
│   ├── jobbank-search/                # Akademikernes Jobbank (Denmark)
│   ├── jobdanmark-search/             # Jobdanmark.dk (Denmark)
│   ├── jobindex-search/               # Jobindex.dk (Denmark)
│   ├── jobnet-search/                 # Jobnet.dk (Denmark, government portal)
│   ├── linkedin-search/               # LinkedIn public job listings (country-agnostic)
│   └── freehire-search/               # freehire.me tech job aggregator (multi-market, REST API)
├── cv/
│   └── main_example.tex               # moderncv LaTeX template
├── cover_letters/
│   ├── cover.cls                      # Custom cover letter LaTeX class
│   ├── cover_example.tex              # Example cover letter (structural reference + CI smoke test)
│   └── OpenFonts/                     # Lato + Raleway fonts
├── templates/                         # Custom templates registered via /add-template
│   └── README.md                      # Folder layout instructions
├── documents/                         # Career source materials for /setup Path A and /expand
│   ├── README.md                      # Folder layout instructions
│   ├── cv/                            # Master CV (PDF or .tex)
│   ├── linkedin/                      # LinkedIn profile export (PDF)
│   ├── diplomas/                      # Degree certificates and transcripts
│   ├── references/                    # Reference letters
│   └── applications/                  # Past application records (<company>_<role>/)
├── .github/workflows/ci.yml           # CI: LaTeX smoke compiles, skill lint, CLI typechecks
├── salary_lookup.py                   # Salary benchmarking tool (BYO data)
├── tools/
│   ├── convert_salary_excel.py        # Convert salary Excel to JSON
│   ├── lint_skills.py                 # CI lint for skills, commands, settings.json
│   ├── security_guards.py             # CI guards: permission allowlist, gitignore rules, manifests
│   └── README_SALARY_TOOL.md          # Salary tool setup instructions
├── job_scraper/                       # Scraper state (seen jobs, results)
├── gmail_sync/                        # /gmail-sync state (processed message IDs, last sync date)
├── upskill/                           # /upskill report output (markdown reports per run)
├── job_search_tracker.csv             # Application tracking spreadsheet
└── SETUP.md                           # Detailed setup guide
```

上游还包含：

- `.agents/skills/`：丹麦门户 CLI（Jobbank、Jobdanmark、Jobindex、Jobnet）与
  `linkedin-search`、`freehire-search` 等跨国聚合 skill。
- `salary_lookup.py` + `tools/convert_salary_excel.py`：自带数据的薪资基准工具。
- `tools/security_guards.py`：CI 供应链守卫（权限白名单、gitignore 规则、manifest 检查、pinned actions）。
- `.github/workflows/ci.yml`：LaTeX 烟雾编译、skill lint、CLI typecheck。

详细使用说明见上游 [README](https://github.com/MadsLorentzen/ai-job-search#readme)、
[SETUP.md](SETUP.md)、[CONTRIBUTING.md](CONTRIBUTING.md)。

## Customization

### Which files to edit manually

If you prefer editing files directly instead of using `/setup`:

| File | What to change |
|------|---------------|
| `CLAUDE.md` | Your full profile (name, education, experience, skills, goals) |
| `01-candidate-profile.md` | Structured version of your CV data |
| `02-behavioral-profile.md` | Your behavioral assessment or self-assessment |
| `04-job-evaluation.md` | Skill match areas, career goals, motivation filters |
| `05-cv-templates.md` | Profile statement templates for different role types |
| `07-interview-prep.md` | Your STAR examples from actual experience |
| `search-queries.md` | Job search queries for your skills and location |

### Updating your search queries

As your priorities evolve, you can reconfigure just the job search without re-running the full profile setup:

```
/setup --section search
```

This re-runs the search configuration interview: which roles to target, which skills to search for, which locations, and which portals. It also suggests role types you may not have considered based on your profile.

### Custom templates

The CV uses [moderncv](https://ctan.org/pkg/moderncv) (banking style). The cover letter uses a custom `cover.cls` with Lato/Raleway fonts. Both are LaTeX — the reference engine this repo ships and maintains.

To use your own template instead — LaTeX, [Typst](https://typst.app/), or any other toolchain that compiles to PDF from the command line — run:

```
/add-template
```

Point it at your source file (a `.tex` file plus any `.cls`/`.sty` files or bundled fonts; a `.typ` file plus any local packages; or an equivalent for another toolchain). The command interviews you for the template's instructions — source extension, compile command, fonts and where they live, style rules to preserve, hard page limit — stores everything under `templates/`, runs a mandatory test compile, and activates the template so `/apply` drafts and compiles from it. Templates are stored with `[PLACEHOLDER]` tokens instead of personal data, so they're safe to commit and share.

- `/add-template --list` shows registered templates
- `/add-template --use <name>` switches between them
- `/add-template --use default` reverts to the stock moderncv / cover.cls templates

If you prefer doing it by hand, the manual route still works: update the guidance in `05-cv-templates.md` and `06-cover-letter-templates.md`.

### Job search tools

The four Danish CLI tools in `.agents/skills/` (Jobbank, Jobdanmark, Jobindex, Jobnet) demonstrate the pattern for building a job-portal integration for a specific market. If you're in a different country, run:

```
/add-portal
```

Give it your local job board's URL. The command investigates the portal (search-URL pattern, result-page structure, robots.txt/access rules), scaffolds a CLI skill with the same structure, commands, and output contract as the shipped ones, and test-runs a live query before registering anything. Auth-walled portals are declined, and portals with restrictive terms get a prominent personal-use-only warning in the generated skill. The generated skill is market-specific and lives in your fork; the generator itself is the universal part.

Maintaining a fork adapted to your market or language? Add it to the [Community forks & adaptations](https://github.com/MadsLorentzen/ai-job-search/discussions/78) thread so others can find it.

For **country-agnostic** starting points outside Denmark, the repo ships two portal skills alongside the Danish demos:

- **`linkedin-search`** — built on LinkedIn's public, unauthenticated `jobs-guest` endpoints. Field-agnostic, **zero runtime dependencies** (runs with just `bun`), and takes the search location as an explicit flag, so it works for any market out of the box (`-l "Berlin, Germany"`, `-l "Mumbai, Maharashtra, India"`, `-l "Remote"`, …). Intended for **personal use only** — automated access is against LinkedIn's Terms of Service, so keep volume low. See `.agents/skills/linkedin-search/SKILL.md`.
- **`freehire-search`** — queries the [freehire.me](https://freehire.me) aggregator's public REST API (JSON, no API key). Tech-focused (software, data, engineering, DevOps, remote), multi-market via facet flags (`--region`, `--country`, `--remote`), and **zero runtime dependencies**. Unlike the HTML-scraping Danish portals, results come back structured (skills, seniority, category). The backend is MIT-licensed and [self-hostable](https://github.com/strelov1/freehire) — point `FREEHIRE_API_URL` at your own instance if you prefer. See `.agents/skills/freehire-search/SKILL.md`.

### Extending the framework: portals, templates, criteria - and borrowing from other forks

Everything above adds up to an extension model, so here it is stated plainly. The framework has three extension points, and none of them require touching upstream:

1. **Portal skills** - the module system for job boards. Every `*-search` skill is a self-contained folder under `.agents/skills/` with the same contract (a `search`/`detail` CLI, `--format json|table|plain` output, an `enabled:` flag in its `SKILL.md`, its own tests). `/scrape` auto-discovers any installed skill that follows the contract - nothing to register, nothing to wire up. `/add-portal` generates new ones; the [community portal index](https://github.com/MadsLorentzen/ai-job-search/discussions/78) catalogs the ones other forks have built.
2. **Document templates** - `/add-template` registers any CV or cover-letter toolchain that compiles to PDF from the command line, LaTeX or otherwise.
3. **Evaluation criteria** - deal-breakers and preferences in your profile are free-form, and the evaluation rubric scores against whatever you put there. "Strong parental-leave terms", "minimum salary X per my union's scale", "no on-call" - each is one profile line, no code, and it carries real weight in `/rank` and `/apply` fit evaluations. Language is the one deal-breaker type with dedicated, structured handling: `/setup` captures every language you work in and your level (asked directly, or inferred from your CV/LinkedIn export) into a `Languages` table, and the Language Gate (`04-job-evaluation.md`) hard-rejects a posting that requires a language you haven't declared at all, while flagging - not auto-rejecting - one that asks for a higher level than you declared in a language you do work in, so a borderline case (a strict "fluent" bar against your own B1/B2, say) gets your judgment instead of a silent drop.

**Borrowing a portal skill from another fork** is the intended way to get a board that upstream doesn't ship: find it in the [portal index](https://github.com/MadsLorentzen/ai-job-search/discussions/78), open that fork, and copy the one folder into your own `.agents/skills/`. Before you run it:

- **Read the code.** All of it - these CLIs run pre-approved on your machine (`.claude/settings.json` allowlists them) against your career data. Check that the only network calls go to the job board it claims to search, that `package.json` has no `dependencies` and no lifecycle scripts (`postinstall` etc.), and that nothing reads or writes outside its own folder.
- **Run its tests offline** (`bun test` in the skill's `cli/` directory) - a well-built skill's tests pass with no network access.
- Check the `enabled:` flag and the skill's own ToS notes.

The copy step is manual on purpose. Your settings already allow installed portal skills to run without asking each time - so an installer that fetched them from third-party repos for you would skip the one check that matters: you, reading the code first. There isn't one, and that's a security decision rather than a missing feature.

Market-specific *data sources* (a national salary database, local award-rate tables) follow the same pattern as portals: they belong in a market fork, shared via [#78](https://github.com/MadsLorentzen/ai-job-search/discussions/78), not upstream.

### Salary benchmarking

The salary tool works with any salary data you provide (union statistics, Glassdoor exports, personal research, etc.). See `tools/README_SALARY_TOOL.md` for the expected format and setup. If you don't have salary data, the salary step is simply skipped.

### Starting over

To wipe your profile data and start fresh:

```
/reset profile    # clears skill files, preserves framework rules
/reset documents  # deletes files from documents/ folder
/reset all        # both
```

`/reset` shows exactly what will be deleted and requires you to type `RESET` to confirm. Nothing is deleted until you do.

### Staying up to date

Upstream moves fast. Rather than pulling raw `master` and hoping, update your fork to a tagged [release](../../releases) - a vetted checkpoint described in [CHANGELOG.md](CHANGELOG.md). `python3 tools/check_upstream_updates.py` previews exactly which of your personalized files an update touches before you merge, and `python3 tools/upstream_triage.py` sorts the commits you're behind into "worth reviewing" vs "probably skip" (a weekly workflow can post this to a rolling issue). Full walkthrough in [SETUP.md, section 8](SETUP.md#8-pulling-upstream-updates-into-your-fork).

## Tips for better results

### Profile depth matters

The single biggest factor in output quality is how much detail you put into your profile. A thin profile produces generic applications; a detailed one enables genuinely tailored results.

- **Role descriptions:** Don't just list job titles. Describe what you actually did in each position: specific projects, tools used, responsibilities, and measurable achievements. The more material you provide, the more precisely the system can reframe your experience for different roles.
- **Skills in context:** Instead of listing "Python" or "project management," describe how and where you applied them. "Built ML pipelines for customer churn prediction in Python using scikit-learn" gives the system far more to work with than "Python, machine learning."
- **All onboarding paths work:** Whether you point `/setup` at your `documents/` folder, paste a single CV, or walk through the interview, the principle is the same: richer input produces sharper output.

### Career path discovery

The framework supports two distinct modes of job searching:

- **Explicit targeting:** You know which roles or sectors you want. The system helps refine and prioritize based on fit.
- **Latent opportunity discovery:** By analyzing your full history (not just job titles, but the actual work you did), the system can surface career paths you haven't considered. Transferable skills that map to unexpected industries, patterns in what you enjoyed or excelled at, or emerging roles that combine your domain expertise with new technology.

To get the most from this, invest time during `/setup` in describing not just your experience, but what energized you, what drained you, and what you'd want more of. This context directly shapes how the system evaluates fit and which roles it surfaces during `/scrape`.

## Contributing

Thinking about a PR? Read [CONTRIBUTING.md](CONTRIBUTING.md) first - it explains what gets merged, what lives in forks, and why.

## Acknowledgements

- [MadsLorentzen/ai-job-search](https://github.com/MadsLorentzen/ai-job-search) as the upstream project
- [Mikkel Krogholm](https://github.com/mikkelkrogsholm) ([skills repo](https://github.com/mikkelkrogsholm/skills)) for the job search CLI skills
- Built with [Claude Code](https://claude.com/claude-code) by [Anthropic](https://anthropic.com)

## License

本项目使用 MIT License。详见 [LICENSE](LICENSE)。

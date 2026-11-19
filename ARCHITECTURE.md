# 架构决策记录（ADR）：账号与仓库分层

> 状态：生效中 · 最近修订 2026-11-09
> 本文件回答一个问题：**一个新仓库该放哪、什么时候该搬家、什么时候该归档。**

## 1. 切分轴

切分依据是**访客该得出什么结论**，不是语言、不是领域、也不是时间先后：

| 访客的问题 | 该看的地方 |
|---|---|
| 他会写系统代码吗 | `holtwood-systems` |
| 他扛得住真实评测和限时吗 | `holtwood-arena` |
| 这些代码是他自己写的吗 | `holtwood-forks`（明确排除） |
| 他以前做过什么 | `holtwood-projects` / `holtwood-genomics`（归档） |
| 这个人是谁、入口在哪 | `holtwood`（Profile / 索引 / 个人产品） |

按"语言"或"领域"切会同时混进原创与二手代码，把作品集的可信度打薄；按"访客结论"切才让每一层都能被独立验证。**这是唯一需要守住的规则。**

## 2. 放置判据

只有四条，按顺序判断，命中即停：

1. **不是自己写的** → `holtwood-forks`。包括上游研读副本、带本地补丁的实验分支、汉化/移植的衍生作品。
2. **有官方评测、榜单、限时提交** → `holtwood-arena`。包括竞赛工作区、题解记录、判题台账、榜单监视。
3. **是仍在推进的原创系统工作** → `holtwood-systems`。
4. **其余**：属于个人身份/工具/站点/个人产品 → `holtwood`；已停止推进的应用或领域主线 → `holtwood-projects` / `holtwood-genomics`。

判据之外的常见误判：
- "它很完整"不是放进 systems 的理由，**是否仍在推进**才是。
- "它很久了"不是放进归档的理由，**是否还有主线投入**才是。

## 3. 晋升规则（arena → systems）

竞赛产物默认留在 `holtwood-arena`，但满足以下**全部三条**时提升到 `holtwood-systems`：

1. 代码已脱离赛题脚手架，能在仓内独立构建与运行；
2. 有一条能在普通硬件上复现的验证路径（不需要评靶机）；
3. 赛期已结束，且不涉及赛期保密约束。

提升时应在新仓库说明来源赛事与赛题，并在 `holtwood-arena` 留下指向新仓库的说明；**不删除**竞赛工作区。

## 4. 补丁规则（forks）

`holtwood-forks` 允许存在本地补丁分支，但必须满足：

- 组织描述与 Profile 必须写明"允许本地补丁"，不得暗示全部是零改动镜像；
- 补丁分支**不计入作品集**，也不在简历语境中称为个人项目；
- 补丁存活超过两周、且有可复现结果或已向上游提 PR 的，按第 3 条评估是否提升到 `holtwood-systems`；
- 衍生作品（汉化、移植、二次修改）在仓库 README 顶部标注上游与改动范围。

## 5. 归档规则

满足任一即归档，并在描述开头写明 `Archived: superseded by X` 或 `归档：不再推进`：

- 能力已被同领域后继仓库覆盖（例：`bwa-rust` → `minibwa-rust`，`open-infra-ai` → `portfolio`）；
- 领域主线整体停止投入（例：生信主线转入维护模式）；
- 赛期与赛后整理均已结束。

**归档不删除。** 被接管的旧实现是"能力随时间收敛"的证据，删除等于抹掉判断过程。

## 6. 命名规则

- 组织一律 `holtwood-<域>`，全部小写连字符；不接受不带前缀的组织名（会读成社区项目，也会从命名空间里漏出去）。
- 仓库名 = **主体 + 可选类型后缀**，全小写；多词用连字符，单 token 品牌名不加连字符（`bitcal`、`utop`、`wb2api`、`recipegrid`、`cudaimg`）。
- 类型后缀**允许且鼓励**，但必须一个词、能消歧。现行约定（由 141 个存量仓库归纳，不是臆造）：

| 后缀 | 用在 | 现存例子 |
|---|---|---|
| `-index` | 结构化数据 + 生成视图的索引层 | `stars-index` `arena-index` |
| `-registry` | 权威登记 / 治理台账 | `repo-registry` `repo-registry-private` |
| `-guide` / `-foundations` | 教学与学习主线 | `cpp-high-performance-guide` `cuda-foundations` |
| `-plan` / `-prep` / `-kit` | 计划、求职资料、工具集 | `ai-infra-serving-study-plan` `career-prep` `agent-skills-kit` |
| `-lab` | 实验体系 | `ascend-llm-infra-lab` |
| `-site` | 站点仓库 | `imshane-site` |
| `-server` | 与同名项目配套的服务端 | `cloud-bench-server` |
| `-minigame` | 微信小游戏（区别于小程序） | `melon-bump-minigame` |
| `-rust` / `-zh_CN` 等 | 同一工具的第二实现 / 本地化变体 | `fq-compressor-rust` `minibwa-rust` |

- 竞赛仓库：`<赛事>-<年份>-<赛道>-<序号>-<题目slug>`，例 `cann-2026-power-04-det`。
- Fork 沿用上游名，不改（`vllm`、`Triton-Puzzles`、`Termius-Pro-zh_CN`）。
- 反面清单：不要把语言写进名字（`xxx-cpp`）；不要造无信息量的泛名（`my-project`、`tools`）；不要在一个功能域里混三种命名风格。
- 默认不为整齐而批量重命名；确需改名时必须同时处理 Pages 断链（同名跳转仓或自定义域名）并更新全部索引。

### 2026-11-19 改名决策：`repos-db` → `repo-registry`，`stars-db` → `stars-index`

`-db` 容易被读成数据库服务，实际两者都是「JSON 事实源 + 生成视图」。本次按显式决策更名：

- `repo-registry`：权威仓库登记与治理台账；私有对照为 `repo-registry-private`。
- `stars-index`：GitHub Star 收藏索引。

Pages 影响已处理：GitHub 只对仓库链接做 301，**project site URL 不会重定向**。因此旧仓名
`holtwood/repos-db`、`holtwood/stars-db` 保留为极小跳转仓，把
`holtwood.github.io/repos-db/`、`/stars-db/` 指向新站点；新地址为
`holtwood.github.io/repo-registry/`、`/stars-index/`。跳转仓不含业务数据，索引与数据只在新仓维护。

## 7. 可见性与索引

- 私有仓库的名称与描述不进入公开文档，只进入公开统计数字。
- 索引层只有 `repo-registry`（公开）与 `repo-registry-private`（私有）维护仓库清单；其余页面是视图，冲突时以 `repo-registry` 为准。
- `holtwood-arena` 的私有赛事仓库以 `arena-index/arena.yaml` 为**唯一事实源**；`repo-registry-private`
  只统计该账号的私有数量，不重复登记赛事仓。两处都出现时以 `arena.yaml` 为准。
- 公开文档中出现性能数字时，必须绑定硬件、提交与测量方式；无复现路径的数字标注为"未独立复现"。
- **有 Pages 站点的仓库必须填 homepage**，且指向当前组织名下的 URL。组织改名不会重定向项目站点，
  旧组织域名的链接会直接 404（2026-11 已修复 `open-infra-ai.github.io` 与 `open-genomics.github.io` 的历史死链）。
- 文档载体固定为 README / `docs/` / Pages；不启用 Wiki 与 Projects 作为文档源（保持默认或关闭即可）。

## 8. 运维要求

索引的准确性依赖两个定时任务，二者都需要**用户级 PAT**（`GITHUB_TOKEN` 无法跨账号读取，也无法读取 Star Lists）：

| 仓库 | Secret | 用途 | 失效表现 |
|---|---|---|---|
| `holtwood/repo-registry` | `AUDIT_TOKEN` | 跨 6 个账号拉取仓库清单与 ahead/behind | 审计步骤报 `Bad credentials (HTTP 401)` |
| `holtwood/stars-index` | `STARS_TOKEN` | 读取 Star 与 Star Lists | `Fetch stars` 步骤报 `Bad credentials (HTTP 401)` |

**PAT 会过期，过期后索引静默变旧。** 轮换后须手动触发一次 workflow，确认徽章日期前进。

## 9. 未决事项

以下两项需要人工在网页端操作，脚本或 API 无法代为完成：

1. **`holtwood/repo-registry` 的 `AUDIT_TOKEN` 与 `holtwood/stars-index` 的 `STARS_TOKEN` 未轮换**。两个 workflow 自 2026-11-09 起持续报 `Bad credentials (HTTP 401)`；索引目前靠本地手动审计维持（见 §8）。
2. **`electronic` 组织不可管理**。`GET /orgs/electronic/teams` 返回 403，说明当前账号不是其 admin，因此无法删除或改描述。它不属于本架构，保持现状即可。

已完成：`open-genomics` → `holtwood-genomics` 组织改名（2026-11-19）。本地目录、13 个仓库的 `origin`、Pages 域名引用与索引数据均已同步；旧项目站点不重定向，因此文档中的 `*.github.io` 必须使用新域名。

已收敛（勿重复讨论）：`repo-registry` / `stars-index` 保留原名（§6）；`career-prep` 归个人号、`ai-infra-interview-prep` 归档；竞赛仓一律进 `holtwood-arena`。

## 10. 本地工作区

本地镜像与远端同构，路径固定为 `~/github/<账号>/<仓库名>`：

- 仓库改名或迁移后，**本地目录与 `origin` 必须同步改**，否则下次 `git pull` 会落到旧路径或旧组织。
- 允许在某个仓库内部放**嵌套工作副本**（例如 `muse-spark-open-kernel-challenge/serving/sglang`），
  但它的 `origin` 必须指向真实仓库；这类副本不计入镜像清单。
- 日常同步：先 `git fetch --prune`，再看 `HEAD..@{u}`；只允许 `--ff-only`，不做隐式合并。
- 出现 `ahead > 0` 且远端已重写历史时**不自动合并**：先比较双方独有提交，必要时把旧本地主线打成
  `archive/...` tag，再删除 backup 分支；不保留两套长期分叉的主线。
- 默认分支：**新建仓库用 `main`**；存量 32 个 `master` 不做批量改名（改名会打断 clone、badge 与 CI 引用，
  收益为零），fork 一律跟随上游分支名。
- **账号目录下只允许放 git 仓库**。非仓库材料统一进 `~/github/_inbox/<主题>/`，处理完再归档或删除；
  空占位目录不留（要有东西才建目录）。
- 例外：`holtwood-arena/groups/` 是 `arena.py link` 生成的软链接视图，属于工具产物，保留；
  只生成 `cannjudge/`、`events/`、`organization/` 三棵语义树，旧的平铺别名已移除。
- 例外：`holtwood-arena/flagos/` 是 FlagOS 系列的内部分组容器，具体仓库与路径由
  `arena-index/arena.yaml` 的 `local_dir` 定义；账号目录下不再平铺 `flagos-s2-*`。
- 未版本控制的工作区属于**风险项**：2026-11 发现 `holtwood-arena/amd`（AMD AI League，160 MB、
  34 个源文件）从未 `git init`，已建为私有仓 `amd-2026-ai-league`。新工作区应第一天就建仓。
- **归档仓库仍需修文档时**（如组织改名导致的死链）：临时 `archived=false` → 推送 → 立即恢复
  `archived=true`；若分支保护要求状态检查，则走分支 + PR，不直接改默认分支。

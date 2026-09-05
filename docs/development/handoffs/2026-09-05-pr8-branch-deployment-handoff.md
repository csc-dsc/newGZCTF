# newGZCTF 分支交付与服务器验证交接

更新时间：2026-09-05。范围仅限 newGZCTF；本文件是此次新会话接手入口。

## 任务目标

- 用户已明确此前的“PR2”实际指 PR #8，不再需要确认编号。
- 用户要求取消 PR #8，改为独立分支交付，不创建新 PR、不合并 main。
- 下一会话继续对比该分支与 `10.24.0.27` 生产版本，验证完整代码能否在隔离环境部署运行，修复后只推送分支。
- 本轮只完成分支交付与文档交接，不继续服务器部署测试；不能把测试授权扩大为生产切换授权。

## 基线与交付

| 项目 | 已核对事实 |
| --- | --- |
| 唯一活动 worktree | `D:\newGZCTF` |
| 原 PR #8 代码提交 | `5047a8afe31aa77f357d630b206ec5267bb2049b` |
| 原分支 | `codex/practice-consolidation`，保留，不删除 |
| 当前分支 | `codex/practice-deployment-validation`，直接继承原 PR 提交，不重写历史 |
| 已推送远端 | `fork-ssh`，即 `csc-dsc/newGZCTF` |
| 主仓库 main | `Y3X1L2/newGZCTF`，核对时为 `bbd5a5d4da8488ad4c32c7bf49523f3136e63831` |
| PR #8 | 已关闭，`closed_at=2026-09-05T14:56:57Z`，`merged_at=null` |
| 主仓库写权限 | API 返回 `push=false`，SSH dry-run 明确拒绝 `csc-dsc`；未能在主仓库创建分支 |
| 交接提交 | 在上述代码基线上仅改文档；精确 SHA 通过 `git log -1` 获取，并核对远端同名分支 |

- 分支：https://github.com/csc-dsc/newGZCTF/tree/codex/practice-deployment-validation
- 已关闭 PR：https://github.com/Y3X1L2/newGZCTF/pull/8
- 用户认为拥有主仓库建分支权限，但当前凭据实测未获得；后续如需移至主仓库，先确认协作者权限或正确账号，不 force push、不再提交 PR。
- 原 #6/#7 及本地资料保留在轻量归档引用与 `.local-notes`，无需克隆或恢复旧 worktree。

## 当前状态

状态：`in progress`。分支交付已完成，完整部署验收为 `NOT_RUN`。

- 已完成：生产只读基线、PR 目标确认、PR 关闭、新分支保存、源码差异初步统计。
- 尚未完成：11 项集成失败定位与修复、完整 CI 门禁、同一提交完整发布包、服务器隔离启动和业务链路验收。
- 本轮及前一轮服务器核对没有创建测试目录、数据库副本、容器或部署制品，没有生产切换、服务启停或生产数据修改；不存在本任务待清理的远端测试资源。
- 不把本文件交接动作称为新会话已经运行，也不把先前 CI 结果归属于未来源码提交。

## 与生产代码的初步对比

生产提交 `3e5526dc1ce336ac5545faacd49a9c0d1ec7ab58` 至原 PR 代码 `5047a8a`：
`49 files changed, 2435 insertions(+), 264 deletions(-)`。这是源码树差异，不是完整运行兼容性证明。

主要增量：

- 练习管理：停用题可管理，容器导入字段、Ready/Docker 模板解析、附件 FileHash 绑定、编辑时清理旧运行字段。
- Content：资产上传与读取 API、Content-Digest、幂等键锁、Token 资源授权优先于上传者身份；未开放外部删除接口。
- Blob：真实业务引用与事务锁保护，覆盖练习附件、头像、海报、课程封面；保留有效上传授权历史。闲置字节自动回收未实现。
- TeamLab：测试目录/能力探测隔离，明确覆盖 nftables/iptables；不改变生产默认行为。
- CI/OpenAPI：生成上传二进制契约、补回归测试；覆盖率仅排除生成的迁移 Designer/Snapshot，不排除手写 Up/Down。

此增量没有修改 migration、Designer 或 ModelSnapshot。创建者产品能力本次未重新启用。
`20260815012026_AddExerciseCreatorTracking` 的历史恢复已在 main，原始提交
`c6a2b7f4b5637f5622cfa6bdb42624d5242a0c80`；不得另造 migration ID 或覆盖历史快照。
两条旧 Theory migration 来源仍未恢复，详见 `current-state.md`，不得伪造。

## 验证证据

CI：https://github.com/Y3X1L2/newGZCTF/actions/runs/33960930409
以下成绩仅对应原代码提交 `5047a8a`，不是本次新建分支的独立运行结果。

| 验证项 | 结果 |
| --- | --- |
| 前端门禁 | 通过，280 项测试；既有记录含类型/locale/lint/架构及生产构建 |
| 后端单元测试 | 971/971 通过 |
| 后端集成测试 | 265/276 通过，11 项失败，不能称全绿 |
| 新 Blob PostgreSQL 回归与生成 OpenAPI 快照 | 通过 |
| 后续迁移模型、查询计划、向后兼容步骤 | 被集成失败阻断，未执行 |
| 服务器完整发布与业务验收 | 未执行 |

11 项失败待办：

1. 7 项旧 VM 测试调用 `/instance-credentials`，当前路由是 `/remote-access`；先核对现行契约再修测试。
2. 2 项 TeamLab 契约冲突：无幂等头的 lease 返回 201，而旧测试一律要求 202；schema 的 `lastError` 与旧禁止断言冲突。不能为全绿擅改产品契约。
3. 2 项图片 GET/DELETE 返回 500，尚未取得确定根因。`ImageStorage` 初始化 `/var/lib/gzctf/images` 的权限问题只是候选，需要异常栈验证。

## 服务器只读基线

以下为本次任务前段的 SSH 实测快照，接手后需重新检查；不表示已部署候选分支。

| 项目 | 实测 |
| --- | --- |
| SSH | `whoami@10.24.0.27:22`，hostname `whoami`，sudo 成功 |
| 活动目录 | `/opt/gzctf/releases/docker-provisioning-inventory-3e5526dc-20260904T093342Z/publish` |
| 活动链接 | `/opt/gzctf/publish` 指向上行；`publish/files` 指向 `/opt/gzctf/shared/files` |
| manifest | gitCommit `3e5526dc1ce336ac5545faacd49a9c0d1ec7ab58`，994 个文件 |
| GZCTF.dll SHA-256 | `c85a1e35822fb31e009927591441cc218b158ff6006a1dd099f8db6ad1f6241c`，与 manifest 一致；本次未声称逐一复核其余 993 个文件 |
| 服务 | `gzctf.service` PID 2692、`gzctf-agent.service` PID 2691，均 active/running、NRestarts 0 |
| 数据库 | 134 条 migration，head `20260816192540_TeamLabCapabilityClosure`；练习 605、ImageTemplate 456 |
| 数据库大小 | 28,145,335,319 bytes，约 26.2 GiB |
| 资源 | 根盘剩余约 49G；内存约 31GB、可用约 29GB |
| .NET | SDK 10.0.300；ASP.NET/Core runtime 10.0.8；生产 runtimeconfig 请求 10.0.0 |
| 基础依赖 | Docker 存在；生产 PostgreSQL 16 / Redis 7 容器运行 |
| 可用镜像 | 已缓存 postgres:16-alpine、redis:7-alpine、ubuntu:22.04、busybox:latest；未发现 dotnet 基础镜像 |
| 构建缺口 | 未找到 Node/pnpm 或常见用户 NuGet cache，不能假设服务器可离线构建 |

连接使用已有 known_hosts；本机现有 Paramiko 可用。认证材料只从本地私有来源读取，
见 `.local-notes/README.md` 的连接说明，不提交或打印密码、完整连接串、数据库或 Cookie。

## 隔离验证方案与禁区

1. 不直接运行 `deploy-gzctf-release.py` 或 `activate-gzctf-release.sh` 做旁路测试。它们会停生产服务、替换 `/usr/local/bin/gzctf-agent`、迁移所复制配置指向的数据库并切换链接；仅修改部署根目录不足以隔离。
2. 主站启动即写库：`EntityConfigurationProvider` 和 `PrelaunchHelper` 自动迁移/初始化，多个 Runtime/API/Webhook Worker 无条件启动。未找到全局关闭开关；`Agent:LocalNodeSchedulable=false` 不是安全模式。
3. 使用独立 PostgreSQL、Redis、files、临时文件、日志与 DataProtection 目录；不复用生产配置、数据目录、Redis 或节点。先空库，再评估服务器内生产副本升级，不下载生产数据库到本地。
4. 数据库约 26.2 GiB 而余量约 49G，必须先预算 dump、恢复、WAL 和制品空间，不能直接执行整库克隆。
5. 可研究单独 `--network none` namespace，让测试 DB/Redis/应用共享 loopback，服务器用 `nsenter` 访问。禁止挂载 Docker/libvirt socket、KVM、生产目录，禁止 host network 或 privileged；必须实测无法连接原节点/Registry。单独 `--internal` 网络不自动等于无宿主访问。
6. 显式隔离 `ConnectionStrings:Database/Storage`、Redis 配置/前缀和 Registry；空 Registry 会回落生产 `10.24.0.28:5000`。禁用 Nginx 配置同步与 Portal SSO；克隆库需评估原加密配置兼容性，不能输出密钥。
7. 真正监听配置为 `ServerPort`/`MetricPort`；`/healthz` 在 MetricPort（默认 3001），不是 8080。空库完整 Web 验收不等于 Docker/KVM/TeamLab 实际执行链通过。
8. 构建入口为 `scripts/deployment/build-gzctf-release.ps1`，输出主站、efbundle、Agent 和相关组件及清单。默认 `FrontendBuildMode=Source` 可能自动安装依赖；未获安装许可前，应使用同提交前端 artifact 和 `FrontendBuildMode=Artifact`，不能用旧前端或 Skip 冒充完整包。

## 下一会话执行顺序

1. 先读本文件、`AGENTS.md`、`current-state.md`，核对工作树、远端分支与主仓库权限。不重开或新建 PR。
2. 从 `codex/practice-deployment-validation` 继续现有任务，不从旧 PR #6 覆盖代码。详细复核生产 SHA 与候选 SHA 的 diff、迁移集合、前端制品和运行需求。
3. 取 CI 异常栈，按现行契约修复 11 项失败，完整跑门禁。新增依赖/软件安装必须先征得用户同意并明确路径。
4. 设计可执行的隔离配置和磁盘预算，记录测试资源清单；先验证隔离，再构建同 SHA 完整包和开展启动、迁移、登录、权限、上传幂等、题目导入与附件读取测试。
5. Docker/KVM/TeamLab 必须另有隔离执行资源才能实测；不足时明确写 NOT_RUN，不冒用生产节点。
6. 核对生产链接、manifest、服务与迁移未被测试改变，报告证据、剩余缺口及服务器测试资源实际状态。
7. 提交修复与脱敏报告，正常 push 此分支，验证远端 SHA；不合并 main、不创建 PR，未获明确生产切换授权不得上线。

## 本地工具与资料

- .NET SDK：`D:\tools\dotnet-sdk-10\dotnet.exe`；Python/Paramiko：`D:\Python\python.exe`；GitHub CLI：`D:\Github\gh.exe`。
- 既有前端依赖位于 `src/GZCTF/ClientApp`；生产 NuGet 依赖在用户缓存，测试依赖不完整，不得静默 restore/install。
- `.local-notes/legacy-20260905` 保留原 12 个文件和本地资料；`.local-notes/contract-export` 是离线 OpenAPI 工具，不是第二个 worktree。清理曾被执行策略拦截，不换工具绕过。
- GitHub SSH 推送使用已配置的 `fork-ssh` 与本机既有 SSH key。API 走旧代理曾反复 EOF，临时移除进程代理后成功关闭 PR；不要修改系统代理或全局 SSH 配置。
- 新会话只承接本文件所列 newGZCTF 工作，不承接此前其他仓库、软件配置、磁盘清理或其他服务器任务。

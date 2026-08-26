# 严西湖水质管理仪表盘：Netlify 静态部署实施计划

> **目标**：放弃现有 IGA Pages（报 401 的托管），改用 **Netlify** 做公开无鉴权的静态部署，消除 401；同时为项目配置 GitHub 远程仓库，实现后续自动部署。
> **For agentic workers:** REQUIRED SUB-SKILL: superpowers:writing-plans。本计划为纯部署流程，**不修改任何业务代码/页面内容**，仅做目录级部署操作。

---

## 0. 背景与诊断结论

- **项目性质**：单文件 HTML 大屏仪表盘（`index.html` = `index.v4.0.html`，51KB 自包含），数据全部硬编码，**无后端 API、无鉴权逻辑**。轮播图 53 张在 `轮播图/` 目录，ECharts 走 CDN。
- **401 根因**：页面本身无需鉴权；401 来自当前托管平台（IGA Pages `o8kft4n02q`、曾规划 veFaaS APIG 网关）的**访问鉴权/网关配置**，浏览器访问首页被拒绝。
- **修复方向**：Netlify 纯静态托管默认公开无鉴权，直接消除 401。
- **已确认决策**：✅ 用 Netlify CLI ✅ 用 Netlify 默认域名 ✅ 配置 GitHub 仓库 `HtO404`。

### 环境探察结果（已完成，只读）

| 前置条件 | 状态 |
|----------|------|
| Node / npm | ✅ v22.14.0 / 10.9.2 |
| gh CLI | ✅ 2.74.2，已登录 `HtO404`（GH_TOKEN） |
| git user | ✅ 仝胡 / 17398206513 |
| Netlify CLI | ⚠️ 未安装（本计划安装） |
| git 远程 | ⚠️ 无（本计划创建） |

---

## 1. 文件结构（部署用）

| 文件/目录 | 作用 | 是否改动 |
|-----------|------|----------|
| `index.v4.0.html` | 唯一源码，本次**不改** | ❌ |
| `index.html` | 部署入口，与源码一致 | ❌ |
| `轮播图/` | 53 张轮播图片，随目录一并上传 | ❌ |
| `docs/superpowers/plans/2026-08-26-*.md` | 本计划文档 | 新增 |
| `.gitignore` | 已有，含 `.iga/`；追加忽略 `/.netlify` | ✅ 追加一行 |
| GitHub 远程仓库（新建 `HtO404/yanxi-lake-dashboard`） | 版本托管 + Netlify 自动部署源 | 新建 |

---

## 2. 任务分解

### 任务 1：配置 GitHub 远程仓库并推送

**说明**：本地已是 Git 仓库（`main` 分支，2 个提交），无远程。用 gh 创建同名校公共仓库并按需私有。

- [ ] **Step 1**: 创建 GitHub 仓库（连接账号 `HtO404`）
```powershell
gh repo create yanxi-lake-dashboard --public --source . --remote origin --push
```
  **期望**：仓库创建成功，`origin` 指向 github.com:HtO404/yanxi-lake-dashboard.git，本地 AOD0 提交被推送，`personal-summary.html/.md` 等已有修改先按现状保留（不强制 commit 未暂存改动）。

> ⚠️ 当前 `git status` 有未暂存改动（index.html 修改、删除文件、新增 personal-* 等）。**推送前需决策**：方案 A `git add -A && commit`（把含 personal-summary 的新内容一并入库）；方案 B 仅 push 已有 2 个提交，未暂存改动留给后续。默认**方案 A**（干净基线），在执行时提示。

---

### 任务 2：本地预览验证（上线前必做）

**说明**：确认 `index.html` 在本地静态服务器下可正常渲染，图片/图表/CDN 无 404。

- [ ] **Step 1**: 启动本地静态服务器
```powershell
Set-Location "e:\TraeCode\严西湖水质管理仪表盘"
python -m http.server 8080
```
- [ ] **Step 2**: 浏览器访问 `http://localhost:8080/index.html`，DevTools Network 检查：
  - [ ] 标题"严西湖水智眸智慧监测平台"
  - [ ] 轮播图自动播放、左右箭头/圆点可切换
  - [ ] 趋势图/热力图/雷达图/WQI 仪表盘渲染
  - [ ] `轮播图/` 下图片全部 200，ECharts CDN 200，无 401/404

---

### 任务 3：安装并登录 Netlify CLI

**说明**：Netlify 登录需在**浏览器完成 OAuth**（用户本人操作一次，进程会暂停等待）。

- [ ] **Step 1**: 全局安装
```powershell
npm i -g netlify-cli
```
- [ ] **Step 2**: 登录（触发浏览器授权）
```powershell
netlify login
```
> ⚠️ 需用户在弹出浏览器中登录/授权 Netlify 账号。若已有 access token，可 `NETLIFY_AUTH_TOKEN=<token>` 免交互。

---

### 任务 4：初始化并静态部署到 Netlify

**说明**：目录本身为纯静态（无构建），publish 目录 = 项目根目录。

- [ ] **Step 1**: 初始化站点（可 `--manual` 创建新站点）
```powershell
netlify init --manual
```
  选择"Create & configure a new site"，站点名保持 `yanxi-lake-dashboard`，publish 目录填 `.`（项目根，含 index.html 与 轮播图/）。无 netlify.toml 时 CLI 会生成。
- [ ] **Step 2**: 新增 `netlify.toml`（纯静态声明，若 init 未生成）
```toml
[build]
  publish = "."

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```
> SPA 兜底：使所有路径回到 index.html（本单页可含，但非必需）。
- [ ] **Step 3**: 部署到生产环境
```powershell
netlify deploy --prod --dir .
```
  **期望**：上传根目录（含全部 HTML + 轮播图），输出 https://yanxi-lake-dashboard.netlify.app 及预览地址。
- [ ] **Step 4**: 查询线上地址
```powershell
netlify open:site
```

---

### 任务 5：线上验证

- [ ] 浏览器访问默认域名，重复任务 2 的检查项，确认无 401/404。
- [ ] 在新建 GitHub 仓库上开启 Netlify **Git 集成**（此后每次 push 到 `main` 自动部署）：
```powershell
netlify init   # 选择 "Connect to existing GitHub repo"，指向 HtO404/yanxi-lake-dashboard
```
> 若已通过 `netlify init --manual` 建站，可改为控制台"Connect to Git"；二选一，避免重复建站。

---

## 3. 关键决策与假设

| 决策 | 说明 |
|------|------|
| **Netlify 默认域名** | `yanxi-lake-dashboard.netlify.app`，免费零配置，先验证。 |
| **纯静态 publish=根目录** | 无构建流程，直接上传全部文件（含中文文件名的 `轮播图/`，Netlify 原生支持 UTF-8 文件名）。 |
| **GitHub 私有与否** | 当前消息建议 public 便于访问；若含敏感内容（轮播图为公开研学照片），可改 `--private`。执行前确认。 |
| **推送前提交未暂存改动** | 默认 `git add -A && commit` 建干净基线；若用户不想把 personal-* 入库，改方案 B。 |
| **Netlify 登录交互** | OAuth 弹窗需用户本人操作，是唯一必须人工介入的步骤。 |

---

## 4. 风险与回滚

- **风险：git push 携带未暂存改动** — 推送前先与用户确认提交范围（A/B 方案）。
- **风险：仓库私有性误判** — 重建/改可见性：`gh repo edit HtO404/yanxi-lake-dashboard --visibility private`。
- **风险：Netlify 站点名冲突** — `netlify deploy` 报 Site already exists 时，改为随机后缀名。
- **风险：登录失败** — 用 `NETLIFY_AUTH_TOKEN` 环境变量注入 token 替代交互登录。
- **回滚**：本地 `index.v4.0.html` 为永不改动的源码副本；Netlify 后台每次部署自动建档，可在 Deploys 页回滚到历史版本；GitHub 仓库可删 `gh repo delete`。

---

## 5. 完成标准

1. `https://yanxi-lake-dashboard.netlify.app` 可公开访问，无 401。
2. 页面所有板块、轮播图、图表、CDN 加载正常，无 404。
3. GitHub `HtO404/yanxi-lake-dashboard` 已创建并接收 `main`。
4. （可选）Netlify Git 集成开启，后续 push 自动部署。
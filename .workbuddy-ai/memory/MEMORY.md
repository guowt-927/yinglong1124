# 应龙平台原型 - 项目记忆

## 产品方向（2026-11 确定，11-24 二轮收敛后）
- 支持 K8s 集群 + Docker 主机**同时纳管**的混部形态；平台核心不感知运行时，统一 Runtime 接口 + K8s/Docker 适配器
- yl-agent：Go 单二进制、systemd 托管、挂载 docker.sock、gRPC 主动外拨 + mTLS；注册流程 3 步（连通性→安装 Agent→Agent 上线）
- **Docker 主机定位为「轻纳管」**：不做环境预检、不做 Agent 版本管理/升级、不做批量导入/升级、不暴露共享存储
- **Docker 服务生命周期极简**：启停/删除/编辑统一在服务列表操作（**运行中不可编辑**，仅停止后可编辑）；主机详情为纯展示，无操作列、无部署说明；**不做蓝绿更新、不做在线扩缩容、不做跨主机重调度、不提供日志入口、不展示存量导入**
- Docker 部署时加速卡由平台按顺序自动分配，**不提供选卡网格**；保留「实例上限提示」
- 预警体系只覆盖 K8s 集群（节点异常 / SLA），**不含 Docker 主机类预警**
- 存量容器导入功能整体不在原型中展示（服务列表、主机详情、集群页均无「存量导入」）
- 集群与主机页：「新建集群」(`newClusterModal`，仅 K8s 账号/配置文件两 Tab) 与「注册 Docker 主机」(`hostRegisterModal`，仅 IP/名称/SSH 端口/账号/密码 5 个字段 + 3 步进度，**无标签、无安装方式**) 是**两个独立弹窗**，顶部按钮 `openRegisterModal()` 按当前运行环境分流
- 工作台「历史预警」弹窗为纯列表：时间 / 级别 / 预警项 / 对象 / 详情 + 分页，**无筛选、无搜索、无持续时长/状态列**
- 不做 DinD；跨运行时同一服务部署放二期

## 原型技术栈
- 纯 HTML + 原生 JS（内联 script），CSS 分三层：`yinglong-v2.css` / `prototype-v4.css` / `assets/css/pages/*.css`
- 侧栏由 `prototype-v4.js` 注入；部署弹窗由 `model-deploy-modal.js` 动态渲染
- 静态资源引用带 `?v=` 版本号，**改 JS/CSS 后必须递增**（当前 infra 为 `-7`，node-detail 为 `-6`，service/deploy-modal 为 `-5`，model-mgmt 为 `-6`，overview 为 `-5`）

## 验证工具链
- `node --check` 校验 JS；Python 标签平衡脚本校验 HTML
- `agent-browser` + `AGENT_BROWSER_EXECUTABLE_PATH` 指向本机 Edge 截图（playwright 不可用、Chromium 下载超时）
- agent-browser daemon 在 Bash 调用间不持久，open→eval→screenshot 放同一条命令
- Python 写含 `${...}` 的 JS 时必须用 raw string `r"""..."""`，否则 `\$` 转义出错

## 已知坑
- `prototype-v4.js` 会给 `nodeGpuUtilization` 打「V1不含」徽章，Docker 模式下已改 id 为 `hostGpuUtilization`
- 工作台预警条 `.alert-strip` 需 flex-wrap，预警项用 `.alert-item` 包裹；按钮包在 `.alert-actions` 里（`margin-left:auto` 在容器上）
- 服务列表 `rowActionMarkup` 的静态 HTML 和 JS 模板要同步改，否则首屏和筛选后不一致
- `model-management.css` 第 483 行是压缩单行，正则删规则容易留残片，删后要 grep 确认
- `yinglong-v2.css` 的 `.full{grid-column:1/-1}` 只对 `.form-field` 生效；`.form-subgrid` 要整行需单独写 `.form-subgrid.full{grid-column:1/-1}`
- 需求文档 `doc/需求文档-Docker主机纳管v0.1.md` 第 5 章已精简为「页面说明 / 核心业务逻辑 / 操作验证规则」，**不要再往里加字段表**；用 Python 整段替换 Markdown 时终点用文件末尾，不要 `index('## N.')`（找不到会抛异常导致整次写入失败）

## 待办
- `sys_model_monitor.html` 实例分布按运行环境分组
- `doc/功能列表v0.2.md`、`doc/白皮书v0.2.md`（9.6 约束措辞）同步更新

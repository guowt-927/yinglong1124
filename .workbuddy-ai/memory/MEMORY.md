# 应龙平台原型 - 项目记忆

## 产品方向（2026-11 确定，11-25 四轮收敛后）
- 支持 K8s 集群 + Docker 主机**同时纳管**的混部形态；平台核心不感知运行时，统一 Runtime 接口 + K8s/Docker 适配器
- yl-agent：Go 单二进制、systemd 托管、挂载 docker.sock、gRPC 主动外拨 + mTLS
- **Docker 主机定位为「轻纳管」**：不做环境预检、不做 Agent 版本管理/升级、不做批量导入/升级、不暴露共享存储
- **Docker 主机面板为「多主机 · 左列表 + 右概览」**（11-25 下午用户明确要求保留多主机，推翻上午的单主机方案）：左侧主机列表（名称/IP/卡数/状态点）点击切换，右侧概览卡仅 名称/IP/状态 + 加速卡 已用/总数 + 运行实例；**不展示 CPU/内存/Agent，无主机详情页**；注册为**追加**并自动选中；移除只删选中那台，有运行实例时阻止；全部移除后空态 + 注册入口
- **Docker 服务生命周期极简**：启停/删除/编辑统一在服务列表操作（**运行中不可编辑**，仅停止后可编辑）；**不做蓝绿更新、不做在线扩缩容、不做跨主机重调度、不提供日志入口、不展示存量导入**
- Docker 部署时加速卡由平台按顺序自动分配，**不提供选卡网格**；保留「实例上限提示」
- 预警体系只覆盖 K8s 集群（节点异常 / SLA），**不含 Docker 主机类预警**
- 存量容器导入功能整体不在原型中展示（服务列表、集群页均无「存量导入」）
- 集群与主机页：「新建集群」(`newClusterModal`，仅 K8s 账号/配置文件两 Tab) 与「注册 Docker 主机」(`hostRegisterModal`，仅 IP/名称/SSH 端口/账号/密码 5 个字段，**无进度条、无 Agent 提示、无标签、无安装方式**) 是**两个独立弹窗**，顶部按钮 `openRegisterModal()` 按当前运行环境分流
- 工作台「历史预警」弹窗为纯列表：时间 / 级别 / 预警项 / 对象 / 详情 + 分页，**无筛选、无搜索、无持续时长/状态列**
- 工作台显卡资源表**不设「运行环境」列**（11-25），但保留顶部「运行环境」下拉作为筛选条件；服务列表的「运行环境」列保留
- 不做 DinD；跨运行时同一服务部署放二期

## 原型技术栈
- 纯 HTML + 原生 JS（内联 script），CSS 分三层：`yinglong-v2.css` / `prototype-v4.css` / `assets/css/pages/*.css`
- 侧栏由 `prototype-v4.js` 注入；部署弹窗由 `model-deploy-modal.js` 动态渲染
- **工作台页面是项目根目录 `index.html`**（`prototypes/index.html` 只是跳转壳）；工作台脚本在 `prototypes/assets/js/system-overview.js`
- 静态资源引用带 `?v=` 版本号，**改 JS/CSS 后必须递增**（当前 infra 为 `-11`，node-detail 为 `-7`，service/deploy-modal 为 `-5`，model-mgmt 为 `-6`，overview 为 `-6`）
- 行尾不统一：`sys_infrastructure_mgmt.html`、需求文档为 **CRLF**；根 `index.html`、`system-overview.js/.css` 为 **LF**。Python 写回前先统计 `\r\n` 数量再决定 `newline`

## 验证工具链
- `node --check` 校验 JS；Python 标签平衡脚本校验 HTML；表头 `<th>` 数与行模板 `<td>` 数比对
- **Node stub 运行时测试**：截取纯函数段注入 `vm` 上下文 + 手写 DOM 桩，可跑通渲染/筛选流程（注入的 `$` 会被代码段内的 `const $` 覆盖，`document` 桩须实现 `querySelector`）
- agent-browser / Edge headless 在本机不可用（msedge 进程占用），放弃浏览器截图，改用 DOM 校验
- Python 写含 `${...}` 的 JS 时必须用 raw string `r"""..."""`，否则 `\$` 转义出错

## 已知坑
- `prototype-v4.js` 会给 `nodeGpuUtilization` 打「V1不含」徽章（Docker 主机详情页已移除，此条仅对 K8s 节点详情有效）
- 工作台预警条 `.alert-strip` 需 flex-wrap，预警项用 `.alert-item` 包裹；按钮包在 `.alert-actions` 里（`margin-left:auto` 在容器上）
- 服务列表 `rowActionMarkup` 的静态 HTML 和 JS 模板要同步改，否则首屏和筛选后不一致
- `model-management.css` 第 483 行是压缩单行，正则删规则容易留残片，删后要 grep 确认
- `yinglong-v2.css` 的 `.full{grid-column:1/-1}` 只对 `.form-field` 生效；`.form-subgrid` 要整行需单独写 `.form-subgrid.full{grid-column:1/-1}`
- 需求文档 `doc/需求文档-Docker主机纳管v0.2.md`（11-25 从 v0.1 改名；**用户明确允许不受格式限制、以完善为准**，v0.2 已含字段表 / 状态机 / 异常表 / 非功能 / 验收 / 待确认）；用 Python 整段替换 Markdown 时终点用文件末尾，不要 `index('## N.')`（找不到会抛异常导致整次写入失败）
- 需求文档表格行被编辑器格式化过（对齐空格不一），精确匹配常失败 → 用 `startswith(编号前缀)` 逐行定位并断言唯一命中；§3 表格编号列有前导空格，正则要写 `^\|\s*(ID)\s*\|`
- 往 Markdown 表格插数据行时**锚定分隔行 `| --------- |` 之后**，不要锚定表头（会把数据行插到表头与分隔行之间，表格直接损坏）
- 校验 Markdown 时文件是 CRLF，必须 `split('\r\n')`；用 `split('\n')` 会留 `\r` 导致分隔行判定全假
- `sys_infrastructure_mgmt.html` 的 Docker 脚本段函数全是**单行压缩**，多行精确匹配必失败（CRLF + 行尾）→ 用 `split('\r\n')` 后**按行号区间整段替换**，替换前对首尾行做 `startswith` 断言；HTML 区块用起止标记截取
- `sys_infrastructure_mgmt.html` 的 `<section class="card cluster-overview">` K8s/Docker 各一个同结构，定位 Docker 面板要用其独有 id（`displayHostName`）限定范围

## 待办
- `sys_model_monitor.html` 实例分布按运行环境分组
- `doc/功能列表v0.2.md`、`doc/白皮书v0.2.md`（9.6 约束措辞）同步更新
- 需求文档 v0.2 §9 的 8 条待确认事项（Q-01~Q-08）需产品/研发对齐后回填
- 根目录 `修改.md` 是最早一版改动清单，多处已被后续收敛推翻（选卡网格、蓝绿更新、4 项预警等），**内容过时**，待用户决定是否更新或删除

# Nuclei Custom Templates

本仓库存放团队自定义的 Nuclei YAML 模板，用于**内网安全风险发现**，不是通用互联网漏洞库。

核心模式：**先指纹识别服务，再精准触发对应低风险验证模板**。所有模板分为 `safe`（只读、无告警）与 `optional`（有登录尝试、可能触发锁定）两类，默认仅运行 `safe`。

## 仓库结构

目录结构遵循 [projectdiscovery/nuclei-templates](https://github.com/projectdiscovery/nuclei-templates) 官方推荐规范：

```text
.
├── workflows/               # 工作流入口
│   ├── internal-full-safe.yaml     # 默认 safe 扫描
│   ├── web-services.yaml           # Web 服务 safe
│   ├── network-services.yaml       # 网络服务 safe
│   ├── databases.yaml              # 数据库专项（含 optional）
│   ├── devops.yaml                 # DevOps 专项（含 optional）
│   ├── middleware.yaml             # 中间件专项（含 optional）
│   ├── panels.yaml                 # 监控面板专项（含 optional）
│   ├── cloud-native.yaml           # 云原生专项（含 optional）
│   ├── cameras.yaml                # 摄像头专项（含 optional）
│   ├── network-devices.yaml        # 网络设备专项（含 optional）
│   ├── domestic-devices.yaml       # 国产设备专项（含 optional）
│   ├── domestic-software.yaml      # 国产中间件/数据库专项（含 optional）
│   └── ics.yaml                    # ICS/SCADA 专项（含 optional）
├── fingerprints/            # 指纹识别模板
│   ├── http/
│   │   ├── middleware/
│   │   ├── devops/
│   │   ├── panels/
│   │   ├── cameras/
│   │   ├── network-devices/
│   │   └── ics/
│   └── network/
│       └── ics/
├── http/                    # HTTP 协议风险模板
│   ├── exposures/           # 敏感暴露
│   ├── misconfig/           # 错误配置
│   ├── panels/              # 面板检测
│   ├── sensitive-files/     # 敏感文件
│   └── default-logins/      # 默认口令（optional）
├── network/                 # 网络协议模板
│   ├── anonymous-access/    # 匿名访问
│   ├── no-auth/             # 未授权访问
│   ├── misconfig/           # 错误配置
│   └── default-logins/      # 默认口令（optional）
├── javascript/              # JavaScript 协议模板
│   ├── default-logins/
│   ├── no-auth/
│   └── protocol-checks/
├── payloads/                # 极小字典
│   ├── users-mini.txt
│   ├── passwords-mini.txt
│   └── service-defaults/
├── metadata/                # 元数据
│   ├── service-map.yml      # 服务-模板映射
│   └── severity-policy.yml  # 严重等级策略
├── scripts/                 # 本地校验脚本
│   ├── validate.sh
│   └── lint-ids.sh
├── .github/workflows/       # CI
│   └── validate-nuclei-templates.yml
├── SCAN_POLICY.md           # 扫描安全政策
├── TEMPLATE_GUIDE.md        # 模板编写指南
├── CONTRIBUTING.md          # 贡献指南
└── README.md
```

## 设计原则

- **指纹先行**：每个服务族尽量具备指纹模板 + 风险模板 + workflow 串联。
- **极小字典**：弱口令模板严格限制为 2 用户名 × 2 密码，最多 4 次尝试。
- **命中即停**：所有 brute 模板均设置 `stop-at-first-match: true`、`threads: 1`。
- **safe / optional 分离**：
  - `safe`：只读、无登录失败计数、不触发服务端告警。可进入默认工作流。
  - `optional`：有锁定风险、有登录失败日志。仅在服务专项工作流中显式触发。
- **结果可审计**：建议所有扫描命令附加 `-o result.json -j`。
- **90% 复用官方**：优先复制/改造 [nuclei-templates](https://github.com/projectdiscovery/nuclei-templates) 官方模板，自研仅用于国产设备/摄像头/OA/门禁等官方未覆盖场景。

## 快速开始

### 推荐扫描命令

```bash
# 默认 safe 扫描（推荐首次使用）
nuclei -l targets.txt -t workflows/internal-full-safe.yaml -j -o results/safe.json

# Web 服务专项
nuclei -l web-targets.txt -t workflows/web-services.yaml -j -o results/web.json

# 网络服务专项
nuclei -l network-targets.txt -t workflows/network-services.yaml -j -o results/network.json

# DevOps 专项（含 optional 登录爆破，-rlm 限速）
nuclei -l devops-targets.txt -t workflows/devops.yaml -rlm 30 -j -o results/devops.json

# 中间件专项（含 optional 登录爆破，-rlm 限速）
nuclei -l middleware-targets.txt -t workflows/middleware.yaml -rlm 30 -j -o results/middleware.json

# 监控面板专项（含 optional 登录爆破，-rlm 限速）
nuclei -l panels-targets.txt -t workflows/panels.yaml -rlm 30 -j -o results/panels.json

# 数据库专项（含 optional 登录爆破，-rlm 限速）
nuclei -l db-targets.txt -t workflows/databases.yaml -rlm 30 -j -o results/databases.json

# 云原生专项（含 optional 登录爆破，-rlm 限速）
nuclei -l k8s-targets.txt -t workflows/cloud-native.yaml -rlm 30 -j -o results/cloud-native.json

# 摄像头专项（含 optional 登录爆破，-rlm 限速）
nuclei -l camera-targets.txt -t workflows/cameras.yaml -rlm 30 -j -o results/cameras.json

# 网络设备专项（含 optional 登录爆破，-rlm 限速）
nuclei -l device-targets.txt -t workflows/network-devices.yaml -rlm 30 -j -o results/network-devices.json

# 国产设备专项（含 optional 登录爆破，-rlm 限速）
nuclei -l domestic-device-targets.txt -t workflows/domestic-devices.yaml -rlm 30 -j -o results/domestic-devices.json

# 国产中间件/数据库专项（含 optional 登录爆破，-rlm 限速）
nuclei -l domestic-sw-targets.txt -t workflows/domestic-software.yaml -rlm 30 -j -o results/domestic-software.json

# ICS/SCADA 专项（含 optional 登录爆破，-c 1 -rlm 10 严格限速）
nuclei -l ics-targets.txt -t workflows/ics.yaml -c 1 -rlm 10 -j -o results/ics.json

# 验证全部模板语法
nuclei -validate -t .
```

### 单模板使用

```bash
# SSH 弱口令检测（optional，需显式指定）
nuclei -t ./javascript/default-logins/ssh-mini-brute.yaml -u <target>:22 -o ssh_weak.json -j

# FTP 弱口令检测（optional）
nuclei -t ./network/default-logins/ftp-mini-brute.yaml -u <target> -o ftp_weak.json -j

# Redis 未授权检测（safe）
nuclei -t ./network/no-auth/redis-no-auth.yaml -u <target>:6379 -o redis_unauth.json -j
```

## 模板清单

完整的服务-模板映射见 [`metadata/service-map.yml`](metadata/service-map.yml)。当前覆盖的主要服务：

| 服务族 | 指纹 | Safe 风险 | Optional 风险 |
|--------|------|-----------|---------------|
| 中间件 | Tomcat, Spring Boot, Nginx, Apache, WebLogic, JBoss, Kafka, RocketMQ | Actuator, Swagger, .git, Server Status, Directory Listing, Kafka Connect, JMX Console | Tomcat Manager Mini Brute |
| DevOps | Jenkins, GitLab, Nexus, Harbor | Swagger, Jolokia | Jenkins / Nexus / Harbor Mini Brute |
| 监控面板 | Grafana, Kibana, Prometheus, Zabbix, MinIO | Prometheus Metrics, Druid, phpinfo | Grafana / Zabbix / MinIO Mini Brute |
| 数据库 | Redis, MySQL, PostgreSQL, MongoDB, Elasticsearch, MSSQL, ZooKeeper, Memcached | 未授权、匿名访问 | MySQL / PostgreSQL / MSSQL / Redis ACL / phpMyAdmin Mini Brute |
| 网络协议 | SSH, FTP, SMB, Telnet, RDP, Rsync, SNMP | 匿名登录、Null Session、Public Community | SSH / FTP Mini Brute |
| 云原生 | — | Docker Registry, K8s API, Docker API, etcd, Consul | RabbitMQ / ActiveMQ Mini Brute |
| 摄像头 | 海康威视, 大华 | — | 海康 / 大华 Mini Brute |
| 网络设备 | H3C, 华为, 锐捷, 深信服, 天融信, 奇安信, 启明星辰, 绿盟 | — | H3C / 华为 / 锐捷 / 深信服 / 天融信 / 奇安信 / 启明星辰 / 绿盟 Mini Brute |
| 国产中间件 | 东方通 TongWeb, 宝兰德 BES, 金蝶 Apusic | — | TongWeb / BES / Apusic Mini Brute |
| 国产数据库 | 达梦 DM, 人大金仓 KingBase, 神通 OSCAR | — | KingBase Mini Brute |
| ICS 协议 | Modbus, IEC 104, S7comm, DNP3, CODESYS, MMS | — | — |
| ICS 设备 | Siemens, Schneider, ABB, Rockwell, GE, Omron | — | Siemens / Schneider / ABB / Rockwell / GE / Omron Mini Brute |
| Web 通用 | — | 监控暴露、phpinfo | — |

## 扫描政策

见 [`SCAN_POLICY.md`](SCAN_POLICY.md)。核心要求：

1. 默认工作流 `internal-full-safe.yaml` **绝不包含** login brute、CVE 验证、写入型探测、Interactsh 依赖模板。
2. 弱口令模板默认最多 **4 次尝试**，必须声明 `metadata.max-request`。
3. 禁止硬编码客户环境信息、真实密码、真实内网 IP、真实扫描结果。

## 账号锁定风险规避

爆破模板（`*-mini-brute`）采用三层防护设计，但仍需注意聚合风险。

### 模板内建防护

| 防护措施 | 说明 |
|---------|------|
| 最大尝试次数 | 4（2 用户 × 2 密码，pitchfork） |
| 命中即停 | `stop-at-first-match: true` |
| 单线程 | `threads: 1` |
| 请求 pacing | 模板无内置 delay，通过 runner 参数 `-rlm`/`-c` 控制 |

### 聚合风险

同一主机被多个模板命中时，总尝试次数 = 模板数 × 4。例如 6 个网络设备模板打同一台 H3C = 24 次登录尝试。nuclei 无模板间全局去重机制，需在 runner 层面控制。

### 运行级防护

```bash
# 普通环境：限制每分钟总请求数
nuclei -l targets.txt -t workflows/network-devices.yaml -rlm 30 -j -o results/network-devices.json

# ICS 环境：单独运行，单线程，低速率
nuclei -l ics-targets.txt -t workflows/ics.yaml -c 1 -rlm 10 -j -o results/ics.json

# 数据库/DevOps 等含 optional 的 workflow，建议加 -rlm
nuclei -l targets.txt -t workflows/databases.yaml -rlm 30 -j -o results/databases.json
```

| 场景 | 建议参数 | 说明 |
|------|---------|------|
| 普通服务 optional workflow | `-rlm 30` | 每分钟最多 30 个请求 |
| ICS/SCADA 专项 | `-c 1 -rlm 10` | 单线程 + 每分钟 10 请求 |
| 网络设备专项（多模板命中同一主机） | `-rlm 20` | 降低速率防聚合 |
| 仅 safe 模板 | 无需限制 | safe 模板无登录尝试 |

### 注意事项

- **ICS/SCADA 设备务必单独运行**，不与普通模板混跑。工控设备对认证请求更敏感，即使 4 次也可能导致设备异常或服务中断。
- **确认目标锁定策略**：如已知目标账号锁定阈值（如 5 次/10 分钟），应据此调整 `-rlm` 或跳过爆破。
- **safe workflow 无风险**：`internal-full-safe.yaml` 不包含任何 `optional` 模板，可放心使用。

## 如何新增模板

1. 阅读 [`TEMPLATE_GUIDE.md`](TEMPLATE_GUIDE.md)。
2. 在对应协议目录下创建 `.yaml` 文件。
3. 本地运行 `nuclei -validate -t <文件>` 通过。
4. 若引用 `payloads/` 字典，实测 `nuclei -t .` 与 `nuclei -t workflows/x.yaml` 两种入口路径解析正确。
5. 更新 `metadata/service-map.yml`。
6. 提交 PR。

## 本地校验

```bash
# 验证全部模板语法与 schema
bash scripts/validate.sh

# 检查 id 唯一性、弱口令字段、旧目录残留
bash scripts/lint-ids.sh
```

## CI

GitHub Actions 在每次 push/PR 时自动执行：

1. `nuclei -validate -t .`
2. `scripts/lint-ids.sh`

CI 范围：**YAML 语法、schema 合法、id 唯一性、弱口令必备字段**。检测逻辑正确性不在 CI 范围，需人工 review 或 lab 环境验证。

## 参考

- [Nuclei Templates 官方文档](https://docs.projectdiscovery.io/tools/nuclei/overview)
- [Nuclei Templates 官方仓库](https://github.com/projectdiscovery/nuclei-templates)
- [`handbook/内网渗透测试手册_v2.md`](https://github.com/RBKD-SEC/Pentest-Playbook/blob/main/handbook/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F%E6%B5%8B%E8%AF%95%E6%89%8B%E5%86%8C_v2.md)

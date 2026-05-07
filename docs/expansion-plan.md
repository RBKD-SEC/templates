# 内网高危服务扩展计划

基于当前仓库已覆盖的 28 指纹 + 16 safe + 9 optional，按内网出现频率、风险等级和实现成本排序，分 5 批补齐常见高危服务。

---

## 第一批：数据库默认登录（4 模板）

已覆盖指纹和 no-auth，但缺少**有认证时的默认口令**场景。官方 nuclei-templates 对 MySQL/PostgreSQL/MSSQL/Redis 均有协议级登录模板，可直接改造收敛。

| 模板 | 协议 | 类型 | 严重等级 | 说明 |
|------|------|------|----------|------|
| `network/default-logins/mysql-mini-brute.yaml` | network | optional | high | TCP 3306，2×2 字典 |
| `network/default-logins/postgres-mini-brute.yaml` | network | optional | high | TCP 5432，2×2 字典 |
| `network/default-logins/mssql-mini-brute.yaml` | network | optional | high | TCP 1433，TDS 登录，2×2 字典 |
| `network/default-logins/redis-acl-mini-brute.yaml` | network | optional | high | TCP 6379，AUTH 命令，2×2 字典 |

**同步更新**：`service-map.yml`（mysql/postgres/mssql/redis 的 `checks.optional`）+ `databases.yaml` workflow。

---

## 第二批：云原生与容器（7 模板）

内网云原生环境一旦出现未授权或默认口令，通常直接导致集群接管，风险极高。

### 2.1 Safe（只读探测）

| 模板 | 协议 | 类型 | 严重等级 | 说明 |
|------|------|------|----------|------|
| `http/exposures/docker-registry-exposed.yaml` | http | safe | medium | Docker Registry V2 `/v2/_catalog` 未授权枚举镜像 |
| `http/exposures/kubernetes-api-unauth.yaml` | http | safe | high | K8s API `/api/v1/namespaces` 未授权访问 |
| `http/exposures/docker-api-unauth.yaml` | http | safe | high | Docker Remote API `/containers/json` 未授权 |
| `http/exposures/etcd-unauth.yaml` | http | safe | critical | etcd `/v2/keys` 或 `/v3/auth/users` 未授权，可读全部密钥 |
| `http/exposures/consul-unauth.yaml` | http | safe | high | Consul `/v1/agent/services` 未授权 |

### 2.2 Optional（默认口令）

| 模板 | 协议 | 类型 | 严重等级 | 说明 |
|------|------|------|----------|------|
| `http/default-logins/rabbitmq-mini-brute.yaml` | http | optional | high | RabbitMQ Management `/api/whoami` 默认口令 |
| `http/default-logins/activemq-mini-brute.yaml` | http | optional | high | ActiveMQ Console 默认口令 |

**同步更新**：新增 `workflows/cloud-native.yaml`（safe + optional）+ `service-map.yml`。

---

## 第三批：消息队列与大数据中间件（5 模板）

| 模板 | 协议 | 类型 | 严重等级 | 说明 |
|------|------|------|----------|------|
| `fingerprints/http/middleware/kafka-detect.yaml` | http | fingerprint | info | Kafka Connect / Kafka Manager 面板识别 |
| `http/exposures/kafka-connect-exposed.yaml` | http | safe | medium | Kafka Connect REST API 暴露 |
| `fingerprints/http/middleware/rocketmq-detect.yaml` | http | fingerprint | info | RocketMQ Console 识别 |
| `http/exposures/rocketmq-console-exposed.yaml` | http | safe | medium | RocketMQ Console 未授权访问 |
| `network/no-auth/kafka-unauth.yaml` | network | safe | medium | Kafka ACL 未启用时的 topic 列表枚举 |

---

## 第四批：国产设备与网络设备（6 模板）

官方 nuclei-templates 覆盖不足，需**自研指纹 + 默认口令**。内网资产盘点中占比极高。

| 模板 | 协议 | 类型 | 严重等级 | 说明 |
|------|------|------|----------|------|
| `fingerprints/http/cameras/hikvision-detect.yaml` | http | fingerprint | info | 海康威视登录页识别（`/doc/page/login.asp`） |
| `http/default-logins/hikvision-mini-brute.yaml` | http | optional | high | 海康 2×2 默认口令（admin/12345 等） |
| `fingerprints/http/cameras/dahua-detect.yaml` | http | fingerprint | info | 大华登录页识别 |
| `http/default-logins/dahua-mini-brute.yaml` | http | optional | high | 大华 2×2 默认口令 |
| `fingerprints/http/network-devices/h3c-detect.yaml` | http | fingerprint | info | H3C/Comware 设备 Web 识别 |
| `http/default-logins/h3c-mini-brute.yaml` | http | optional | high | H3C 设备 2×2 默认口令 |

> 华为、锐捷、思科等品牌可后续按同样模式追加指纹 + optional brute。

**同步更新**：`workflows/network-devices.yaml` + `workflows/cameras.yaml` + `service-map.yml`。

---

## 第五批：Web 中间件管理面板与应用（8 模板）

内网常见的 Java 中间件、运维平台和存储管理面板。

### 5.1 Safe

| 模板 | 协议 | 类型 | 严重等级 | 说明 |
|------|------|------|----------|------|
| `fingerprints/http/middleware/weblogic-detect.yaml` | http | fingerprint | info | WebLogic Console `/console` 识别 |
| `http/exposures/weblogic-console-exposed.yaml` | http | safe | medium | WebLogic Console 未授权访问检测 |
| `fingerprints/http/middleware/jboss-detect.yaml` | http | fingerprint | info | JBoss/JMX Console 识别 |
| `http/exposures/jboss-jmx-console-exposed.yaml` | http | safe | medium | JBoss JMX Console 未授权 |
| `fingerprints/http/panels/zabbix-detect.yaml` | http | fingerprint | info | Zabbix 面板识别 |
| `fingerprints/http/panels/minio-detect.yaml` | http | fingerprint | info | MinIO Console `/minio/login` 识别 |

### 5.2 Optional

| 模板 | 协议 | 类型 | 严重等级 | 说明 |
|------|------|------|----------|------|
| `http/default-logins/weblogic-mini-brute.yaml` | http | optional | high | WebLogic Console 2×2 默认口令 |
| `http/default-logins/zabbix-mini-brute.yaml` | http | optional | high | Zabbix Admin 2×2 默认口令 |
| `http/default-logins/minio-mini-brute.yaml` | http | optional | high | MinIO 2×2 默认口令（minioadmin/minioadmin 等） |

**同步更新**：`workflows/middleware.yaml`（追加 WebLogic/JBoss）+ `workflows/panels.yaml`（追加 Zabbix/MinIO）+ `service-map.yml`。

---

## 扩展后仓库全景

| 分类 | 当前 | 扩展后 | 增幅 |
|------|------|--------|------|
| 指纹 | 28 | 47 | +19 |
| Safe 风险 | 16 | 30 | +14 |
| Optional 风险 | 9 | 20 | +11 |
| Workflow | 6 | 9 | +3 |
| 服务族 | 25 | 40+ | +15+ |

---

## 实施建议

1. **第一批优先**：数据库默认登录是内网渗透测试中最直接的突破口，ROI 最高。
2. **第二批次之**：云原生环境一旦命中通常是"单点突破整个集群"，但需确认目标环境实际部署比例。
3. **第四批需实测**：国产设备指纹依赖实际页面特征，建议在有真实设备的 lab 环境验证后再合并。
4. **每批独立 PR**：按此计划每批一个 PR，保持 review 粒度可控。
5. **全部 safe 模板优先落地**：同一服务族先完成 safe 模板和指纹，optional 可在下一批或同一批末尾补充。

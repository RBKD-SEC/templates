# Nuclei 自定义模板

本目录存放团队自定义的 Nuclei YAML 模板，用于内网渗透测试中的快速验证。

## 设计原则

- **极小字典**：弱口令模板严格限制为 2 用户名 × 2 密码，最多 4 次尝试
- **命中即停**：所有 brute 模板均设置 `stop-at-first-match: true`
- **结果可审计**：建议所有扫描命令附加 `-o result.json -j`

## 模板列表

| 模板文件 | 用途 | 目标服务 |
|----------|------|----------|
| `ssh-mini-brute.yaml` | SSH 弱口令检测 | SSH (22) |
| `ftp-mini-brute.yaml` | FTP 弱口令检测 | FTP (21) |
| `network-device-mini-brute.yaml` | 网络设备 HTTP 弱口令 | HTTP (80/443) |

## 使用方法

```bash
# SSH 弱口令检测
nuclei -t ./templates/nuclei/ssh-mini-brute.yaml -u <target>:22 -o ssh_weak.json -j

# FTP 弱口令检测
nuclei -t ./templates/nuclei/ftp-mini-brute.yaml -u <target> -o ftp_weak.json -j

# 网络设备弱口令检测
nuclei -t ./templates/nuclei/network-device-mini-brute.yaml -u http://<target> -o device_weak.json -j
```

> 注：本目录模板配合 [`handbook/内网渗透测试手册_v2.md`](https://github.com/RBKD-SEC/Pentest-Playbook/blob/main/handbook/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F%E6%B5%8B%E8%AF%95%E6%89%8B%E5%86%8C_v2.md) 使用。

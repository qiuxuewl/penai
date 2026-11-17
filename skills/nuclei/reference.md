# Nuclei 参考手册

## 常用命令示例

### 按 tag 扫描（推荐日常用法）
```bash
# 扫 nginx 相关已知漏洞
nuclei -u https://target.example.com \
  -tags nginx,cve \
  -json-export /tmp/nuclei_nginx.json \
  -no-interactsh -rate-limit 20 -timeout 10 -silent

# 扫 Spring Boot / Spring Framework
nuclei -u https://target.example.com \
  -tags spring,springboot,cve \
  -json-export /tmp/nuclei_spring.json \
  -no-interactsh -rate-limit 20 -timeout 10 -silent

# 扫 WordPress
nuclei -u https://target.example.com \
  -tags wordpress,wp-plugin,cve \
  -json-export /tmp/nuclei_wp.json \
  -no-interactsh -rate-limit 20 -timeout 10 -silent
```

### 按 CVE 号精准验证
```bash
# 验证单个 CVE
nuclei -u https://target.example.com \
  -id CVE-2021-44228 \
  -json-export /tmp/nuclei_log4shell.json \
  -no-interactsh -silent

# 同时验证多个 CVE
nuclei -u https://target.example.com \
  -id CVE-2021-44228,CVE-2022-22965 \
  -json-export /tmp/nuclei_cves.json \
  -no-interactsh -silent
```

### 仅做技术指纹识别（低噪音）
```bash
nuclei -u https://target.example.com \
  -tags tech \
  -json-export /tmp/nuclei_tech.json \
  -no-interactsh -silent
```

### 读取 JSON 结果（命令行快速查看）
```bash
cat /tmp/nuclei_out.json | python3 -c "
import sys, json
for line in sys.stdin:
    r = json.loads(line)
    sev = r['info']['severity']
    if sev in ('critical','high','medium'):
        print(f\"[{sev.upper()}] {r['info']['name']} @ {r['matched-at']}\")
"
```

---

## 结果 JSON 字段说明

| 字段 | 含义 |
|------|------|
| `template-id` | 模板 ID，通常是 CVE 号或描述性 slug |
| `info.name` | 漏洞/检测项名称 |
| `info.severity` | 严重级别：`critical` / `high` / `medium` / `low` / `info` |
| `info.description` | 漏洞描述 |
| `info.reference` | 参考链接（CVE、Advisory） |
| `matched-at` | 实际触发的完整 URL |
| `request` | 发出的原始 HTTP 请求 |
| `response` | 收到的原始 HTTP 响应 |
| `curl-command` | 等效 curl 命令，可直接复现 |
| `extracted-results` | 提取到的动态内容（如 OOB 回调 token、版本字符串） |
| `host` | 目标主机 |
| `timestamp` | 触发时间 |

---

## 推荐参数说明

| 参数 | 值 | 说明 |
|------|----|------|
| `-no-interactsh` | 无需参数值 | 禁用 OOB 交互（OAST）检测，避免把流量发到外部服务器；在无法出网或不希望 OOB 的场景必加 |
| `-rate-limit` | `10`–`50` | 每秒最大请求数。生产目标建议 10-20，实验/沙盒可以更高 |
| `-timeout` | `10` | 单个请求超时秒数，默认 10 |
| `-silent` | 无需参数值 | 只输出结果，不输出 banner/进度条，便于脚本解析 |
| `-json-export` | 文件路径 | 把每条结果以 JSONL 格式写入文件（每行一个 JSON 对象）；比 `-json` 到 stdout 更易解析 |
| `-retries` | `1`–`3` | 失败重试次数，默认 1 |
| `-bulk-size` | `25` | 并发模板数，降低时可减少目标压力 |
| `-concurrency` | `10` | 并发请求数 |

---

## 常见组件 → Nuclei tag 速查表

| 组件 / 技术栈 | 推荐 tag |
|--------------|---------|
| Nginx | `nginx`, `cve` |
| Apache HTTP Server | `apache`, `httpd`, `cve` |
| Apache Tomcat | `tomcat`, `apache`, `cve` |
| Spring Boot / Spring MVC | `springboot`, `spring`, `java`, `cve` |
| Spring Cloud / Netflix Zuul | `spring-cloud`, `spring`, `cve` |
| WordPress | `wordpress`, `wp-plugin`, `wp-theme`, `cve` |
| Drupal | `drupal`, `cve` |
| Joomla | `joomla`, `cve` |
| phpMyAdmin | `phpmyadmin`, `cve` |
| Jenkins | `jenkins`, `cve` |
| GitLab | `gitlab`, `cve` |
| Jira / Confluence | `jira`, `confluence`, `atlassian`, `cve` |
| Log4j（Log4Shell） | `log4j`, `cve` 或 `-id CVE-2021-44228` |
| Elasticsearch | `elasticsearch`, `cve` |
| Redis | `redis`, `cve` |
| Apache Solr | `solr`, `cve` |
| Hadoop / Yarn | `hadoop`, `cve` |
| Node.js / Express | `nodejs`, `express`, `cve` |
| Ruby on Rails | `rails`, `ruby`, `cve` |
| Laravel / PHP | `laravel`, `php`, `cve` |
| Microsoft Exchange | `exchange`, `microsoft`, `cve` |
| Cisco 设备 | `cisco`, `network`, `cve` |
| F5 BIG-IP | `bigip`, `f5`, `cve` |
| Fortinet | `fortinet`, `cve` |
| Palo Alto | `paloalto`, `network`, `cve` |
| VMware vCenter | `vmware`, `vcenter`, `cve` |
| MinIO | `minio`, `cve` |
| Grafana | `grafana`, `cve` |

---

## 高频 CVE 快速参考（近年高危）

| CVE | 组件 | 类型 |
|-----|------|------|
| CVE-2021-44228 | Log4j 2.x | RCE（Log4Shell） |
| CVE-2022-22965 | Spring Framework 5.3.x | RCE（Spring4Shell） |
| CVE-2022-22963 | Spring Cloud Function | RCE |
| CVE-2021-26855 | MS Exchange | SSRF + RCE（ProxyLogon） |
| CVE-2021-34527 | Windows Print Spooler | RCE（PrintNightmare） |
| CVE-2022-26134 | Confluence Server | RCE |
| CVE-2022-1388 | F5 BIG-IP iControl | RCE |
| CVE-2023-46805 | Ivanti Connect Secure | Auth Bypass |
| CVE-2024-21762 | FortiOS SSL-VPN | RCE |
| CVE-2021-21985 | VMware vCenter | RCE |

---

## 注意事项

1. **更新模板库**：运行前先 `nuclei -update-templates` 确保用最新模板。
2. **OOB 漏洞**：Log4Shell 等需要 OOB 回调的漏洞，不加 `-no-interactsh` 才能检测；加了则跳过。视授权范围决定是否启用。
3. **误报处理**：nuclei 对某些漏洞只做被动检测（版本匹配），需人工确认是否真实可利用。
4. **速率限制**：云 WAF 可能对高速扫描触发封锁，`-rate-limit 10` 更安全。

---
name: nuclei-scan
description: 识别出目标组件版本后，用 Nuclei 模板库对已知漏洞做精准验证
trigger: 识别到具体组件及版本号（如 nginx 1.18.0、Spring Boot 2.5.x、WordPress 5.8）时调用
tags: [nuclei, cve, fingerprint, exploit]
---

# Nuclei 扫描技能

## 什么时候调用

当你完成指纹识别、确认了目标组件及其版本号之后，立即调用本技能。典型触发场景：

- HTTP 响应头 / 页面源码泄露了 `Server`、`X-Powered-By`、`Generator` 等版本信息
- 目录遍历或错误页面暴露了框架/中间件版本
- `/actuator`、`/wp-login.php`、`/phpmyadmin/` 等路径确认了具体应用
- JS 资源文件中发现 `version: "x.y.z"` 等版本字符串

**不要** 在未识别到具体组件版本时泛泛调用。

---

## 执行流程

### 1. 指纹确认
在运行 nuclei 前，确认以下信息已记录为资产（asset）：
- 目标 URL / IP
- 组件名称及版本号（props 中标注）
- 服务类型（web / ssh / ftp / …）

### 2. 按组件选择模板 tag
参考 `reference.md` 中的「组件→tag 速查表」，选择最精准的 tag 组合。  
优先用 `cve` + 组件专属 tag（如 `nginx`），避免使用 `fuzzer` / `dos` 等破坏性 tag。

### 3. 运行 nuclei
使用 JSON 输出格式，加安全参数（见 `reference.md`）：

```bash
nuclei -u <target_url> \
  -tags <tag1,tag2> \
  -json-export /tmp/nuclei_out.json \
  -no-interactsh \
  -rate-limit 20 \
  -timeout 10 \
  -silent
```

若需按 CVE 号精准验证：
```bash
nuclei -u <target_url> -id <CVE-ID> -json-export /tmp/nuclei_out.json -no-interactsh -silent
```

### 4. 解析结果
读取 `/tmp/nuclei_out.json`，按以下字段提取关键信息：
- `info.severity`：严重级别（critical / high / medium / low / info）
- `info.name`：漏洞名称
- `info.description`：漏洞描述
- `matched-at`：实际触发的 URL
- `request` / `response`：请求/响应证据（用于 PoC）
- `curl-command`：可直接复现的 curl 命令

### 5. 记录结果
| 严重级别 | 处理方式 |
|---------|---------|
| critical / high | `report_finding`（含 curl-command 或 request/response 作为 PoC 证据） |
| medium | `record_fact`（记录为待验证的可疑发现，标注 `inferred`） |
| low / info | 视上下文决定，通常仅记录为资产 props 补充 |

**重要**：nuclei 命中≠漏洞已确认。对 critical/high 结果，务必用 curl 或其他方式独立复现一次，再调用 `report_finding`。

---

## 禁止行为

- **禁止全量扫描**：不得使用 `-t nuclei-templates/` 或不加 `-tags`/`-id` 的全量扫描，模板数量过多会产生大量噪音和不必要的流量
- **禁止在未授权目标运行**：只在当前任务已定义的 scope 范围内的目标上执行；若当前目标不在 scope 内，直接跳过本 skill，不中断任务流程
- **禁止破坏性 tag**：不使用 `dos`、`fuzz`、`brute-force` 等可能造成服务中断的 tag
- **禁止跳过独立验证**：不得仅凭 nuclei 的命中结果就直接 `report_finding`，critical/high 必须独立复现
- **禁止在无版本信息时调用**：没有具体版本号就没有精准 tag 可用，调用只会产生低价值噪音

---

## 结果处理示例

```json
{
  "template-id": "CVE-2021-44228",
  "info": {
    "name": "Apache Log4j2 RCE (Log4Shell)",
    "severity": "critical"
  },
  "matched-at": "https://target.example.com/api/login",
  "curl-command": "curl -H 'X-Api-Version: ${jndi:ldap://...}' https://target.example.com/api/login"
}
```

→ 调用 `report_finding`，severity=critical，evidence 中贴入 curl-command 和响应截图。

---
name: nuclei-scan
description: 识别出目标组件版本后，用 Nuclei 模板库对已知漏洞做精准验证
trigger: 识别到具体组件及版本号（如 nginx 1.18.0、Spring Boot 2.5.x、WordPress 5.8）时调用
tags: [nuclei, cve, fingerprint, exploit]
---

# Nuclei 扫描技能

Nuclei 是一个基于模板的漏洞扫描工具，模板库每日更新，覆盖数千个已知 CVE 和常见配置缺陷。

## 基本调用

```bash
# 按组件 tag 扫描
nuclei -u <target_url> -tags <tag> -json-export /tmp/out.json -no-interactsh -silent

# 按 CVE 号精准验证
nuclei -u <target_url> -id <CVE-ID> -json-export /tmp/out.json -no-interactsh -silent

# 技术指纹识别
nuclei -u <target_url> -tags tech -json-export /tmp/out.json -silent
```

组件→tag 映射及更多命令示例见 `reference.md`。

## 输出格式（JSONL，每行一条）

关键字段：`template-id`、`info.severity`（critical/high/medium/low/info）、`info.name`、`matched-at`、`request`、`response`、`curl-command`。

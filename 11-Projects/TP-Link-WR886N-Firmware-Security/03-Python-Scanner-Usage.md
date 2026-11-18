# Python 固件敏感信息扫描器及使用说明

## 1. 工具简介

本工具用于扫描 Binwalk 提取后的固件目录，通过正则表达式匹配敏感字符串，并尝试解析 PEM 私钥和 X.509 证书。

开发这个工具的原因是：在第一次分析 TP-Link TL-WR886N 固件时，我主要使用 strings、grep 等命令进行人工检索。虽然能够找到敏感信息，但需要重复执行命令，分析结果也不方便统一保存。

因此使用 Python 将目录遍历、特征匹配、密钥验证和报告生成整合到一个程序中。

当前版本：V1.1。

## 2. 功能说明

### 2.1 敏感字符串检测

当前支持六类特征：

| 规则 | 匹配内容 |
|---|---|
| PASSWORD | password、passwd、webPwd、fac_password |
| SECRET | secret |
| API_KEY | api_key、apikey、access_key |
| TOKEN | access_token、auth_token、refresh_token、bearer_token |
| SSH_KEY | OpenSSH 私钥标记及常见 SSH 公钥格式 |
| PRIVATE_KEY | RSA、EC、OpenSSH 等 PEM 私钥标记 |

扫描器首先提取连续可打印 ASCII 字符串，再使用正则表达式匹配。

关键词命中仅作为敏感信息候选，不直接认定为有效凭证或漏洞。

### 2.2 PEM 私钥分析

支持识别以下格式：

- RSA PRIVATE KEY
- PRIVATE KEY
- EC PRIVATE KEY
- OPENSSH PRIVATE KEY

识别到标记后，程序查找对应结束标记，提取完整 PEM 内容，并调用 cryptography 进行解析。

解析成功后进一步判断密钥类型。

### 2.3 RSA 密钥分析

对于成功解析的 RSA 私钥，提取：

- RSA 密钥长度
- 对应公钥
- 公钥 DER 编码
- SHA-256 公钥指纹
- 重复密钥情况

将低于 2048 bit 的密钥标记为 WEAK_CANDIDATE。

该阈值用于初步筛查，不代表对密钥实际可利用性的完整判断。

### 2.4 X.509 证书分析

支持识别 PEM 格式 X.509 证书，并提取：

- Subject
- Issuer
- Serial Number
- Not Before
- Not After
- 当前有效期状态
- 公钥类型及长度
- 签名算法
- SHA-256 证书指纹

当前没有实现完整证书链验证、吊销检查及设备运行时部署验证。

### 2.5 扫描报告

运行后生成：

| 文件 | 用途 |
|---|---|
| scan_report.json | 完整结构化扫描结果 |
| scan_report.md | 人工阅读的扫描报告 |
| scan_report.csv | 表格筛选和结果复核 |

## 3. 环境准备

使用 Kali Linux。

安装 Python 依赖：

```bash
sudo apt update
sudo apt install python3 python3-pip
```

安装 cryptography：

```bash
python3 -m pip install cryptography
```

检查版本：

```bash
python3 - <<'PY'
import cryptography
from cryptography import x509

print("Cryptography:", cryptography.__version__)
print("X.509 module: OK")
PY
```

本次实验使用 cryptography 46.0.6。

## 4. 项目目录

```text
firmware/
├── wr886ngv2.bin
├── _wr886ngv2.bin.extracted/
│   ├── A200
│   ├── C4500
│   ├── C3240
│   └── ...
│
└── firmware-scanner/
    ├── scanner.py
    ├── scanner_v1.0_backup.py
    └── reports/
        ├── scan_report.json
        ├── scan_report.md
        └── scan_report.csv
```

当前版本中的固件目录通过 `firmware_dir` 配置。

如果更换固件，需要修改该路径，使其指向对应的提取目录。

## 5. 运行方法

进入工具目录：

```bash
cd ~/firmware/firmware-scanner
```

先进行语法检查：

```bash
python3 -m py_compile scanner.py
```

执行扫描：

```bash
python3 scanner.py
```

扫描完成后检查报告：

```bash
ls -lh reports/
```

验证 JSON 格式：

```bash
python3 -m json.tool reports/scan_report.json > /dev/null
echo $?
```

输出 0 表示 JSON 文件能够正常解析。

## 6. 本次实验结果

使用 V1.1 扫描 TP-Link TL-WR886N 固件提取目录。

### 6.1 文件扫描统计

| 项目 | 结果 |
|---|---:|
| 遍历文件 | 134 |
| 成功读取 | 134 |
| 读取错误 | 0 |

### 6.2 敏感字符串匹配

| 类别 | 命中次数 |
|---|---:|
| PASSWORD | 107 |
| SECRET | 4 |
| API_KEY | 0 |
| TOKEN | 0 |
| SSH_KEY | 0 |
| PRIVATE_KEY | 2 |

这里的次数是匹配记录数量，不代表实际密码或漏洞数量。

### 6.3 PEM 私钥分析

| 项目 | 结果 |
|---|---:|
| PEM 标记 | 2 |
| 成功解析 | 1 |
| 未验证候选 | 1 |

其中：

- A200：存在私钥标记，但内容为空，无法解析。
- C4500：成功解析出 RSA 私钥。

### 6.4 RSA 分析

| 项目 | 结果 |
|---|---:|
| 有效 RSA 私钥 | 1 |
| 唯一 RSA 公钥指纹 | 1 |
| C4500 密钥长度 | 512 bit |

### 6.5 X.509 证书分析

| 文件 | 状态 | 公钥长度 | 有效期 |
|---|---|---:|---|
| C3240 | PARSED_VALID | 2048 bit | CURRENTLY_VALID |
| C46C3 | PARSED_VALID | 512 bit | EXPIRED |
| A200 | PARSE_FAILED | - | - |

C3240 和 C46C3 均成功解析。A200 中的证书标记为空内容，解析失败。

## 7. 代码模块说明

| 函数 | 作用 |
|---|---|
| analyze_private_key() | 解析 PEM 私钥，分析 RSA 类型、长度及指纹 |
| scan_pem() | 定位并提取 PEM 私钥 |
| scan_certificates() | 定位和解析 X.509 证书 |
| scan_strings() | 匹配敏感字符串 |
| generate_markdown() | 生成 Markdown 报告 |
| generate_csv() | 生成 CSV 报告 |
| main() | 遍历目录、调用扫描模块并汇总结果 |

程序使用 `Path.rglob()` 递归遍历提取目录中的普通文件。

对每个成功读取的文件，分别调用字符串、PEM 和证书扫描模块。

## 8. 实际开发中遇到的问题

### 8.1 证书全部解析失败

最初 V1.1 运行时，C3240 和 C46C3 均被标记为 PARSE_FAILED。

后来通过单独提取证书内容，发现证书本身可以被正确解析。

进一步检查 `scan_certificates()` 函数，发现问题出在证书指纹计算：

错误写法：

```python
cert.fingerprint(hashlib.sha256())
```

正确写法：

```python
cert.fingerprint(hashes.SHA256())
```

原因是 cryptography 的 fingerprint 接口需要该库自己的 HashAlgorithm 对象。

修正后重新运行，成功解析两份证书。

### 8.2 A200 被错误计入有效证书候选

A200 中存在 BEGIN 和 END 标记，但标记之间没有实际证书内容。

程序尝试解析后失败。

最终将其保留为解析失败候选，而不是有效证书。

### 8.3 私钥标记与有效私钥的区别

最初使用 grep 搜索到 PEM 标记时，容易直接认为存在完整密钥。

后来通过打印标记附近的原始字节，以及 OpenSSL 和 cryptography 验证，确认 A200 和 C4500 的内容不同。

这也是后来增加密码学解析模块的原因。

## 9. 当前功能限制

1. 当前主要提取可打印 ASCII 字符串，不覆盖所有字符编码。
2. 不会自动解压所有嵌套压缩包。
3. 对 `.7z` 扩展名文件直接跳过。
4. 正则匹配不能证明凭证有效。
5. 密钥解析成功不能证明密钥正在运行时使用。
6. 证书解析不等于完整证书链验证。
7. 尚未建立独立测试集，不能给出准确误报率和漏报率。
8. 当前扫描结果只来自一个固件提取目录，尚未完成多固件版本对比测试。

## 10. 后续改进方向

后续改进以提高检测质量为主，不盲目增加功能：

- 增加阳性和阴性测试样本，评估误报与漏报。
- 改进同一字符串中重复命中的定位能力。
- 补充 UTF-16 等常见编码识别。
- 完善嵌套压缩文件处理。
- 增加多固件版本扫描和结果对比。

## 11. 总结

通过开发这个工具，我将原本依赖 strings、grep 的人工敏感信息检索过程整合为 Python 自动化扫描流程。

目前工具已经能够递归扫描固件提取目录，识别多类敏感字符串，解析 RSA 私钥和 X.509 证书，并生成 JSON、Markdown、CSV 三种报告。

它的定位是固件静态安全分析中的辅助初筛工具，最终安全结论仍然需要结合人工复核、密码学验证和必要的动态测试。
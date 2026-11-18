# TP-Link TL-WR886N V2.0 固件静态分析记录

## 1. 实验环境与样本

### 1.1 实验环境

| 项目 | 配置 |
|---|---|
| 操作系统 | Kali Linux |
| Binwalk | 2.4.3 |
| 主要工具 | file、strings、grep、OpenSSL、Radare2 |
| Python | Python 3 |
| 密码学库 | cryptography 46.0.6 |

### 1.2 固件样本

- 设备：TP-Link TL-WR886N 千兆版 V2.0
- 文件：wr886ngv2.bin
- MD5：bf2c9226a119045028f3b71c60ddd1bd
- 文件大小：约 1.28 MB
- 固件日期：2017-05-13

## 2. 第一阶段：固件获取与完整性检查

最初计划使用 OWASP IoTGoat 作为分析对象，但当时 GitHub 下载不顺利，无法正常获取实验环境。

后来改为下载 TP-Link 官方提供的 TL-WR886N V2.0 固件。

过程中遇到的问题：

| 问题 | 处理方式 |
|---|---|
| TP-Link 下载中心返回 403 | 使用 wget 添加浏览器 User-Agent 和 Referer |
| 压缩包中文目录名乱码 | 使用 find 定位 .bin 文件，避免手动输入乱码路径 |
| Binwalk 2.4.3 与 Capstone 5.x 存在架构常量兼容问题 | 检查并修正本地 disasm.py 中的架构常量 |

样本下载后记录 MD5，后续分析均以该样本为基础。

常用命令：

```bash
md5sum wr886ngv2.bin
sha256sum wr886ngv2.bin
file wr886ngv2.bin
```

这一阶段主要学到的是：分析前先固定样本，保留哈希值，避免后续误用不同固件文件。

## 3. 第二阶段：Binwalk 固件识别与提取

### 3.1 固件识别

首先使用：

```bash
binwalk wr886ngv2.bin
```

查看固件内部的签名、偏移及可能存在的文件系统和组件。

重点是先理解固件的大致结构，而不是一看到 Linux、uImage 等字符串就认定主系统是 Linux。

### 3.2 固件提取

执行：

```bash
binwalk -e wr886ngv2.bin
```

提取后进入对应目录：

```bash
cd ~/firmware/_wr886ngv2.bin.extracted
```

查看提取结果：

```bash
find . -type f
```

后续对提取出的 134 个文件进行审计。

### 3.3 遇到的问题：标准 squashfs 解包失败

最初发现根文件系统涉及 TP-Link 定制 LZMA 压缩内容，直接使用标准 unsquashfs 不能正常解包。

后来结合 Binwalk 的识别结果和手动 LZMA 处理，继续分析提取出的文件。

这里的教训是：文件系统解包失败，不代表固件无法分析。需要先判断实际压缩格式和厂商是否使用了定制封装。

### 3.4 遇到的问题：A200 被误判为 Linux 内核

第一次看到 A200 中存在：

- rootfstype=jffs2
- uImage
- Linux version

一开始按照 Linux 固件的思路理解。

后来重新检查发现，这些字符串可能是跨产品共用的默认字符串，不能直接作为当前设备运行 Linux 的证据。

第二轮通过 Binwalk 对 A200 进行进一步识别，结合 VxWorks 签名、MIPS 架构及编译日期等信息，确认主系统为 VxWorks 5.5.1。

最终识别结果：

- 37BC：U-Boot 相关引导程序。
- A200：主系统镜像。
- C4500：RSA 私钥文件。
- C3240 等：提取出的证书及其他文件。

这里我真正理解了：Binwalk 的输出是识别线索，需要结合镜像内容、架构和上下文判断，不能仅凭单个字符串下结论。

## 4. 第三阶段：文件系统和敏感信息审计

固件提取完成后，我没有只依赖 Binwalk 输出，而是结合 Linux 工具对文件进行分类检查。

### 4.1 文件类型识别

```bash
find . -type f -exec file {} \;
```

作用：

批量识别文件类型，帮助区分可执行文件、配置文件、证书和其他二进制内容。

### 4.2 字符串提取

```bash
strings -a A200
```

作用：

从二进制文件中提取可打印字符串。

重点关注：

- password、passwd
- fac_password、webPwd
- TELNET、UPnP
- listen_http_wan
- HEARTBEAT、USER_NAME
- CyaSSL、jQuery
- 内部路径和构建信息

### 4.3 关键词搜索

```bash
grep -aRniE \
'password|passwd|secret|token|key' \
. 2>/dev/null
```

参数说明：

- `-a`：将二进制文件按文本处理。
- `-R`：递归搜索。
- `-n`：显示匹配行号。
- `-i`：忽略大小写。
- `-E`：启用扩展正则表达式。

### 4.4 定位文件

```bash
grep -aRli "BEGIN RSA PRIVATE KEY" . 2>/dev/null
```

`-l` 只输出匹配文件名，适合先定位目标文件，再单独检查内容。

### 4.5 查看上下文

```bash
strings -a A200 | grep -B3 -A3 "fac_password"
```

- `-B3`：显示匹配内容之前三行。
- `-A3`：显示匹配内容之后三行。

这一阶段最重要的发现是：

```text
webPwd
fac_password
WaQ7xbhc9TefbwK
listen_http_lan
listen_http_wan
```

这些字符串在 A200 中相邻出现。

由此发现了硬编码出厂口令候选值，并进一步关注其与管理服务配置之间的关系。

需要注意，字符串相邻不等于程序执行顺序，也不能单凭字符串证明认证绕过已经成立。

## 5. 第四阶段：密钥分析与验证

### 5.1 最初的 PEM 标记搜索

```bash
grep -aRniE \
'-----BEGIN RSA PRIVATE KEY-----|-----END RSA PRIVATE KEY-----' \
. 2>/dev/null
```

第一次分析时发现 A200 和 C4500 均存在相关标记。

但后来发现，两者的实际内容并不相同。

### 5.2 验证 C4500

使用 OpenSSL 检查：

```bash
openssl rsa -in C4500 -check -noout
```

查看密钥信息：

```bash
openssl rsa -in C4500 -text -noout
```

导出公钥：

```bash
openssl rsa -in C4500 -pubout -out public.pem
```

第二轮复核确认：

- C4500 中存在完整 RSA 私钥。
- OpenSSL 能够解析。
- Python cryptography 也能成功解析。
- 密钥长度为 512 bit。

因此，C4500 是已验证的弱密钥候选，不只是字符串命中。

### 5.3 复核 A200

后来专门使用 Python 读取 A200，并打印 PEM 标记附近的字节：

```python
from pathlib import Path

data = Path("A200").read_bytes()

markers = [
    b"-----BEGIN CERTIFICATE-----",
    b"-----END CERTIFICATE-----",
    b"-----BEGIN RSA PRIVATE KEY-----",
    b"-----END RSA PRIVATE KEY-----"
]

for marker in markers:
    print("\n", marker.decode())
    start = 0

    while True:
        pos = data.find(marker, start)
        if pos == -1:
            break

        print(f"Offset: 0x{pos:X}")
        print(repr(data[max(0, pos-50):pos+150]))
        start = pos + len(marker)
```

结果显示 A200 中的证书和私钥标记之间只有空内容，并没有完整密钥材料。

因此：

- A200：标记存在，解析失败。
- C4500：完整私钥存在，解析成功。

这次复核纠正了第一次只凭 PEM 标记判断密钥存在的不足。

## 6. 第五阶段：X.509 证书分析

第二轮继续检查证书。

### 6.1 查找证书标记

```bash
grep -aRniE -- \
'-----BEGIN CERTIFICATE-----|-----BEGIN X509 CERTIFICATE-----|-----END CERTIFICATE-----' \
. 2>/dev/null
```

同时使用 file：

```bash
find . -type f -exec file {} \; 2>/dev/null |
grep -Ei 'certificate|X\.509|DER'
```

定位到 C3240、C46C3 等文件。

### 6.2 验证 C3240

```bash
openssl x509 -in C3240 -noout \
-subject \
-issuer \
-dates \
-serial \
-fingerprint \
-sha256
```

提取公钥：

```bash
openssl x509 -in C3240 -pubkey -noout |
openssl pkey -pubin -text -noout
```

结果：

- Subject：VeriSign Class 3 Public Primary Certification Authority - G5
- RSA 公钥：2048 bit
- 有效期：2006-11-08 至 2036-07-16
- 签名算法：sha1WithRSAEncryption

### 6.3 验证 C46C3

使用相同命令检查：

```bash
openssl x509 -in C46C3 -noout \
-subject \
-issuer \
-dates \
-serial \
-fingerprint \
-sha256
```

结果：

- Subject：TP-Link / SDMP
- RSA 公钥：512 bit
- 有效期：2014-05-08 至 2017-02-01
- 当前状态：已过期
- 签名算法：sha1WithRSAEncryption

证书解析成功，但还没有完成证书链验证、实际部署关系及 C4500 私钥与 C46C3 证书的匹配验证。

## 7. 第六阶段：组件识别与 CVE 适配

第一轮分析中识别到：

- VxWorks 5.5.1
- CyaSSL 相关字符串
- jQuery 1.10.1
- miniUPnP 相关实现
- Telnet、UPnP 等服务线索

随后对照公开漏洞资料进行版本适配。

过程中遇到的主要问题：

| 问题 | 后续处理 |
|---|---|
| 一开始想将 URGENT/11 直接套用到 VxWorks | 重新核对受影响版本，不再仅凭系统名称匹配 |
| CyaSSL 无法确认精确版本 | 将版本信息标记为推断，不能写成确定事实 |
| jQuery 版本过旧 | 继续核查对应漏洞的影响条件 |
| 组件存在不代表漏洞可利用 | 将组件识别与漏洞验证分开记录 |

这里形成的原则是：

**CVE 必须逐项核对产品版本、受影响范围、配置条件和实际调用路径。不能因为组件名称相同就直接认定漏洞适用。**

## 8. 第七阶段：报告整理与证据复核

第一轮完成了九项安全发现的整理，并形成 Word 版初步评估报告。

在报告整理过程中遇到的问题：

- 一度把 CyaSSL 的搜索过程描述错误，后来查记录发现实际是 grep 搜索 `ssl` 时匹配到了 CyaSSL。
- 曾使用 find 按文件名查找 jQuery，结果为空，后来发现 jQuery 是文件内容中的路径字符串，应该使用 grep -r 搜索内容。
- Word 文件覆盖操作导致部分已插入截图丢失，最后通过重新执行命令、重新截图恢复证据。
- 部分漏洞结论写得过于绝对，第二轮重新区分了静态发现与动态验证。

报告最终采用以下内容组织方式：

1. 目标信息
2. 分析方法
3. 安全发现
4. 证据
5. 风险等级
6. 整改建议
7. 结论与局限性

## 9. 两轮分析后的收获

第一轮让我熟悉了 Binwalk、Linux 文件检索工具和固件静态分析的基本流程。

第二轮则重点解决了第一次分析中存在的证据不足问题：

- 不再把标记出现等同于密钥有效。
- 不再把系统字符串直接当作真实系统证据。
- 不再把组件存在等同于漏洞可利用。
- 不再把证书解析成功等同于证书可信。
- 不再把静态攻击面分析写成已经完成的远程攻击验证。

最终形成了从固件识别、文件提取、敏感信息审计到密钥验证和风险评估的完整分析流程。

## 10. 后续待完善事项

- 进一步核对 C4500 私钥与 C46C3 证书是否匹配。
- 确认 CyaSSL、miniUPnP 等组件的精确版本。
- 补充关键发现的证据截图。
- 对扫描器建立独立的阳性和阴性测试集。
- 如具备设备和授权条件，再开展动态验证。

本项目以当前已有的静态分析成果为基础，不将尚未完成的工作写成已完成。
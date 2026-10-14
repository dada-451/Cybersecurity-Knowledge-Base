
# Cybersecurity Knowledge Base

> A continuously maintained knowledge base for cybersecurity learning, security testing, hands-on labs, and technical projects.

本仓库用于系统整理网络安全知识、实验记录和项目经验，重点关注 **Web 安全、渗透测试、IoT 设备安全及安全测试方法**。

## About This Repository

本知识库主要用于：

- 建立可持续维护的网络安全技术知识体系。
- 将理论知识与实际测试、抓包分析和漏洞验证结合。
- 记录实验环境、操作过程、验证证据及问题排查。
- 整理安全测试方法、技术项目经验和面试复盘。

## Knowledge Map

| 分类 | 内容 |
|---|---|
| [01 · Computer Networks](./01-Computer-Networks/) | TCP/IP、DNS、HTTP/HTTPS、TLS、Wireshark |
| [02 · Linux](./02-Linux/) | Linux 系统、进程、权限、网络服务 |
| [03 · Python](./03-Python/) | Python 基础、网络编程、安全自动化 |
| [04 · Web Security](./04-Web-Security/) | SQL 注入、XSS、SSRF、越权、文件上传 |
| [05 · Penetration Testing](./05-Penetration-Testing/) | 测试流程、信息收集、漏洞验证、报告 |
| [06 · IoT Security](./06-IoT-Security/) | 固件分析、MQTT、设备安全、IoT 标准 |
| [07 · Security Testing](./07-Security-Testing/) | 测试用例、风险评估、安全测试规范 |
| [08 · Tools](./08-Tools/) | Burp Suite、Nmap、Binwalk、Wireshark |
| [09 · Labs](./09-Labs/) | 授权实验、实验记录、验证证据 |
| [10 · Interview Preparation](./10-Interview-Preparation/) | 技术问题、常见追问、复习总结 |
| [11 · Projects](./11-Projects/) | 技术项目介绍与独立仓库索引 |

## Featured Projects

### TP-Link TL-WR886N Firmware Security Analysis

路由器固件静态安全分析项目，涉及固件结构识别、敏感信息检查、组件识别及潜在安全风险分析。

**进度：** 已完成初步静态评估，部分安全发现仍需进一步验证。

### OpenHarmony BSP Adaptation

OpenHarmony 系统移植与硬件适配实践，涉及显示、触控、摄像头等模块的调试与问题排查。

**进度：** 持续调试与整理中。

### MQTT Security Testing

计划搭建基于 Mosquitto、Python 和 Wireshark 的 MQTT 通信安全测试环境，分析认证、授权、明文通信和 TLS 配置。

**进度：** 计划中。

> 独立项目仓库将逐步建立，完整代码与项目报告保存在对应仓库中。

## Documentation Templates

- [Technical Note](./Templates/Technical-Note.md) — 技术知识笔记模板
- [Security Lab Report](./Templates/Security-Lab-Report.md) — 安全实验与验证记录模板

## Learning & Documentation Principles

1. **Understand before memorizing** — 优先理解技术原理和适用条件。
2. **Practice and verify** — 尽可能通过实验验证技术结论。
3. **Evidence-based findings** — 严格区分已验证、疑似风险和待验证。
4. **Reproducibility** — 记录实验环境、工具版本、操作步骤和结果。
5. **Continuous improvement** — 持续复盘、修订和补充知识。

## Security Notice

本仓库内容仅用于合法授权的安全学习、实验与研究。

公开内容会避免包含真实账号凭据、访问令牌、私钥、未脱敏敏感数据，以及未经授权公开的漏洞细节。

---

*Maintained by [dada-451](https://github.com/dada-451).*

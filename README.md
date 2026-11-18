# Cybersecurity Knowledge Base

<div align="center">

### 网络安全学习 · 安全测试 · 技术项目实践

<p>
  <img src="https://img.shields.io/badge/Web-Security-blue?style=flat-square">
  <img src="https://img.shields.io/badge/IoT-Security-green?style=flat-square">
  <img src="https://img.shields.io/badge/Firmware-Analysis-orange?style=flat-square">
  <img src="https://img.shields.io/badge/Python-Automation-yellow?style=flat-square">
</p>

<p>
  <a href="https://github.com/dada-451">GitHub Profile</a>
  &nbsp;·&nbsp;
  <a href="./11-Projects/">Technical Projects</a>
  &nbsp;·&nbsp;
  <a href="./09-Labs/">Security Labs</a>
</p>

</div>

> A continuously maintained knowledge base for cybersecurity learning, security testing, hands-on labs, and technical projects.

面向网络安全学习与项目实践的个人技术知识库，记录技术原理、实验过程、安全测试方法及项目实践经验。

重点关注 **Web 安全、渗透测试、IoT 设备与固件安全、嵌入式系统及安全自动化**。

---

## About This Repository

本知识库主要用于：

- 建立可持续维护的网络安全技术知识体系。
- 将理论知识与实际测试、抓包分析和漏洞验证结合。
- 记录实验环境、操作过程、验证证据及问题排查。
- 整理安全测试方法、技术项目经验和面试复盘。
- 通过项目实践积累可复现、可追溯的技术经验。

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

---

## Featured Projects

| 项目 | 技术方向 | 项目记录 |
|---|---|---|
| TP-Link TL-WR886N 固件安全分析 | 固件分析 / Python / OpenSSL | [查看](./11-Projects/TP-Link-WR886N-Firmware-Security/) |
| OpenHarmony BSP 适配 | OpenHarmony / Linux / BSP | [查看](./11-Projects/OpenHarmony-BSP-Adaptation/) |
| 企业网络红蓝对抗 | Web 安全 / 内网渗透 / Windows | [查看](./09-Labs/Enterprise-Red-Blue-Exercise.md) |
| 网络流量异常检测系统 | Python / Scapy / Flask / ML | [查看](./11-Projects/Network-Traffic-Anomaly-Detection/) |
| 职网清源 Demo | OCR / Python / 风险分析 | [查看](./11-Projects/ZhiwangQingyuanDemo/) |

### 01. TP-Link TL-WR886N V2.0 Firmware Security Analysis

<sub>个人实践项目 · IoT 固件安全</sub>

针对 TP-Link 路由器固件开展静态安全分析，重点关注固件结构、敏感信息及潜在安全风险。

- 使用 Binwalk 识别固件结构并提取文件系统。
- 排查固件中的敏感信息、密钥及相关安全特征。
- 使用 OpenSSL 检查 RSA 私钥，结合 Radare2 开展 MIPS 二进制初步分析。
- 编写 Python 敏感信息扫描脚本，生成结构化分析结果。

**当前状态：** 已完成初步静态评估与扫描器开发，部分安全发现仍需进一步复核。

项目文档：[项目总览](./11-Projects/TP-Link-WR886N-Firmware-Security/README.md) · [安全评估报告](./11-Projects/TP-Link-WR886N-Firmware-Security/01-Firmware-Security-Assessment.md) · [静态分析记录](./11-Projects/TP-Link-WR886N-Firmware-Security/02-Static-Analysis-Record.md) · [Python 扫描器](./11-Projects/TP-Link-WR886N-Firmware-Security/03-Python-Scanner-Usage.md)

### 02. OpenHarmony 6.1 BSP Adaptation

<sub>校企合作项目 · 嵌入式系统与硬件适配</sub>

参与基于 OpenHarmony 6.1 的智能手持终端 BSP 适配与调试。

- 参与目标工程编译环境搭建、系统构建及 PAC 打包。
- 参考企业 BSP 与相关补丁，开展显示、触控相关代码和配置集成。
- 排查显示 ID、LK 与内核配置不一致等适配问题。
- 完成目标设备屏幕驱动适配及实际运行验证。

**当前状态：** 已完成屏幕驱动移植与设备验证，其他模块持续整理。

项目文档：[项目总览](./11-Projects/OpenHarmony-BSP-Adaptation/README.md) · [Display Driver Porting](./11-Projects/OpenHarmony-BSP-Adaptation/Display-Driver-Porting.md)

### 03. Enterprise Network Red Team Exercise

<sub>网络攻防技术课程项目 · 红队安全测试</sub>

基于 WireGuard 的虚拟企业分区网络开展授权红队安全测试，涉及 DMZ1、DMZ2 和 Windows 目标。

- 使用 Nmap、Gobuster、curl、sqlmap 等工具进行资产发现、服务枚举及 Web 漏洞验证。
- 实践 SQL 注入、命令注入、文件上传、Struts2 S2-045 等漏洞验证。
- 使用 Chisel、Proxychains 建立授权内网访问路径。
- 开展 Windows 远程管理、权限验证及 PrintNightmare 实验。
- 整理攻击链复盘与内网渗透标准作业流程（SOP v1.0）。

项目文档：[项目复盘](./09-Labs/Enterprise-Red-Blue-Exercise.md) · [内网渗透 SOP](./05-Penetration-Testing/Internal-Pentest-SOP.md)

### 04. Network Traffic Anomaly Detection System

<sub>课程实训 · 双人项目 · Python / Scapy / Flask / Machine Learning</sub>

围绕离线 PCAP 数据开展网络流量分析、特征提取与异常检测。

- 使用 Python 和 Scapy 解析数据包，进行协议分布与流量统计。
- 结合流量特征开展规则与机器学习模型检测。
- 整理 Random Forest、XGBoost、LightGBM、CatBoost 等模型实验结果。
- 使用 Flask 展示流量分析及检测结果。

项目文档：[项目记录](./11-Projects/Network-Traffic-Anomaly-Detection/README.md)

### 05. 职网清源 · 招聘风险辅助初筛 Demo

<sub>挑战杯校赛项目 · 团队开发</sub>

围绕招聘信息中的潜在风险线索，参与辅助初筛原型的设计与实现。

- 参与 OCR 字段抽取、风险规则及相关接口对齐。
- 结合规则分级与 Union-Find 关联聚类整理风险线索。
- 参与 Streamlit 展示、测试与人工复核流程整合。
- 通过证据展示和 JSON 报告辅助风险核验。

**当前状态：** 已完成本机演示闭环与初步验证，持续完善标注与评测。

项目文档：[项目记录](./11-Projects/ZhiwangQingyuanDemo/)

### 06. MQTT Security Testing

<sub>IoT 通信安全 · 学习计划</sub>

计划搭建基于 Mosquitto、Python 和 Wireshark 的 MQTT 通信安全测试环境，研究发布订阅机制、身份认证、访问控制、明文通信及 TLS 配置。

**当前状态：** 计划中，后续逐步补充实验记录。

> 独立项目仓库将逐步建立，完整代码与项目报告保存在对应仓库中。

---

## Documentation Templates

- [Technical Note](./Templates/Technical-Note.md) — 技术知识笔记模板
- [Security Lab Report](./Templates/Security-Lab-Report.md) — 安全实验与验证记录模板

## Technical Stack

以下为知识库及项目中涉及的技术和工具。

**Programming**

<p>
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&amp;logo=python&amp;logoColor=white">
  <img src="https://img.shields.io/badge/C-00599C?style=flat-square&amp;logo=c&amp;logoColor=white">
</p>

**Security Testing**

<p>
  <img src="https://img.shields.io/badge/Burp%20Suite-FF6633?style=flat-square">
  <img src="https://img.shields.io/badge/Nmap-4682B4?style=flat-square">
  <img src="https://img.shields.io/badge/Wireshark-1679A7?style=flat-square&amp;logo=wireshark&amp;logoColor=white">
</p>

**Firmware & Embedded**

<p>
  <img src="https://img.shields.io/badge/OpenHarmony-000000?style=flat-square">
  <img src="https://img.shields.io/badge/Binwalk-Firmware%20Analysis-orange?style=flat-square">
  <img src="https://img.shields.io/badge/OpenSSL-721412?style=flat-square">
  <img src="https://img.shields.io/badge/Radare2-Binary%20Analysis-purple?style=flat-square">
</p>

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

<div align="center">

**Maintained by [dada-451](https://github.com/dada-451)**

*Cybersecurity Learning · Security Testing · Technical Projects*

</div>
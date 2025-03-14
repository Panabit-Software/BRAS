# BRAS 配置指南（支持 RAAS 代理与代理代拨）

## 目录
- [BRAS 概述](#bras-概述)
- [RAAS 功能](#raas-功能)
  - [RAAS 代理](#raas-代理)
  - [RAAS 代理代拨](#raas-代理代拨)
- [逻辑拓扑](#逻辑拓扑)
- [配置说明](#配置说明)
  - [RAAS 代理配置](#raas-代理配置)
  - [RAAS 代理代拨配置](#raas-代理代拨配置)
- [相关命令与管理](#相关命令与管理)
- [常见问题 FAQ](#常见问题-faq)

---

## BRAS 概述

**BRAS（Broadband Remote Access Server，宽带远程访问服务器）**，主要用于 **PPPoE 拨号、VPN 认证、用户管理、流量控制**。在 ISP（互联网服务提供商）架构中，BRAS 负责接入认证，并可以 **通过 RAAS 代理** 进行 **身份认证与计费**。

---

## RAAS 功能

RAAS（RADIUS Authentication and Accounting Server）提供代理认证、代理代拨、负载均衡等功能。

### RAAS 代理
**应用场景**
**外包拨号/VPN 认证**：RAAS 代理 RADIUS 认证，保护 RADIUS 服务器不暴露公网。
**大规模 RADIUS 代理**：RAAS 代理多个 RADIUS 服务器，提高负载均衡能力。

### RAAS 代理代拨
**应用场景**
**Wi-Fi 热点、企业 VPN 认证**。
**自动注册并动态绑定拨号信息**，降低运维成本。

---

## 逻辑拓扑

```
+-----------+        +-------------+        +-----------------+
|  终端用户  | ----> |   BRAS 设备   | ----> |  RAAS 代理服务器  |
+-----------+        +-------------+        +-----------------+
                          |                     |
                          |                     |
                   +-----------------+   +----------------+
                   | RADIUS 服务器 1  |   | RADIUS 服务器 2 |
                   +-----------------+   +----------------+
```

**数据流**
终端用户（PPPoE/VPN）拨号 → BRAS 设备
BRAS 设备将 **认证请求** 发送至 **RAAS**
RAAS 代理 **解析用户名**，转发给 **RADIUS 服务器**
认证通过后，用户成功接入

---

## 配置说明

### RAAS 代理配置

#### ① 创建项目
- 进入 **【当前项目】 > 【项目列表】**
- **认证模式选择**：`RADIUS 代理认证`

#### ② 添加 RADIUS 服务器
- 进入 **【运营商接入管理】**  
- 配置 **RADIUS 服务器信息**：
```sh
radius-server add name=isp1 address=192.168.1.10 secret=123456
```

#### ③ 创建用户组
- 进入 **【当前项目】 > 【用户组】**
- **注册用户组**，RAAS 代理会 **自动归类认证用户**

#### ④ 修改项目设置
- 进入 **【当前项目】 > 【项目列表】**
- **选择默认运营商 & 默认用户组**

#### ⑤ 对接 BRAS
- 进入 **【对象管理】 > 【RADIUS】**
- **添加 RAAS 设备**，并在 **PPPoE/Web 认证** 中调用

---

### RAAS 代理代拨配置

#### ① 启用 RAAS 代理
- 参考 **[RAAS 代理配置](#41-raas-代理配置)**

#### ② 绑定代拨信息
- **方式 1**：手动绑定  
  - 进入 **【当前项目】 > 【所有用户】**，编辑用户，添加 **代拨信息**。
  
- **方式 2**：用户自助绑定  
  - 访问 `http://RAAS的IP` 进入 **自助服务后台**，绑定拨号信息。

#### ③ BRAS 代拨配置
- 进入 **【用户认证】 > 【PPPoE 代拨】**
- 在 **参数设置** 中，启用 **代拨功能**：
```sh
pppoe enable proxy-authentication
pppoe proxy-auth-server 192.168.1.20 secret=raas123
```

---

## 相关命令与管理

### BRAS 设备管理

```sh
# 查看当前 RADIUS 服务器状态
show radius status

# 测试 RADIUS 服务器连通性
test radius server 192.168.1.10 user=test password=123456

# 查看 PPPoE 认证日志
show pppoe log
```

### RAAS 设备管理

```sh
# 查看代理状态
raasctl show requests

# 查看已绑定的代拨用户
raasctl show users

# 统计认证请求
raasctl show stats
```

---

## 常见问题 FAQ

### **Q1：RAAS 代理请求失败，BRAS 认证不通过？**
**解决方案：**
**检查 RAAS 配置**  
   ```sh
   raasctl show config
   ```
**检查 RADIUS 服务器连接**  
   ```sh
   ping 192.168.1.10
   ```

### **Q2：用户拨号后无法自动创建账号？**
**解决方案：**
- 确保 **RAAS 代理代拨功能已启用**
- 检查 **默认用户组** 配置
- 查看 RAAS 日志：
  ```sh
  raasctl show logs
  ```

### **Q3：如何启用日志调试模式？**
```sh
raasctl enable debug
```

---

## 相关链接
- [PPPoE 认证配置指南](#)
- [RADIUS 服务器对接](#)
- [Panabit 官方文档](https://www.panabit.com/)

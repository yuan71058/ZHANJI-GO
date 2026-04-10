<div align="center">

# 冬浩验证系统 - Go SDK

![Version](https://img.shields.io/badge/version-1.1-blue.svg)
![Go](https://img.shields.io/badge/Go-1.25+-00ADD8.svg?logo=go)
![License](https://img.shields.io/badge/license-MIT-green.svg)

**冬浩验证系统 API 的完整 Go 语言封装**

[安装使用](#安装) | [快速开始](#快速开始) | [API 文档](#api-接口列表) | [示例代码](example/main.go)

</div>

---

## 更新日志

### v1.1 (2026-04-10)

#### Bug 修复
- **修复登录后心跳失败的问题**
  - 问题原因：SDK 在解析响应时直接使用了顶层的 `token`（MD5 hash），而没有优先使用 `result.tokenid`（数据库自增 ID）
  - 影响范围：所有加密模式（RC4/RSA/Base64/AES-GCM）下的登录和心跳功能
  - 修复方案：修改 `httpPost` 函数中的 Token 设置逻辑，优先使用 `result.tokenid`

#### 代码变更
- `donghao.go`: 修改 `httpPost` 函数中三处 Token 设置逻辑
  ```go
  // 修复前
  if result.Token != "" {
      c.currentToken = result.Token
  }
  
  // 修复后
  if tokenID := result.GetTokenID(); tokenID != "" {
      c.currentToken = tokenID
  } else if result.Token != "" {
      c.currentToken = result.Token
  }
  ```

#### 技术细节
- 服务端登录响应包含两个字段：
  - `result.tokenid`: 数据库自增 ID（数字类型），用于心跳验证
  - `token`: MD5 hash 字符串，用于签名验证
- 正确流程：客户端应使用 `tokenid` 作为心跳参数，而非 `token`

### v1.0 (2026-03-31)
- 初始版本发布
- 完整的 API 封装
- 支持多种加密方式
- 设备信息采集功能

---

## 功能特性

- **完整的 API 封装** - 用户管理、卡密管理、云变量、云计算函数、黑名单等全部接口
- **多种加密方式** - RC4 / RSA / Base64 / AES-256-GCM / 自定义加密
- **MD5 签名机制** - 确保请求安全性和数据完整性
- **设备信息采集** - 机器码、硬件 ID 获取，支持 Windows/Android
- **自动 Token 轮换** - 服务端返回新 Token 时自动更新

---

## 安装

```bash
go get github.com/yuan71058/DONGHAO-GO-SDK
```

**环境要求**: Go 1.25+

---

## 快速开始

```go
package main

import (
    "fmt"
    "log"

    donghao "github.com/yuan71058/DONGHAO-GO-SDK"
)

func main() {
    // 创建客户端
    client := donghao.NewClient("https://your-api-domain.com", 1)
    client.SetTimeout(30)

    // 用户登录
    result, err := client.Login("username", "password", "1.0", "mac123", "192.168.1.1", "client001")
    if err != nil {
        log.Fatal(err)
    }

    if result.IsSuccess() {
        fmt.Println("登录成功, Token:", client.GetToken())
    } else {
        fmt.Println("登录失败:", result.Msg())
    }
}
```

---

## 客户端配置

### 创建客户端

```go
client := donghao.NewClient(baseURL string, appID int)
```

| 参数 | 说明 |
|------|------|
| `baseURL` | API 服务端地址 |
| `appID` | 软件ID（从管理后台获取） |

### 配置方法

| 方法 | 说明 |
|------|------|
| `SetTimeout(seconds int)` | 设置 HTTP 超时时间（默认 30 秒） |
| `SetEncryption(encType int, key string)` | 设置加密方式和密钥 |
| `SetSignConfig(appKey, template, needSign)` | 配置 MD5 签名 |
| `SetHeartbeatInterval(seconds int)` | 设置心跳间隔（默认 150 秒） |

### 加密类型

| 常量 | 值 | 说明 |
|------|-----|------|
| `ENC_NONE` | 0 | 明文传输 |
| `ENC_RC4` | 1 | RC4 对称加密 |
| `ENC_RSA` | 2 | RSA 非对称加密 |
| `ENC_BASE64` | 3 | Base64 编码 |
| `ENC_CUSTOM` | 4 | 自定义加密 |
| `ENC_AES_GCM` | 5 | AES-256-GCM 认证加密 |

```go
// 使用 RC4 加密
client.SetEncryption(donghao.ENC_RC4, "your-secret-key")

// 使用 AES-256-GCM 加密
client.SetEncryption(donghao.ENC_AES_GCM, "32-byte-key-here!!!")
```

---

## 设备信息获取

```go
// 获取机器码
machineCode, err := donghao.GetMachineCode()
if err != nil {
    machineCode = donghao.GetMachineCodeSafe() // 备用方法
}

// 获取硬件 ID
hardwareID, _ := donghao.GetHardwareID()

// 获取详细硬件信息
hwInfo, _ := donghao.GetHardwareInfo()

// 生成随机客户端 ID
clientID := donghao.GenerateClientID()
```

| 函数 | 返回值 | 平台 |
|------|--------|------|
| `GetMachineCode()` | 机器码 (XXXX-XXXX-XXXX-XXXX) | Win/Android |
| `GetMachineCodeSafe()` | 机器码（不报错） | 全平台 |
| `GetHardwareID()` | 硬件 ID | Win/Android |
| `GetHardwareInfo()` | 硬件信息 Map | Win/Android |
| `GenerateClientID()` | 随机客户端 ID | 全平台 |

---

## API 接口列表

### 返回结果结构体

```go
type Result struct {
    Code   int         // 状态码: 200/1=成功
    Result interface{} // 结果数据
    Data   interface{} // 业务数据
    Uuid   string      // UUID
    Token  string      // 新 Token
}
```

| 方法 | 说明 |
|------|------|
| `IsSuccess() bool` | 判断是否成功 |
| `Msg() string` | 获取返回消息 |
| `GetTokenID() string` | 获取 TokenID（数据库自增 ID） |
| `GetData() (string, error)` | 获取用户数据（Base64 解码） |
| `GetGroupData() (string, error)` | 获取分组数据（Base64 解码） |
| `GetVariableValue() (string, error)` | 获取变量值（Base64 解码） |

### 用户管理

| 方法 | 说明 |
|------|------|
| `Login(user, pwd, ver, mac, ip, clientid)` | 用户登录 |
| `LoginCard(card, ver, mac, ip, clientid)` | 卡密登录 |
| `Reg(user, pwd, card, qq, email, tjr, ver, mac, ip, clientid)` | 用户注册 |
| `Logout(user, tokenid, ver, mac, ip, clientid)` | 用户注销 |
| `Heartbeat(user, tokenid, ver, mac, ip, clientid)` | 心跳维持在线 |
| `GetUser(user, tokenid, ver, mac, ip, clientid)` | 获取用户信息 |
| `Uppwd(user, pwd, newpwd, ver, mac, ip, clientid)` | 修改密码 |
| `Binding(user, pwd, newuser, newmac, newip, qq, ver, mac, ip, clientid)` | 绑定新设备 |
| `Bindreferrer(user, pwd, tjr, ver, mac, ip, clientid)` | 绑定推荐人 |

### 卡密管理

| 方法 | 说明 |
|------|------|
| `Recharge(user, card, ver, mac, ip, clientid)` | 卡密充值 |

### 用户数据

| 方法 | 说明 |
|------|------|
| `GetUdata(user, tokenid, ver, mac, ip, clientid)` | 获取用户数据 |
| `SetUdata(user, tokenid, udata, ver, mac, ip, clientid)` | 设置用户数据 |
| `GetUdata2(user, tokenid, ver, mac, ip, clientid)` | 获取用户数据2 |
| `SetUdata2(user, tokenid, udata, ver, mac, ip, clientid)` | 设置用户数据2 |

### 云变量/常量

| 方法 | 说明 |
|------|------|
| `GetVariable(user, tokenid, key, ver, mac, ip, clientid)` | 获取云变量 |
| `SetVariable(user, tokenid, key, value, ver, mac, ip, clientid)` | 设置云变量 |
| `DelVariable(user, tokenid, key, ver, mac, ip, clientid)` | 删除云变量 |
| `Constant(user, tokenid, key, ver, mac, ip, clientid)` | 获取云常量 |

### 云计算函数

| 方法 | 说明 |
|------|------|
| `Func(user, tokenid, func, para, ver, mac, ip, clientid)` | 云计算函数（需登录） |
| `Func2(func, para, ver, mac, ip, clientid)` | 云计算函数（免登录） |
| `CallPHP(user, tokenid, func, para, ver, mac, ip, clientid)` | 调用 PHP 函数（需登录） |
| `CallPHP2(func, para, ver, mac, ip, clientid)` | 调用 PHP 函数（免登录） |

### 黑名单管理

| 方法 | 说明 |
|------|------|
| `GetBlack(bType, bData)` | 查询黑名单 (type: ip/mac/user) |
| `SetBlack(bType, data, note, ver, mac, ip, clientid)` | 添加黑名单 |

### 其他接口

| 方法 | 说明 |
|------|------|
| `DeductPoints(user, tokenid, count, ver, mac, ip, clientid)` | 扣除积分 |
| `AddLog(user, info, ver, mac, ip, clientid)` | 添加日志 |
| `CheckAuth(user, pwd, md5, ver, mac, ip, clientid)` | 验证授权 |
| `Init(ver, mac, ip, clientid)` | 软件初始化 |
| `Notice(title, ver, mac, ip, clientid)` | 获取公告 |
| `Ver(ver, mac, ip, clientid)` | 版本检查 |
| `Relay(params, ver, mac, ip, clientid)` | 中继转发 |

---

## 错误处理

```go
result, err := client.Login(user, pwd, ver, mac, ip, clientid)

if err != nil {
    fmt.Println("网络错误:", err)
    return
}

if result.IsSuccess() {
    fmt.Println("成功")
} else {
    fmt.Printf("失败: code=%d, msg=%s\n", result.Code, result.Msg())
}
```

| 错误码 | 含义 |
|--------|------|
| 200 / 1 | 成功 |
| 其他值 | 业务错误（具体含义参考服务端文档） |

---

## 项目结构

```
DONGHAO-GO/
├── donghao.go       # SDK 核心实现
├── example/
│   └── main.go      # 完整使用示例
├── README.md         # 本文件
├── API.md           # 详细 API 文档
├── go.mod
└── LICENSE          # MIT 协议
```

---

## 许可证

[MIT License](LICENSE)

---

<div align="center">

**冬浩验证系统** - 让软件保护变得简单高效

</div>

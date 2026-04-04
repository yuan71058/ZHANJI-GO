<div align="center">

# 🚀 冬浩验证系统 - Go语言 SDK

![Version](https://img.shields.io/badge/version-1.0-blue.svg?style=for-the-badge)
![Go](https://img.shields.io/badge/Go-1.16+-00ADD8.svg?style=for-the-badge&logo=go)
![License](https://img.shields.io/badge/license-MIT-green.svg?style=for-the-badge)

**完整的Go语言API客户端实现，支持用户管理、卡密系统、云变量、云计算等功能**

[开始使用](#快速开始) • [API文档](#api接口) • [示例代码](#完整示例)

</div>

---

## 📖 目录

- [特性](#特性)
- [环境要求](#环境要求)
- [安装](#安装)
- [快速开始](#快速开始)
- [配置说明](#配置说明)
- [设备标识](#设备标识)
- [API接口](#api接口)
- [加密通信](#加密通信)
- [自动心跳（已禁用）](#自动心跳已禁用)
- [错误处理](#错误处理)
- [完整示例](#完整示例)
- [许可证](#许可证)

---

## ✨ 特性

- 🚀 **完整API支持** - 覆盖所有冬浩验证系统接口
- 🔐 **多种加密** - 支持RC4、RSA、Base64、明文传输
- 📝 **签名验证** - 支持MD5签名防篡改
- 🎯 **标准库实现** - 纯Go标准库，无外部依赖
- 📚 **详细文档** - 完整的中文文档和示例代码
- 💳 **卡密登录** - 支持直接使用卡密登录（需后端开启）
- 🔧 **设备标识** - 跨平台硬件设备码生成（Windows/Android）

---

## 🛠️ 环境要求

- **Go 1.16+**
- 依赖：标准库（无需额外依赖）

---

## 📦 安装

```bash
go get gitee.com/yuan71058/dong-hao-verification
```

---

## 🚀 快速开始

### 1. 基本使用

```go
package main

import (
    "fmt"
    "log"
    "gitee.com/yuan71058/dong-hao-verification"
)

func main() {
    // 创建客户端（第二个参数是软件ID，从管理后台获取）
    client := donghao.NewClient("https://your-api-domain.com", 1)
    client.SetTimeout(30)

    // 用户登录
    result, err := client.Login("username", "password", "1.0", "mac123", "192.168.1.1", "client001")
    if err != nil {
        log.Fatal(err)
    }

    if result.IsSuccess() {
        fmt.Println("登录成功，Token:", client.GetToken())
    } else {
        fmt.Println("登录失败:", result.Msg())
    }
}
```

### 2. 卡密登录

```go
// 使用卡密直接登录（需后端开启充值卡登录模式）
result, err := client.LoginCard("YKUvGvYSuVFFTp41T1WO", "1.0", "mac123", "192.168.1.1", "client001")
if err != nil {
    log.Fatal(err)
}

if result.IsSuccess() {
    fmt.Println("卡密登录成功，Token:", client.GetToken())
    fmt.Println("用户名:", client.GetCurrentUser())
} else {
    fmt.Println("卡密登录失败:", result.Msg())
}
```

---

## ⚙️ 配置说明

### 创建客户端

```go
client := donghao.NewClient(baseURL string, appID int)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| baseURL | string | API服务器地址，如：https://your-domain.com |
| appID | int | 软件ID，从管理后台获取 |

### 配置方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `SetTimeout(seconds int)` | 超时秒数 | 设置HTTP请求超时，默认30秒 |
| `SetEncryption(encType int, key string)` | 加密类型, 密钥 | 设置数据加密方式 |
| `SetSignConfig(appKey, template string, needSign bool)` | 应用密钥, 签名模板, 是否签名 | 设置签名配置 |
| `SetHeartbeatInterval(seconds int)` | 间隔秒数 | 设置自动心跳间隔，默认60秒（已禁用，保留接口） |

### 加密类型常量

```go
const (
    ENC_NONE   = 0  // 不加密，明文传输
    ENC_RC4    = 1  // RC4流加密算法
    ENC_RSA    = 2  // RSA非对称加密
    ENC_BASE64 = 3  // Base64编码
    ENC_CUSTOM = 4  // 自定义加密算法
)
```

---

## 🔧 设备标识

SDK提供跨平台的设备标识生成功能，支持Windows和Android系统。

### 获取机器码

```go
// 获取基于硬件的机器码（跨平台）
machineCode, err := donghao.GetMachineCode()
if err != nil {
    // 使用安全获取方式（不会失败）
    machineCode = donghao.GetMachineCodeSafe()
}
fmt.Println("机器码:", machineCode)
// 输出: XXXX-XXXX-XXXX-XXXX
```

### 获取硬件ID

```go
// 获取详细硬件ID
hardwareID, err := donghao.GetHardwareID()
if err != nil {
    log.Fatal(err)
}
fmt.Println("硬件ID:", hardwareID)
```

### 获取硬件信息

```go
// 获取详细硬件信息
hwInfo, err := donghao.GetHardwareInfo()
if err != nil {
    log.Fatal(err)
}
for key, value := range hwInfo {
    fmt.Printf("%s: %s\n", key, value)
}
```

### 生成客户端ID

```go
// 生成客户端ID（基于时间戳和随机数）
clientID := donghao.GenerateClientID()
fmt.Println("客户端ID:", clientID)
```

### 设备标识说明

| 函数 | 说明 | 平台支持 |
|------|------|----------|
| `GetMachineCode()` | 获取基于硬件的机器码 | Windows/Android |
| `GetMachineCodeSafe()` | 安全获取机器码（不会失败） | 所有平台 |
| `GetHardwareID()` | 获取详细硬件ID | Windows/Android |
| `GetHardwareInfo()` | 获取详细硬件信息 | Windows/Android |
| `GenerateClientID()` | 生成客户端ID | 所有平台 |
| `GenerateDeviceID()` | 生成设备唯一标识 | 所有平台 |

### 机器码生成原理

机器码采用**双层MD5加密**：

```
第一层: 硬件信息(CPU+主板+硬盘+MAC+BIOS) -> MD5 -> 中间串(XXXX-XXXX-XXXX-XXXX)
第二层: 中间串 -> MD5 -> 最终机器码(XXXX-XXXX-XXXX-XXXX)
```

---

## 📡 API接口

### 返回结果结构

```go
type Result struct {
    Code   int         // 返回状态码 200/1=成功
    Result interface{} // 返回结果数据
    Data   interface{} // 返回数据内容
    Token  string      // Token
}

func (r *Result) IsSuccess() bool      // 检查是否成功
func (r *Result) Msg() string          // 获取返回消息
func (r *Result) GetTokenID() string   // 获取TokenID
func (r *Result) GetData() (string, error)         // 获取解码后的用户数据
func (r *Result) GetGroupData() (string, error)    // 获取解码后的用户组数据
func (r *Result) GetVariableValue() (string, error) // 获取解码后的变量值
```

### 用户管理接口

#### 用户登录
```go
result, err := client.Login(user, pwd, ver, mac, ip, clientid string)
```

#### 卡密登录
```go
result, err := client.LoginCard(card, ver, mac, ip, clientid string)
// 注意: 需要后端开启"充值卡登录模式"(dl_type=1)
```

#### 用户注册
```go
result, err := client.Reg(user, pwd, card, userqq, email, tjr, ver, mac, ip, clientid string)
```

#### 心跳检测
```go
result, err := client.Heartbeat(user, tokenid, ver, mac, ip, clientid string)
```

#### 用户登出
```go
result, err := client.Logout(user, tokenid, ver, mac, ip, clientid string)
```

#### 获取用户信息
```go
result, err := client.GetUser(user, tokenid, ver, mac, ip, clientid string)
```

#### 修改密码
```go
result, err := client.Uppwd(user, pwd, newpwd, ver, mac, ip, clientid string)
```

#### 换绑信息
```go
result, err := client.Binding(user, pwd, card, ver, mac, ip, clientid string)
```

#### 绑定推荐人
```go
result, err := client.Bindreferrer(user, pwd, tjr, ver, mac, ip, clientid string)
```

#### 卡密充值
```go
result, err := client.Recharge(user, card, ver, mac, ip, clientid string)
```

### 数据管理接口

#### 获取用户云数据
```go
result, err := client.GetUdata(user, tokenid, ver, mac, ip, clientid string)
data, err := result.GetData()  // 自动base64解码
```

#### 设置用户云数据
```go
result, err := client.SetUdata(user, tokenid, udata, ver, mac, ip, clientid string)
```

#### 获取用户云数据2
```go
result, err := client.GetUdata2(user, tokenid, ver, mac, ip, clientid string)
```

#### 设置用户云数据2
```go
result, err := client.SetUdata2(user, tokenid, udata, ver, mac, ip, clientid string)
```

### 云变量接口

#### 获取云变量
```go
result, err := client.GetVariable(user, tokenid, cloudkey, ver, mac, ip, clientid string)
value, err := result.GetVariableValue()  // 自动base64解码
```

#### 设置云变量
```go
result, err := client.SetVariable(user, tokenid, cloudkey, cloudvalue, ver, mac, ip, clientid string)
```

#### 删除云变量
```go
result, err := client.DelVariable(user, tokenid, cloudkey, ver, mac, ip, clientid string)
```

#### 获取云常量
```go
result, err := client.Constant(user, tokenid, cloudkey, ver, mac, ip, clientid string)
```

### 云计算接口

#### 云计算函数（需登录）
```go
result, err := client.Func(user, tokenid, fun, para, ver, mac, ip, clientid string)
```

#### 云计算函数（免登录）
```go
result, err := client.Func2(fun, para, ver, mac, ip, clientid string)
```

#### 调用PHP函数1（需登录）
```go
result, err := client.CallPHP(user, tokenid, fun, para, ver, mac, ip, clientid string)
```

#### 调用PHP函数2（免登录）
```go
result, err := client.CallPHP2(fun, para, ver, mac, ip, clientid string)
```

### 黑名单接口

#### 查询黑名单
```go
result, err := client.GetBlack(bType, bData string)
// bType: ip, mac, user
```

#### 添加黑名单
```go
result, err := client.SetBlack(bType, bData, bBz, ver, mac, ip, clientid string)
```

### 其他接口

#### 扣除点数
```go
result, err := client.DeductPoints(user, tokenid string, sl int, ver, mac, ip, clientid string)
```

#### 添加日志
```go
result, err := client.AddLog(user, infos, ver, mac, ip, clientid string)
```

#### 验证授权
```go
result, err := client.CheckAuth(user, pwd, md5, ver, mac, ip, clientid string)
```

#### 软件初始化
```go
result, err := client.Init(ver, mac, ip, clientid string)
```

#### 获取公告
```go
result, err := client.Notice(title, ver, mac, ip, clientid string)
```

#### 版本检测
```go
result, err := client.Ver(ver, mac, ip, clientid string)
```

#### 数据转发
```go
result, err := client.Relay(params map[string]string, ver, mac, ip, clientid string)
```

---

## 🔐 加密通信

### RC4加密

```go
client.SetEncryption(donghao.ENC_RC4, "your_rc4_secret_key")
```

### RSA加密

```go
client.SetEncryption(donghao.ENC_RSA, privateKeyPEM)
```

### Base64编码

```go
client.SetEncryption(donghao.ENC_BASE64, "")
```

### 无加密

```go
client.SetEncryption(donghao.ENC_NONE, "")
```

---

## 💓 自动心跳（已禁用）

> ⚠️ **注意**: 自动心跳功能存在问题，已禁用。请使用手动心跳 `Heartbeat()` 方法代替。

```go
// 手动心跳示例
result, err := client.Heartbeat(user, tokenid, ver, mac, ip, clientid)
if err != nil {
    fmt.Println("心跳失败:", err)
}
```

---

## 🚨 错误处理

### 检查结果

```go
result, err := client.Login(user, pwd, ver, mac, ip, clientid)

if err != nil {
    // 网络错误或请求失败
    fmt.Println("请求错误:", err)
    return
}

if result.IsSuccess() {
    // 成功 (code == 200 || code == 1)
    fmt.Println("成功:", result.Data)
} else {
    // 业务错误
    fmt.Println("错误码:", result.Code)
    fmt.Println("错误信息:", result.Msg())
}
```

### 常见状态码

| 状态码 | 说明 |
|--------|------|
| 200/1 | 成功 |
| -1 | 请求失败/解析错误 |
| 其他 | 业务错误，参见msg字段 |

---

## 📋 完整示例

```go
package main

import (
    "fmt"
    "log"
    "gitee.com/yuan71058/dong-hao-verification"
)

func main() {
    client := donghao.NewClient("http://your-domain.com", 1)
    client.SetTimeout(30)
    
    // 获取机器码
    machineCode, err := donghao.GetMachineCode()
    if err != nil {
        machineCode = donghao.GetMachineCodeSafe()
    }
    
    // 生成客户端ID
    clientID := donghao.GenerateClientID()
    
    user := "testuser"
    pwd := "testpwd"
    ver := "1.0.0"
    mac := machineCode
    ip := "192.168.1.100"
    
    // 1. 用户登录
    result, err := client.Login(user, pwd, ver, mac, ip, clientID)
    if err != nil || !result.IsSuccess() {
        log.Fatal("登录失败:", result.Msg())
    }
    fmt.Println("登录成功")
    token := client.GetToken()
    
    // 2. 手动心跳（自动心跳已禁用）
    heartResult, _ := client.Heartbeat(user, token, ver, mac, ip, clientID)
    fmt.Println("心跳结果:", heartResult.IsSuccess())
    
    // 3. 获取用户信息
    userResult, _ := client.GetUser(user, token, ver, mac, ip, clientID)
    if data, err := userResult.GetData(); err == nil {
        fmt.Println("用户数据:", data)
    }
    
    // 4. 获取云变量
    varResult, _ := client.GetVariable(user, token, "config", ver, mac, ip, clientID)
    if value, err := varResult.GetVariableValue(); err == nil {
        fmt.Println("云变量:", value)
    }
    
    // 5. 用户登出
    client.Logout(user, token, ver, mac, ip, clientID)
    fmt.Println("已登出")
}
```

更多示例请查看 [example/main.go](example/main.go)

---

## 📝 版本历史

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0 | 2026-03-31 | 项目更名，版本重置 |

---

## 📄 许可证

本项目采用 [MIT](LICENSE) 许可证开源。

---

<div align="center">

**冬浩验证系统** - 专业的软件授权验证解决方案

</div>

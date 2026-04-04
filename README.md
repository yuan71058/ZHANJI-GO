<div align="center">

# 冬浩验证系统 - Go语言SDK

![Version](https://img.shields.io/badge/version-1.0-blue.svg?style=for-the-badge)
![Go](https://img.shields.io/badge/Go-1.16+-00ADD8.svg?style=for-the-badge&logo=go)
![License](https://img.shields.io/badge/license-MIT-green.svg?style=for-the-badge)

**冬浩验证系统API的完整Go语言封装，支持用户管理、卡密管理、云变量、云计算函数等全功能**

[快速开始](#快速开始) | [API文档](#api文档) | [使用示例](#完整示例)

</div>

---

## 目录

- [功能特性](#功能特性)
- [环境要求](#环境要求)
- [快速安装](#快速安装)
- [快速开始](#快速开始)
- [客户端配置](#客户端配置)
- [加密方式](#加密方式)
- [设备信息获取](#设备信息获取)
- [API文档](#api文档)
- [加密通信](#加密通信)
- [自动心跳维持在线状态](#自动心跳维持在线状态)
- [错误处理](#错误处理)
- [完整示例](#完整示例)
- [版本更新记录](#版本更新记录)
- [开源协议](#开源协议)

---

## 功能特性
- 完整 **冬浩验证系统API封装** - 支持登录、注册、注销、心跳等所有核心接口
- 支持 **多种加密方式** - RC4加密/RSA加密/Base64编码/自定义加密
- 强大的 **MD5签名机制** - 确保请求安全性和数据完整性
- 完善 **超时重试机制** - 自动处理网络异常和服务器响应延迟
- 丰富的 **设备信息采集** - 获取机器码、硬件ID等信息用于设备绑定
- 全面的 **错误处理** - 提供详细的错误信息和调试支持
- 跨平台兼容性** - 同时支持Windows/Android平台

---

## 环境要求

- **Go 1.16+**
- 需要能访问冬浩验证系统的网络连接（用于API调用）

---

## 快速安装

```bash
go get github.com/yuan71058/DONGHAO-GO-SDK
```

---

## 快速开始
### 1. 基本登录示例

```go
package main

import (
    "fmt"
    "log"
    "github.com/yuan71058/DONGHAO-GO-SDK"
)

func main() {
    // 创建客户端实例（第二个参数是软件ID，从管理后台获取）
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

### 2. 卡密登录示例

```go
// 使用卡密进行免账号密码登录（需后台开启"允许卡密登录"选项，即dl_type=1）
result, err := client.LoginCard("YKUvGvYSuVFFTp41T1WO", "1.0", "mac123", "192.168.1.1", "client001")
if err != nil {
    log.Fatal(err)
}

if result.IsSuccess() {
    fmt.Println("卡密登录成功，Token:", client.GetToken())
    fmt.Println("当前用户:", client.GetCurrentUser())
} else {
    fmt.Println("卡密登录失败:", result.Msg())
}
```

---

## 客户端配置

### 创建客户端
```go
client := donghao.NewClient(baseURL string, appID int)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| baseURL | string | API服务端地址，例如 https://your-domain.com |
| appID | int | 软件ID，从管理后台获取 |

### 配置方法

| 方法 | 参数 | 说明 |
|------|------|------|
| `SetTimeout(seconds int)` | 超时秒数 | 设置HTTP请求超时时间，默认30秒 |
| `SetEncryption(encType int, key string)` | 加密类型, 密钥 | 设置数据加密方式和密钥 |
| `SetSignConfig(appKey, template string, needSign bool)` | 应用密钥, 签名模板, 是否签名 | 设置请求签名配置 |
| `SetHeartbeatInterval(seconds int)` | 心跳间隔秒数 | 设置自动心跳间隔时间，默认60秒（仅StartAutoHeartbeat生效） |

### 加密类型常量定义

```go
const (
    ENC_NONE   = 0  // 不加密，明文传输
    ENC_RC4    = 1  // RC4对称加密
    ENC_RSA    = 2  // RSA非对称加密
    ENC_BASE64 = 3  // Base64编码
    ENC_CUSTOM = 4  // 自定义加密方式
)
```

---

## 设备信息获取

SDK提供跨平台的设备信息采集功能，支持Windows和Android系统。
### 获取机器码
```go
// 获取当前设备的唯一机器码（可能出错）
machineCode, err := donghao.GetMachineCode()
if err != nil {
    // 如果获取失败，使用备用方法生成机器码
    machineCode = donghao.GetMachineCodeSafe()
}
fmt.Println("机器码:", machineCode)
// 输出格式: XXXX-XXXX-XXXX-XXXX
```

### 获取硬件ID

```go
// 获取当前设备的唯一硬件ID
hardwareID, err := donghao.GetHardwareID()
if err != nil {
    log.Fatal(err)
}
fmt.Println("硬件ID:", hardwareID)
```

### 获取硬件详细信息

```go
// 获取当前设备的详细硬件信息
hwInfo, err := donghao.GetHardwareInfo()
if err != nil {
    log.Fatal(err)
}
for key, value := range hwInfo {
    fmt.Printf("%s: %s\n", key, value)
}
```

### 生成随机客户端ID

```go
// 生成随机客户端ID，用于标识不同设备会话
clientID := donghao.GenerateClientID()
fmt.Println("客户端ID:", clientID)
```

### 设备信息函数说明

| 函数 | 返回值 | 平台支持 |
|------|--------|----------|
| `GetMachineCode()` | 当前设备的唯一机器码 | Windows/Android |
| `GetMachineCodeSafe()` | 备用方法获取机器码（不会报错） | 所有平台 |
| `GetHardwareID()` | 当前设备的唯一硬件ID | Windows/Android |
| `GetHardwareInfo()` | 当前设备的详细硬件信息 | Windows/Android |
| `GenerateClientID()` | 随机生成的客户端ID | 所有平台 |
| `GenerateDeviceID()` | 随机生成的设备ID | 所有平台 |

### 机器码生成算法说明
机器码基于以下信息通过**MD5哈希算法**生成：
```
Windows: 硬件信息(CPU+主板+磁盘+MAC+BIOS) -> MD5 -> 格式化(XXXX-XXXX-XXXX-XXXX)
Android: 设备信息 -> MD5 -> 格式化(XXXX-XXXX-XXXX-XXXX)
```

---

## API文档

### 统一返回结果结构体

```go
type Result struct {
    Code   int         // 返回状态码: 200/1=成功
    Result interface{} // 返回的结果内容
    Data   interface{} // 返回的业务数据
    Token  string      // Token值
}

func (r *Result) IsSuccess() bool      // 判断请求是否成功
func (r *Result) Msg() string          // 获取返回的错误消息
func (r *Result) GetTokenID() string   // 获取TokenID
func (r *Result) GetData() (string, error)         // 获取返回的用户数据（Base64解码后）
func (r *Result) GetGroupData() (string, error)    // 获取返回的分组数据（Base64解码后）
func (r *Result) GetVariableValue() (string, error) // 获取返回的变量值（Base64解码后）
```

### 用户相关接口

#### 用户登录
```go
result, err := client.Login(user, pwd, ver, mac, ip, clientid string)
```

#### 卡密登录
```go
result, err := client.LoginCard(card, ver, mac, ip, clientid string)
// 注意: 需要后台开启"允许卡密登录"选项(dl_type=1)
```

#### 用户注册
```go
result, err := client.Reg(user, pwd, card, userqq, email, tjr, ver, mac, ip, clientid string)
```

#### 心跳维持
```go
result, err := client.Heartbeat(user, tokenid, ver, mac, ip, clientid string)
```

#### 用户注销
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

#### 绑定新设备
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

### 用户数据相关接口

#### 获取用户数据
```go
result, err := client.GetUdata(user, tokenid, ver, mac, ip, clientid string)
data, err := result.GetData()  // Base64解码后的数据
```

#### 设置用户数据
```go
result, err := client.SetUdata(user, tokenid, udata, ver, mac, ip, clientid string)
```

#### 获取用户数据2
```go
result, err := client.GetUdata2(user, tokenid, ver, mac, ip, clientid string)
```

#### 设置用户数据2
```go
result, err := client.SetUdata2(user, tokenid, udata, ver, mac, ip, clientid string)
```

### 云变量操作接口
#### 获取云变量
```go
result, err := client.GetVariable(user, tokenid, cloudkey, ver, mac, ip, clientid string)
value, err := result.GetVariableValue()  // Base64解码后的变量值
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

### 云计算函数接口
#### 带用户信息的云计算函数调用
```go
result, err := client.Func(user, tokenid, fun, para, ver, mac, ip, clientid string)
```

#### 免登录云计算函数调用
```go
result, err := client.Func2(fun, para, ver, mac, ip, clientid string)
```

#### 调用PHP函数（需要登录）
```go
result, err := client.CallPHP(user, tokenid, fun, para, ver, mac, ip, clientid string)
```

#### 调用PHP函数（无需登录）
```go
result, err := client.CallPHP2(fun, para, ver, mac, ip, clientid string)
```

### 黑名单管理接口
#### 查询黑名单
```go
result, err := client.GetBlack(bType, bData string)
// bType: ip, mac, user
```

#### 添加黑名单
```go
result, err := client.SetBlack(bType, bData, bBz, ver, mac, ip, clientid string)
```

### 其他辅助接口

#### 扣除积分
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

#### 版本检查
```go
result, err := client.Ver(ver, mac, ip, clientid string)
```

#### 中继转发
```go
result, err := client.Relay(params map[string]string, ver, mac, ip, clientid string)
```

---

## 加密通信

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

### 明文传输
```go
client.SetEncryption(donghao.ENC_NONE, "")
```

---

## 自动心跳维持在线状态
> **重要提示**: 自动心跳维持在线功能目前需要手动调用`Heartbeat()` 方法实现。
```go
// 手动发送心跳维持在线
result, err := client.Heartbeat(user, tokenid, ver, mac, ip, clientid)
if err != nil {
    fmt.Println("心跳失败:", err)
}
```

---

## 错误处理

### 基础用法
```go
result, err := client.Login(user, pwd, ver, mac, ip, clientid)

if err != nil {
    // 处理网络错误或参数错误
    fmt.Println("发生错误:", err)
    return
}

if result.IsSuccess() {
    // 成功 (code == 200 || code == 1)
    fmt.Println("成功:", result.Data)
} else {
    // 业务逻辑错误
    fmt.Println("错误码:", result.Code)
    fmt.Println("错误信息:", result.Msg())
}
```

### 错误码说明

| 错误码 | 含义 |
|--------|------|
| 200/1 | 成功 |
| -1 | 请求失败/网络错误 |
| 其他 | 具体的业务错误（请参考Go SDK文档）|

---

## 完整示例

```go
package main

import (
    "fmt"
    "log"
    "github.com/yuan71058/DONGHAO-GO-SDK"
)

func main() {
    client := donghao.NewClient("http://your-domain.com", 1)
    client.SetTimeout(30)
    
    // 1. 获取机器码
    machineCode, err := donghao.GetMachineCode()
    if err != nil {
        machineCode = donghao.GetMachineCodeSafe()
    }
    
    // 2. 生成客户端ID
    clientID := donghao.GenerateClientID()
    
    user := "testuser"
    pwd := "testpwd"
    ver := "1.0.0"
    mac := machineCode
    ip := "192.168.1.100"
    
    // 3. 用户登录
    result, err := client.Login(user, pwd, ver, mac, ip, clientID)
    if err != nil || !result.IsSuccess() {
        log.Fatal("登录失败:", result.Msg())
    }
    fmt.Println("登录成功")
    token := client.GetToken()
    
    // 4. 发送心跳维持在线状态
    heartResult, _ := client.Heartbeat(user, token, ver, mac, ip, clientID)
    fmt.Println("心跳结果:", heartResult.IsSuccess())
    
    // 5. 获取用户信息
    userResult, _ := client.GetUser(user, token, ver, mac, ip, clientID)
    if data, err := userResult.GetData(); err == nil {
        fmt.Println("用户数据:", data)
    }
    
    // 6. 获取云变量
    varResult, _ := client.GetVariable(user, token, "config", ver, mac, ip, clientID)
    if value, err := varResult.GetVariableValue(); err == nil {
        fmt.Println("云变量:", value)
    }
    
    // 7. 用户注销
    client.Logout(user, token, ver, mac, ip, clientID)
    fmt.Println("已退出登录")
}
```

完整示例代码请查看[example/main.go](example/main.go)

---

## 版本更新记录

| 版本 | 日期 | 更新说明 |
|------|------|----------|
| 1.0 | 2026-03-31 | 初始版本发布，支持完整的API接口 |

---

## 开源协议
本项目采用[MIT](LICENSE)开源协议，可自由使用和分发。

---

<div align="center">

**冬浩验证系统** - 让软件保护变得简单高效
</div>

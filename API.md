# 冬浩验证系统 - Go SDK API 文档

**版本**: 1.4  
**最后更新**: 2026-04-11  
**GitHub**: https://github.com/yuan71058/DONGHAO-GO-SDK

---

## 更新日志

### v1.4 (2026-04-11)

#### Bug 修复
- **修复 httpPost token 参数被无条件覆盖的问题**
  - 问题原因：`httpPost()` 在注入 `currentToken` 时，会覆盖 `Heartbeat()` 等方法手动传入的 `token` 参数
  - 修复方案：添加 `params["token"] == ""` 判断，只在参数中没有 token 时才自动注入

- **修复 heartbeatParams 字段命名误导**
  - 字段名 `TokenID` 改为 `Token`，与实际用途保持一致

#### 代码变更
```go
// httpPost - 修复前
if c.currentToken != "" {
    params["token"] = c.currentToken
}

// httpPost - 修复后
if c.currentToken != "" && params["token"] == "" {
    params["token"] = c.currentToken
}

// heartbeatParams - 修复前
type heartbeatParams struct {
    TokenID string // 登录Token
}

// heartbeatParams - 修复后
type heartbeatParams struct {
    Token string // 登录Token
}
```

---

### v1.3 (2026-04-10)

#### 重大变更：统一使用 token 参数
- **移除 tokenid 参数，统一使用 token**
  - 所有 API 接口参数名从 `tokenid` 改为 `token`
  - 服务端所有模块（login/heartbeat/logout/profiler/constant）统一使用 `token` 字段
  - `GetTokenID()` 方法只读取 `token` 字段，不再兼容旧版 `tokenid`

#### 变更文件
| 文件 | 变更内容 |
|------|----------|
| `login.php` | 返回字段 `tokenid` → `token` |
| `heartbeat.php` | 接收参数 `tokenid` → `token` |
| `logout.php` | 接收参数 `tokenid` → `token` |
| `profiler.php` | 接收参数 `tokenid` → `token` |
| `constant.php` | 接收参数 `tokenid` → `token` |
| `donghao.go` | 所有函数参数和请求参数改为 `token` |

---

### v1.2 (2026-04-10)

#### 重大变更：Token 认证机制升级
- **服务端改用随机 Token 替代数据库自增 ID**
  - 登录时服务端生成随机 Token（MD5 格式）并存储到 `heartbeat.token` 字段
  - 所有会话验证（心跳、注销、取常量等）改用 `token` 字段查询
  - 客户端无需修改，SDK 已正确处理 Token 轮换机制

#### 认证流程变化
```
之前: 登录返回数据库自增ID → 心跳使用ID查询
现在: 登录生成随机Token → 心跳使用Token查询
```

#### 优势
- **更安全**：Token 随机生成，不易被猜测
- **更灵活**：Token 可随时更换，不依赖数据库 ID
- **更兼容**：Token 格式统一，便于跨系统集成

---

### v1.1 (2026-04-10)

#### Bug 修复
- **修复登录后心跳失败的问题**
  - 问题原因：SDK 在解析响应时直接使用了顶层的 `token`（MD5 hash），而没有优先使用 `result.token`
  - 影响范围：所有加密模式下的登录和心跳功能
  - 修复方案：修改 `httpPost` 函数中的 Token 设置逻辑，优先使用 `result.token`

### v1.0 (2026-03-31)
- 初始版本发布
- 完整的 API 封装
- 支持多种加密方式
- 设备信息采集功能

---

## 目录

- [快速开始](#快速开始)
- [Client 客户端](#client-客户端)
- [Result 返回结果结构体](#result-返回结果结构体)
- [用户相关接口](#用户相关接口)
- [卡密相关接口](#卡密相关接口)
- [用户数据相关接口](#用户数据相关接口)
- [云变量操作接口](#云变量操作接口)
- [云计算函数接口](#云计算函数接口)
- [辅助功能接口](#辅助功能接口)
- [设备信息获取工具函数](#设备信息获取工具函数)
- [错误处理](#错误处理)

---

## 快速开始
### 安装

```bash
go get github.com/yuan71058/DONGHAO-GO-SDK
```

### 基本使用

```go
package main

import (
    "fmt"
    "log"
    "github.com/yuan71058/DONGHAO-GO-SDK"
)

func main() {
    // 创建客户端实例
    client := donghao.NewClient("http://your-domain.com", 1)
    client.SetTimeout(30)
    
    // 设置加密方式（可选）
    client.SetEncryption(donghao.ENC_RC4, "your-rc4-key")
    
    // 设置签名配置（可选）
    client.SetSignConfig("your-app-key", "[data]123[key]456", true)
    
    // 用户登录
    result, err := client.Login("username", "password", "1.0", "mac", "ip", "clientid")
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

---

## Client 客户端
### NewClient

创建一个新的API客户端实例。
```go
func NewClient(baseURL string, appID int) *Client
```

**参数**:
- `baseURL`: API服务端的基础URL，例如 `"http://your-domain.com"`
- `appID`: 软件ID，从管理后台获取

**返回值**:
- `*Client`: 客户端实例指针

**使用示例**:
```go
client := donghao.NewClient("http://your-domain.com", 1)
```

---

### SetTimeout

设置HTTP请求超时时间（单位：秒）。

```go
func (c *Client) SetTimeout(seconds int)
```

**参数**:
- `seconds`: 超时秒数，建议值30秒

**使用示例**:
```go
client.SetTimeout(30) // 设置30秒超时
```

---

### SetTimeoutMs

设置HTTP请求超时时间（单位：毫秒）。

```go
func (c *Client) SetTimeoutMs(ms int64)
```

**参数**:
- `ms`: 超时毫秒数，默认30000ms

**使用示例**:
```go
client.SetTimeoutMs(15000) // 设置15秒超时
```

---

### SetEncryption

设置数据加密方式和密钥。

```go
func (c *Client) SetEncryption(encType int, key string)
```

**参数**:
- `encType`: 加密类型 (`ENC_NONE`, `ENC_RC4`, `ENC_RSA`, `ENC_BASE64`, `ENC_CUSTOM`)
- `key`: 加密密钥

**使用示例**:
```go
client.SetEncryption(donghao.ENC_RC4, "qHBqZ-vsGzS-32oiT-trIpg-iJhR")
```

---

### SetSignConfig

设置请求签名配置。

```go
func (c *Client) SetSignConfig(appKey, template string, needSign bool)
```

**参数**:
- `appKey`: 应用签名密钥
- `template`: 签名模板，格式如 `"[data]123[key]456"`
- `needSign`: 是否启用签名验证

**使用示例**:
```go
client.SetSignConfig("hbjBpRbXqNnMwbS3", "[data]123[key]456", true)
```

---

### SetHeartbeatInterval

设置自动心跳维持在线的间隔时间。

```go
func (c *Client) SetHeartbeatInterval(seconds int)
```

**参数**:
- `seconds`: 心跳间隔时间，建议值60秒

**使用示例**:
```go
client.SetHeartbeatInterval(60)
```

---

### GetToken

获取当前用户的Token字符串。

```go
func (c *Client) GetToken() string
```

**返回值**:
- `string`: 当前登录会话的Token字符串，用于后续API调用的身份验证

---

### GetCurrentUser

获取当前登录的用户名。

```go
func (c *Client) GetCurrentUser() string
```

**返回值**:
- `string`: 当前登录的用户名字符串，如果未登录则返回空字符串

---

## Result 返回结果结构体
### IsSuccess

判断请求是否成功执行。

```go
func (r *Result) IsSuccess() bool
```

**返回值**:
- `bool`: true表示请求成功，false表示请求失败

---

### Msg

获取返回的错误消息信息。

```go
func (r *Result) Msg() string
```

**返回值**:
- `string**: 错误信息描述文本，包含服务器返回的具体错误原因

---

### GetResultMap

获取返回结果的map形式。

```go
func (r *Result) GetResultMap() map[string]interface{}
```

**返回值**:
- `map[string]interface{}`: 结果数据的map形式

---

### GetData

获取返回的用户数据（Base64解码后）。

```go
func (r *Result) GetData() (string, error)
```

**返回值**:
- `string`: Base64解码后的用户数据
- `error`: 解码过程中的错误

---

### GetGroupData

获取返回的分组数据（Base64解码后）。

```go
func (r *Result) GetGroupData() (string, error)
```

**返回值**:
- `string`: Base64解码后的分组数据
- `error`: 解码过程中的错误

---

### GetVariableValue

获取返回的变量值（Base64解码后）。

```go
func (r *Result) GetVariableValue() (string, error)
```

**返回值**:
- `string`: 云变量的值
- `error`: 解码过程中的错误

---

### GetTokenID

获取登录返回的 Token（从 result.token 字段读取）。

```go
func (r *Result) GetTokenID() string
```

**返回值**:
- `string`: 登录返回的随机 Token（MD5 格式，32位）

**说明**（v1.2+）：
- 此方法从 `result.token` 字段获取 Token
- Token 由服务端登录时生成，存储在 heartbeat 表的 token 字段
- 用于心跳、注销等需要会话验证的接口

---

## 用户相关接口

### Login

用户登录

```go
func (c *Client) Login(user, pwd, ver, mac, ip, clientid string) (*Result, error)
```

**参数**:
- `user`: 用户名
- `pwd`: 密码
- `ver`: 软件版本号
- `mac`: 设备机器码/MAC地址
- `ip`: 客户端IP地址
- `clientid`: 客户端ID

**返回值**:
- `*Result`: 登录结果
- `error`: 网络或参数错误

**使用示例**:
```go
result, err := client.Login("testuser", "password", "1.0", "00:11:22:33:44:55", "192.168.1.1", "client001")
if result.IsSuccess() {
    fmt.Println("登录成功，Token:", client.GetToken())
}
```

---

### LoginCard

卡密登录 - 使用卡密代替账号密码进行免密登录。

```go
func (c *Client) LoginCard(card, ver, mac, ip, clientid string) (*Result, error)
```

**参数**:
- `card`: 卡密
- `ver`: 软件版本号
- `mac`: 设备机器码/MAC地址
- `ip`: 客户端IP地址
- `clientid`: 客户端ID

**返回值**:
- `*Result`: 登录结果
- `error`: 网络或参数错误

**使用示例**:
```go
result, err := client.LoginCard("YKUvGvYSuVFFTp41T1WO", "1.0", "mac", "ip", "client001")
if result.IsSuccess() {
    fmt.Println("卡密登录成功，Token:", client.GetToken())
}
```

---

### Reg

用户注册

```go
func (c *Client) Reg(user, pwd, card, userqq, email, tjr, ver, mac, ip, clientid string) (*Result, error)
```

**参数**:
- `user`: 用户名
- `pwd`: 密码
- `card`: 注册所需卡密（可选）
- `userqq`: QQ号（可选）
- `email`: 电子邮箱地址（可选）
- `tjr`: 推荐人用户名（可选）
- `ver`: 软件版本号
- `mac`: 设备机器码
- `ip`: IP地址
- `clientid`: 客户端ID

**使用示例**:
```go
result, err := client.Reg("newuser", "password", "CARD-123", "123456", "user@example.com", "", "1.0", "mac", "ip", "client001")
```

---

### Logout

用户注销/退出登录。

```go
func (c *Client) Logout(user, token, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.Logout("testuser", client.GetToken(), "1.0", "mac", "ip", "client001")
```

---

### GetUser

获取用户详细信息。

```go
func (c *Client) GetUser(user, token, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.GetUser("testuser", client.GetToken(), "1.0", "mac", "ip", "client001")
if result.IsSuccess() {
    m := result.GetResultMap()
    if endtime, ok := m["endtime"].(string); ok {
        fmt.Println("到期时间:", endtime)
    }
}
```

---

### Uppwd

修改密码

```go
func (c *Client) Uppwd(user, pwd, newpwd, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.Uppwd("testuser", "oldpwd", "newpwd", "1.0", "mac", "ip", "client001")
```

---

### Heartbeat

发送心跳包

```go
func (c *Client) Heartbeat(user, token, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.Heartbeat("testuser", client.GetToken(), "1.0", "mac", "ip", "client001")
```

---

### StartAutoHeartbeat 启动自动心跳

启动自动心跳维持在线功能（基于 context.Context 实现）。

**函数签名**:
```go
func (c *Client) StartAutoHeartbeat(user, token, ver, mac, ip, clientid string) error
```

**参数说明**:
- `user`: 用户名，为空时自动使用 `currentUser`
- `token`: 登录 Token，为空时自动使用 `currentToken`
- `ver`: 软件版本号
- `mac`: 设备机器码
- `ip`: 客户端IP地址
- `clientid`: 客户端ID

**返回值**: `error` - 启动失败时返回错误

**使用示例**:
```go
// 基本用法：登录后启动自动心跳
err := client.StartAutoHeartbeat(
    "", // user: 为空自动填充
    "", // token: 为空自动填充
    "1.0.0",
    donghao.GetMachineCodeSafe(),
    donghao.GetLocalIP(),
    donghao.GenerateClientID(),
)

// 检查状态
if client.IsHeartbeatRunning() {
    fmt.Println("心跳运行中")
}

// 停止心跳
client.StopAutoHeartbeat()
```

**高级用法（带错误回调）**:
```go
err := client.StartAutoHeartbeatWithCallback(
    "",
    "",
    "1.0.0",
    mac,
    ip,
    clientID,
    func(err error, count int) {
        log.Printf("⚠️ 心跳连续失败(%d次): %v\n", count, err)
        if count >= 3 {
            log.Println("🔴 连续失败超过3次，建议重新登录！")
        }
    },
)
```

### StopAutoHeartbeat 停止自动心跳

停止后台运行的自动心跳 goroutine。

```go
func (c *Client) StopAutoHeartbeat()
```

**使用示例**:
```go
client.StopAutoHeartbeat()
fmt.Println("心跳已停止")
```

### IsHeartbeatRunning 检查心跳状态

检查当前是否有自动心跳在运行。

```go
func (c *Client) IsHeartbeatRunning() bool
```

**返回值**:
- `bool`: true 表示心跳正在运行

---

## 卡密相关接口

### Recharge

卡密充值

```go
func (c *Client) Recharge(user, card, ver, mac, ip, clientid string) (*Result, error)
```

**参数**:
- `user`: 用户名
- `card`: 卡密
- `ver`: 软件版本号
- `mac`: 设备机器码
- `ip`: IP地址
- `clientid`: 客户端ID

**使用示例**:
```go
result, err := client.Recharge("testuser", "YKUvGvYSuVFFTp41T1WO", "1.0", "mac", "ip", "client001")
if result.IsSuccess() {
    fmt.Println("充值成功")
}
```

---

### Binding

绑定新设备 - 将当前账号绑定到新的设备信息上（支持更换用户名、MAC、IP、QQ等）

```go
func (c *Client) Binding(user, pwd, newuser, newmac, newip, newuserqq, ver, mac, ip, clientid string) (*Result, error)
```

**参数**:
- `user`: 当前用户名
- `pwd`: 当前密码
- `newuser`: 新的用户名（如果要更改用户名则填写新用户名，否则留空）
- `newmac`: 新的设备MAC地址（如果要更换设备则填写新MAC地址，否则留空）
- `newip: 新的IP地址（如果要更换设备则填写新IP地址，否则留空）
- `newuserqq`: 新的QQ号（如果要更换则填写新QQ号，否则留空）
- `ver`: 软件版本号
- `mac`: 当前设备机器码
- `ip`: 当前客户端IP地址
- `clientid`: 客户端ID

**使用示例**:
```go
result, err := client.Binding("testuser", "password", "newuser", "newmac", "newip", "newqq", "1.0", "mac", "ip", "client001")
```

---

## 用户数据相关接口

### GetUdata

获取用户数据

```go
func (c *Client) GetUdata(user, token, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.GetUdata("testuser", client.GetToken(), "1.0", "mac", "ip", "client001")
if result.IsSuccess() {
    data, _ := result.GetData()
    fmt.Println("用户数据:", data)
}
```

---

### SetUdata

设置用户数据

```go
func (c *Client) SetUdata(user, token, udata, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
jsonData := `{"level": 5, "exp": 1000}`
result, err := client.SetUdata("testuser", client.GetToken(), jsonData, "1.0", "mac", "ip", "client001")
```

---

### GetUdata2

获取用户数据2（独立存储空间，与GetUdata互不影响）

```go
func (c *Client) GetUdata2(user, token, ver, mac, ip, clientid string) (*Result, error)
```

---

### SetUdata2

设置用户数据2（独立存储空间，与SetUdata互不影响）

```go
func (c *Client) SetUdata2(user, token, udata, ver, mac, ip, clientid string) (*Result, error)
```

---

## 云变量操作接口
### GetVariable

获取云变量

```go
func (c *Client) GetVariable(user, token, cloudkey, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.GetVariable("testuser", client.GetToken(), "config_key", "1.0", "mac", "ip", "client001")
if result.IsSuccess() {
    value, _ := result.GetVariableValue()
    fmt.Println("变量值:", value)
}
```

---

### SetVariable

设置云变量

```go
func (c *Client) SetVariable(user, token, cloudkey, cloudvalue, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.SetVariable("testuser", client.GetToken(), "config_key", `{"enabled":true}`, "1.0", "mac", "ip", "client001")
```

---

### DelVariable

删除云变量

```go
func (c *Client) DelVariable(user, token, cloudkey, ver, mac, ip, clientid string) (*Result, error)
```

---

### Constant

获取云常量（只读常量）

```go
func (c *Client) Constant(user, token, cloudkey, ver, mac, ip, clientid string) (*Result, error)
```

---

## 云计算函数接口
### Func

调用云计算函数 - 需要已登录状态才能调用。

```go
func (c *Client) Func(user, token, fun, para, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.Func("testuser", client.GetToken(), "jia", "1,2", "1.0", "mac", "ip", "client001")
if result.IsSuccess() {
    fmt.Println("计算结果:", result.Data)
}
```

---

### Func2

调用云计算函数 - 免登录方式调用云计算函数。

```go
func (c *Client) Func2(fun, para, ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.Func2("jia", "1,2", "1.0", "mac", "ip", "client001")
```

---

### CallPHP

调用PHP函数 - 需要已登录状态才能调用。

```go
func (c *Client) CallPHP(user, token, fun, para, ver, mac, ip, clientid string) (*Result, error)
```

---

### CallPHP2

调用PHP函数 - 免登录方式调用PHP函数。

```go
func (c *Client) CallPHP2(fun, para, ver, mac, ip, clientid string) (*Result, error)
```

---

## 辅助功能接口

### Init

软件初始化

```go
func (c *Client) Init(ver, mac, ip, clientid string) (*Result, error)
```

**使用示例**:
```go
result, err := client.Init("1.0", "mac", "ip", "client001")
if result.IsSuccess() {
    m := result.GetResultMap()
    if name, ok := m["name"].(string); ok {
        fmt.Println("软件名称:", name)
    }
}
```

---

### Notice

获取公告内容

```go
func (c *Client) Notice(title, ver, mac, ip, clientid string) (*Result, error)
```

---

### Ver

获取软件版本信息

```go
func (c *Client) Ver(ver, mac, ip, clientid string) (*Result, error)
```

---

### GetBlack

查询黑名单

```go
func (c *Client) GetBlack(bType, bData string) (*Result, error)
```

**参数**:
- `bType`: 黑名单类型(ip/mac/user)
- `bData`: 黑名单数据

**使用示例**:
```go
result, err := client.GetBlack("ip", "192.168.1.1")
```

---

### SetBlack

添加黑名单

```go
func (c *Client) SetBlack(bType, bData, bBz, ver, mac, ip, clientid string) (*Result, error)
```

---

### AddLog

添加日志

```go
func (c *Client) AddLog(user, infos, ver, mac, ip, clientid string) (*Result, error)
```

---

### DeductPoints

扣除积分

```go
func (c *Client) DeductPoints(user, token string, sl int, ver, mac, ip, clientid string) (*Result, error)
```

---

### Bindreferrer

绑定推荐人

```go
func (c *Client) Bindreferrer(user, pwd, tjr, ver, mac, ip, clientid string) (*Result, error)
```

---

### Relay

中继转发请求

```go
func (c *Client) Relay(params map[string]string, ver, mac, ip, clientid string) (*Result, error)
```

---

## 设备信息获取工具函数

### GetMachineCode

获取当前设备的唯一机器码。该函数基于**硬件信息**生成。

```go
func GetMachineCode() (string, error)
```

**返回值**:
- **Windows**: 收集CPU、主板序列号、磁盘序列号、MAC地址、BIOS信息等硬件特征信息
- **Android**: 收集设备型号、品牌、序列号、安卓ID等信息
- **其他平台**: 收集操作系统和架构信息
- 所有平台均会对收集到的硬件信息进行MD5哈希处理
- **最终结果通过MD5哈希算法生成唯一标识**

**生成算法流程**:
```
硬件信息 -> MD5(第一次哈希) -> 格式化 -> MD5(第二次哈希) -> 最终机器码
```

**返回值**:
- `string`: 格式化的机器码，固定格式为 `XXXX-XXXX-XXXX-XXXX`（16个字符，由4组4字符组成）
- `error`: 获取硬件信息过程中可能发生的错误

**使用示例**:
```go
code, err := donghao.GetMachineCode()
if err != nil {
    log.Fatal("获取机器码失败:", err)
}
fmt.Println("机器码:", code) // 输出格式: A3F9-B2C1-D8E5-A7B4
```

---

### GetHardwareID

获取当前设备的硬件ID，采用更复杂的算法生成。

```go
func GetHardwareID() (string, error)
```

**返回值**:
- **Windows**: 收集CPU、主板、磁盘、MAC地址、操作系统、BIOS ID等信息
- **Android**: 收集设备型号、品牌、序列号、制造商、硬件信息等
- **其他平台**: 收集操作系统和架构信息
- 会收集更多硬件细节信息以增强唯一性

**返回值**:
- `string`: 硬件ID，格式为 `HWID-XXXXXXXX-XXXXXXXX-XXXXXXXX-XXXXXXXX`
- `error`: 获取硬件信息过程中可能发生的错误

**使用示例**:
```go
hwid, err := donghao.GetHardwareID()
if err != nil {
    log.Fatal(err)
}
fmt.Println("硬件ID:", hwid)
```

---

### GetHardwareInfo

获取当前设备的详细硬件信息，返回键值对形式的详细信息。

```go
func GetHardwareInfo() (map[string]string, error)
```

**返回值**:
- **Windows**: 包含 cpu, board, disk, mac, bios, uuid
- **Android**: 包含 serialno, device, brand, model, board, manufacturer, hardware, android_id
- **其他平台**: 包含 os, arch, hostname, cpu_num
- 会收集更多硬件细节信息以增强唯一性

**返回值**:
- `map[string]string`: 硬件信息的键值对映射表
- `error`: 获取硬件信息过程中可能发生的错误

**使用示例**:
```go
info, err := donghao.GetHardwareInfo()
if err != nil {
    log.Fatal(err)
}
for key, value := range info {
    fmt.Printf("%s: %s\n", key, value)
}
```

---

### GetMachineCodeSafe

安全地获取机器码，不会抛出错误。当无法获取真实机器码时，会生成一个基于随机数的备用机器码。

```go
func GetMachineCodeSafe() string
```

**返回值**:
- `string`: 机器码字符串，格式为 `XXXX-XXXX-XXXX-XXXX`（16个字符）

**说明**:
- 此方法不会返回错误
- 当硬件信息获取失败时，会生成一个随机的备用机器码
- 适用于需要保证一定能获取到机器码的场景

**使用示例**:
```go
machineCode := donghao.GetMachineCodeSafe()
fmt.Println("机器码:", machineCode) // 总是能返回一个有效的机器码
```

---

### GenerateClientID

生成一个随机的客户端ID，用于标识不同的客户端会话或设备实例。

```go
func GenerateClientID() string
```

**返回值**:
- `string`: 随机生成的客户端ID字符串

**说明**:
- 每次调用都会生成一个新的唯一ID
- 用于在多次调用之间保持会话一致性
- 可用于区分不同的客户端连接

**使用示例**:
```go
clientID := donghao.GenerateClientID()
fmt.Println("客户端ID:", clientID) // 例如: a1b2c3d4e5f6g7h8
```

---

### GenerateDeviceID

生成一个随机的设备ID，用于标识不同的设备。

```go
func GenerateDeviceID() string
```

**返回值**:
- `string`: 随机生成的设备ID字符串

**说明**:
- 与GenerateClientID类似，但专门用于标识物理设备
- 可用于设备管理和追踪

**使用示例**:
```go
deviceID := donghao.GenerateDeviceID()
fmt.Println("设备ID:", deviceID)
```

---

## Token 认证机制（v1.2+）

### 概述

从 v1.2 版本开始，冬浩验证系统采用**随机 Token** 替代数据库自增 ID 进行会话认证。

### Token 流程

```
┌──────────┐    登录请求     ┌────────────┐
│   客户端  │ ──────────────> │   服务端    │
│          │                 │            │
│          │ <────────────── │ 生成随机Token│
│          │   返回Token      │ (MD5格式)   │
└────┬─────┘                 └──────┬─────┘
     │                               │
     │ 存储 currentToken             │ 存入heartbeat.token
     ▼                               ▼
┌──────────┐                 ┌────────────┐
│ 后续API  │ ── token参数──> │   验证Token  │
│ 调用     │                 │            │
└──────────┘                 └────────────┘
```

### SDK 自动管理

SDK 会自动处理 Token 的存储和传递：

1. **登录时自动保存**: `Login()` / `LoginCard()` 成功后自动将 Token 存入 `currentToken`
2. **心跳时自动传递**: `StartAutoHeartbeat("", "", ...)` 中 user/token 为空时自动填充
3. **手动获取**: 通过 `client.GetToken()` 获取当前 Token

### 使用示例

```go
// 登录 - 自动获取并保存 Token
result, err := client.LoginCard("CARD-KEY", "1.0", mac, ip, clientID)
if result.IsSuccess() {
    fmt.Println("Token:", client.GetToken()) // 自动获取
}

// 心跳 - 自动使用登录的 Token
err = client.StartAutoHeartbeat(
    "", // user: 空 → 自动填充 currentUser
    "", // token: 空 → 自动填充 currentToken
    "1.0", mac, ip, clientID,
)

// 手动调用 API - 显式传入 Token
result, err := client.GetUser("user", client.GetToken(), "1.0", mac, ip, clientID)
```

---

## 错误处理

### 基础错误处理模式

所有API调用都遵循统一的错误处理模式：

```go
result, err := client.SomeMethod(...)
if err != nil {
    // 处理网络错误、参数错误等底层错误
    log.Printf("发生错误: %v\n", err)
    return
}

if result.IsSuccess() {
    // 处理成功的业务逻辑
    fmt.Println("操作成功:", result.Data)
} else {
    // 处理业务逻辑层面的失败
    fmt.Println("操作失败:", result.Code, result.Msg())
}
```

### 常见错误类型

| 错误场景 | 处理方式 | 说明 |
|---------|---------|------|
| 网络连接失败 | 检查err是否为nil | 通常表示DNS解析失败、连接超时等 |
| HTTP请求超时 | 增加SetTimeout的超时时间 | 默认30秒可能不够 |
| 参数为空 | 确保所有必需参数都已传递 | 特别是tokenid等关键参数 |
| 服务器返回错误 | 检查result.Code和result.Msg() | 业务逻辑错误会在这里体现 |
| 数据解码失败 | 检查GetData等方法的error返回 | Base64解码可能失败 |

### Result结构体详解

```go
type Result struct {
    Code   int         // 状态码: 200/1=成功, 其他=失败
    Result interface{} // 原始返回结果
    Data   interface{} // 业务数据
    Token  string      // Token值（如果有）
}

// 判断是否成功
func (r *Result) IsSuccess() bool {
    return r.Code == 200 || r.Code == 1
}

// 获取错误信息
func (r *Result) Msg() string {
    // 从Result中提取错误消息
}
```

### 最佳实践建议

1. **始终检查error返回值** - 不要忽略任何可能的错误
2. **使用IsSuccess()判断业务结果** - 不要直接检查Code值
3. **合理设置超时时间** - 根据网络环境调整SetTimeout参数
4. **妥善处理Token失效** - 当收到401或类似错误时，应重新登录获取新Token
5. **日志记录** - 在生产环境中记录所有API调用和错误信息

---

## 版本历史

| 版本 | 日期 | 更新内容 |
|-----|------|----------|
| 1.0 | 2026-03-31 | 初始版本发布 |
| 1.0.1 | 2026-04-04 | 更新文档，修复编码问题 |

---

## 许可证

本项目采用 MIT 开源许可证。详见 [LICENSE](LICENSE) 文件。

# Access Control Security Audit Report v2

## 执行摘要

| 项目 | 详情 |
|------|------|
| **目标项目** | Hello-Java-Sec |
| **语言/框架** | Java 8 / Spring Boot 2.4.1 |
| **审计方法** | 六维度访问控制审计 (D1-D6) |
| **发现总数** | 20 个 |
| **严重程度分布** | Critical: 2, High: 9, Medium: 7, Low: 2 |

---

## 审计范围

### 攻击面概览

```
┌─────────────────────────────────────────────────────────────────┐
│                    认证拦截器配置                                │
├─────────────────────────────────────────────────────────────────┤
│  拦截路径: /**                                                   │
│  排除路径:                                                       │
│    - /user/login, /user/ldap, /login (登录相关)                 │
│    - /css/**, /js/**, /img/**, /video/** (静态资源)              │
│    - /vulnapi/unauth/** ⚠️ (未授权API - 高风险)                  │
│    - /captcha (验证码)                                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## D1: 认证边界 (Authentication Boundary)

### Finding 1: 未授权API暴露敏感PII数据

- **Dimension:** D1
- **Severity:** High
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/unauth/api/info`

#### Protected Target
- 个人身份信息 (PII): 姓名、身份证号

#### Expected Control
- 敏感数据访问应要求认证

#### Actual Control
- **无。** 路径被显式排除在认证拦截器之外

#### Evidence
1. `MvcConfig.java:57`:
```java
.excludePathPatterns("/user/login", ..., "/vulnapi/unauth/**", "/captcha");
```
2. `Unauth.java:17-25`:
```java
@GetMapping("/api/info")
public String vul() {
    Map<String, String> m = new HashMap<>();
    m.put("name", "zhangwei");
    m.put("card", "130684199512173416");  // 身份证号泄露
    return JSON.toJSONString(m);
}
```

#### Exploitability Reasoning
- 任何未认证用户可直接访问，获取敏感个人信息

#### Verdict
- Confirmed

#### Recommended Fix
- 从排除路径中移除 `/vulnapi/unauth/**`，或在端点级别添加认证要求

---

### Finding 2: JWT密钥硬编码且过弱

- **Dimension:** D1
- **Severity:** High
- **Confidence:** Confirmed

#### Entry Point
- JWT令牌验证机制

#### Protected Target
- 用户认证令牌

#### Expected Control
- 强随机密钥，安全存储

#### Actual Control
- 硬编码弱密钥 `"123456"`

#### Evidence
1. `JwtUtils.java:20`:
```java
private static final String SECRET = "123456";
```

#### Exploitability Reasoning
- 攻击者可使用已知密钥伪造任意用户的JWT令牌，绕过认证

#### Verdict
- Confirmed

#### Recommended Fix
- 使用强随机密钥(256+位)，存储于环境变量或密钥管理系统

---

### Finding 3: CORS配置不当允许凭证窃取

- **Dimension:** D1
- **Severity:** High
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/cors/vul`

#### Protected Target
- 所有会话数据

#### Expected Control
- CORS应限制允许的源

#### Actual Control
- 反射任意Origin头，允许携带凭证

#### Evidence
1. `CORS.java:23-31`:
```java
@GetMapping("/vul")
public String corsVul(HttpServletRequest request, HttpServletResponse response) {
    String origin = request.getHeader("origin");
    response.setHeader("Access-Control-Allow-Origin", origin);  // 反射任意origin
    response.setHeader("Access-Control-Allow-Credentials", "true");  // 允许cookie
    return "cors vul";
}
```

#### Exploitability Reasoning
- 攻击者可托管恶意站点，发起带凭证的跨域请求，读取响应窃取会话数据

#### Verdict
- Confirmed

#### Recommended Fix
- 使用Origin白名单而非反射

---

### Finding 4: 开放重定向漏洞

- **Dimension:** D1
- **Severity:** Medium
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/redirect/vul?url={url}`
- `GET /vulnapi/redirect/vul2?url={url}`
- `GET /vulnapi/redirect/vul3?url={url}`

#### Protected Target
- 登录后重定向流程

#### Expected Control
- 仅重定向到白名单URL

#### Actual Control
- 无验证，重定向到任意URL

#### Evidence
1. `Redirect.java:30-34`:
```java
@GetMapping("/vul")
public String vul(String url) {
    return "redirect:" + url;  // 无验证
}
```

#### Exploitability Reasoning
- 可用于钓鱼攻击，URL看似来自可信域名

#### Verdict
- Confirmed

#### Recommended Fix
- 实施URL白名单验证

---

## D2: 功能级授权 (Function-Level Authorization)

### Finding 5: RCE端点无角色验证

- **Dimension:** D2
- **Severity:** Critical
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/RCE/Runtime/vul?cmd={command}`
- `GET /vulnapi/RCE/ProcessBuilder/vul?filepath={path}`

#### Protected Target
- 操作系统命令执行

#### Expected Control
- 仅管理员访问，强输入验证

#### Actual Control
- 仅需认证，无角色检查

#### Evidence
1. `RuntimeVul.java:23-43`:
```java
@RequestMapping("/vul")
public static String vul(String cmd) {
    Process proc = Runtime.getRuntime().exec(cmd);  // 执行任意命令
    // ...
}
```

#### Exploitability Reasoning
- 任何已认证用户可在服务器上执行任意系统命令

#### Verdict
- Confirmed

#### Recommended Fix
- 限制RCE功能仅管理员可用，添加角色验证

---

### Finding 6: 文件上传无权限控制

- **Dimension:** D2
- **Severity:** High
- **Confidence:** Confirmed

#### Entry Point
- `POST /vulnapi/upload/uploadVul`
- `POST /vulnapi/upload/uploadVul2`

#### Protected Target
- 文件系统写入能力

#### Expected Control
- 管理员或受限用户上传，强文件类型验证

#### Actual Control
- 仅需认证，任意用户可上传任意文件

#### Evidence
1. `Upload.java:29-55`:
```java
@PostMapping("/uploadVul")
public String uploadVul(@RequestParam("file") MultipartFile file, ...) {
    Files.write(path, bytes);  // 无类型验证，无角色检查
}
```

#### Exploitability Reasoning
- 已认证用户可上传WebShell，可能导致RCE

#### Verdict
- Confirmed

#### Recommended Fix
- 实施角色上传权限，严格文件类型白名单

---

### Finding 7: Admin系统信息泄露

- **Dimension:** D2
- **Severity:** Medium
- **Confidence:** Confirmed

#### Entry Point
- `GET /admin/sysinfo`

#### Protected Target
- 系统版本信息

#### Expected Control
- 仅管理员访问

#### Actual Control
- 需认证但无角色验证

#### Evidence
1. `Admin.java:24-38`:
```java
@GetMapping("/sysinfo")
@ResponseBody
public String sysInfo() {
    m.put("tomcat_version", ServerInfo.getServerInfo());
    m.put("java_version", System.getProperty("java.version"));
    m.put("fastjson_version", JSON.VERSION);  // 1.2.41 有已知RCE漏洞
    // 无角色检查
}
```

#### Exploitability Reasoning
- 版本信息有助于识别已知漏洞(Fastjson 1.2.41存在RCE)

#### Verdict
- Confirmed

#### Recommended Fix
- 添加管理员角色验证

---

### Finding 8: Actuator端点完全暴露

- **Dimension:** D2
- **Severity:** High
- **Confidence:** Confirmed

#### Entry Point
- `GET /actuator/**`

#### Protected Target
- Spring Boot Actuator端点

#### Expected Control
- 管理员访问或禁用敏感端点

#### Actual Control
- 配置暴露所有端点

#### Evidence
1. `application.properties:4`:
```
management.endpoints.web.exposure.include=*
```
2. `pom.xml:106-108`: Jolokia已启用，可导致RCE

#### Exploitability Reasoning
- 暴露的actuator端点泄露配置，Jolokia可导致RCE

#### Verdict
- Confirmed

#### Recommended Fix
- 限制actuator端点仅管理员访问或禁用

---

### Finding 9: Swagger API文档暴露

- **Dimension:** D2
- **Severity:** Low
- **Confidence:** Confirmed

#### Entry Point
- `GET /swagger-ui.html`
- `GET /v2/api-docs`

#### Protected Target
- 完整API文档

#### Expected Control
- 生产环境禁用或管理员访问

#### Actual Control
- 公开可访问

#### Evidence
1. `Swagger2.java:26-33`: 对controller包所有路径启用Swagger

#### Exploitability Reasoning
- 攻击者可发现所有API端点包括漏洞端点

#### Verdict
- Confirmed

#### Recommended Fix
- 生产环境禁用Swagger或添加认证

---

## D3: 对象级授权 (Object-Level Authorization)

### Finding 10: 水平越权 - 用户数据查询无所有权验证

- **Dimension:** D3
- **Severity:** High
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/IDOR/vul/info?name={username}`
- `GET /vulnapi/IDOR/vul/userid?id={id}`

#### Protected Target
- 用户记录

#### Expected Control
- 用户仅能访问自己的数据

#### Actual Control
- 无所有权验证，可查询任意用户

#### Evidence
1. `IDOR1.java:32-36`:
```java
@GetMapping("/vul/info")
public List<UserIDOR> vul(String name) {
    return userMapper.queryByUser2(name);  // 返回任意用户数据
}
```
2. `UserMapper.java:33-37`: 使用可枚举的整数ID

#### Exploitability Reasoning
- 已认证用户可遍历ID或猜测用户名获取任意用户数据

#### Verdict
- Confirmed

#### Recommended Fix
- 返回数据前验证session用户与资源所有者匹配

---

### Finding 11: 垂直越权 - 管理页面无角色检查

- **Dimension:** D3
- **Severity:** High
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/IDOR/vul/admin`

#### Protected Target
- 管理功能/页面

#### Expected Control
- 仅管理员访问

#### Actual Control
- 无任何授权检查

#### Evidence
1. `IDOR2.java:22-25`:
```java
@GetMapping(value = "/vul/admin")
public String vul() {
    return "idor/idoradmin";  // 直接返回管理页面
}
```

#### Exploitability Reasoning
- 任何已登录用户可访问管理页面

#### Verdict
- Confirmed

#### Recommended Fix
- 实施角色验证，检查用户是否为管理员

---

## D4: 业务上下文授权 (Business-Context Authorization)

### Finding 12: CSRF转账功能无保护

- **Dimension:** D4
- **Severity:** High
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/CSRF/transfer/vul?amount={}&receiver={}`

#### Protected Target
- 转账操作

#### Expected Control
- CSRF Token或至少POST方法

#### Actual Control
- GET端点无CSRF保护

#### Evidence
1. `CSRF.java:20-33`:
```java
@GetMapping("/transfer/vul")
public Map<String, Object> transferMoney(...) {
    String amount = request.getParameter("amount");
    String receiver = request.getParameter("receiver");
    // 无CSRF Token验证
    result.put("success", true);
    return result;
}
```

#### Exploitability Reasoning
- 攻击者可构造恶意页面，诱导已登录用户访问触发转账

#### Verdict
- Confirmed

#### Recommended Fix
- 使用POST方法和CSRF Token验证

---

### Finding 13: Referer头验证可绕过

- **Dimension:** D4
- **Severity:** Medium
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/CSRF/transfer/referer`

#### Protected Target
- 转账操作

#### Expected Control
- 服务端不可伪造的请求来源验证

#### Actual Control
- Referer头可被客户端控制

#### Evidence
1. `CSRF.java:43-48`:
```java
String referer = request.getHeader("referer");
if (referer == null || !referer.startsWith("http://baidu.com")) {
    // 拒绝
}
```

#### Exploitability Reasoning
- Referer可被伪造，或注册相似域名绕过

#### Verdict
- Confirmed

#### Recommended Fix
- 使用CSRF Token替代Referer验证

---

### Finding 14: 密码重置验证码前端回显

- **Dimension:** D4
- **Severity:** Critical
- **Confidence:** Confirmed

#### Entry Point
- `POST /vulnapi/LogicFlaw/PasswordReset/sendcode`

#### Protected Target
- 密码重置机制

#### Expected Control
- 验证码仅通过短信发送

#### Actual Control
- 验证码在API响应中返回

#### Evidence
1. `PasswordReset.java:42-47`:
```java
response.put("authCode", authCode);  // 验证码直接返回前端
return response;
```

#### Exploitability Reasoning
- 攻击者可调用接口获取任意用户的验证码

#### Verdict
- Confirmed

#### Recommended Fix
- 验证码仅通过可信渠道发送，不在响应中返回

---

### Finding 15: 验证码可暴力破解

- **Dimension:** D4
- **Severity:** High
- **Confidence:** Confirmed

#### Entry Point
- `POST /vulnapi/LogicFlaw/PasswordReset/reset2`

#### Protected Target
- 密码重置

#### Expected Control
- 强验证码、次数限制、过期时间

#### Actual Control
- 4位验证码，无次数限制，无过期

#### Evidence
1. `PasswordReset.java:131-133`:
```java
private String generateRandomCode2() {
    return String.format("%04d", new SecureRandom().nextInt(10000));  // 仅10000种可能
}
```

#### Exploitability Reasoning
- 4位验证码可在几分钟内暴力破解

#### Verdict
- Confirmed

#### Recommended Fix
- 使用6+位验证码，添加次数限制和过期时间

---

## D5: 客户端授权 (Client-Side Only Authorization)

### Finding 16: X-Forwarded-For头伪造绕过IP限制

- **Dimension:** D5
- **Severity:** Medium
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/IPForgery/vul`

#### Protected Target
- 本地IP限制的管理功能

#### Expected Control
- 服务端确定的客户端IP

#### Actual Control
- 使用客户端可控的X-Forwarded-For头

#### Evidence
1. `IPForgery.java:33-45`:
```java
String ip = (request.getHeader("X-Forwarded-For") != null) 
    ? request.getHeader("X-Forwarded-For") 
    : request.getRemoteAddr();
if (Objects.equals(ip, "127.0.0.1")) {
    m.put("flag", "...");  // 授权成功
}
```

#### Exploitability Reasoning
- 攻击者可添加X-Forwarded-For头绕过localhost检查

#### Verdict
- Confirmed

#### Recommended Fix
- 使用getRemoteAddr()或仅对可信代理验证X-Forwarded-For

---

## D6: 一致性检查 (Consistency Checks)

### Finding 17: 同一功能HTTP方法权限不一致

- **Dimension:** D6
- **Severity:** Medium
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/CSRF/transfer/vul` (无保护)
- `POST /vulnapi/CSRF/transfer/doTransferToken` (有CSRF保护)

#### Protected Target
- 转账功能

#### Expected Control
- 所有转账端点应有相同保护级别

#### Actual Control
- GET版本无保护，POST版本有保护

#### Evidence
1. 同一功能两个端点保护级别不一致

#### Exploitability Reasoning
- 攻击者可选择攻击无保护的GET端点

#### Verdict
- Confirmed

#### Recommended Fix
- 统一同一功能的安全策略

---

### Finding 18: IDOR vul/safe实现差异分析

- **Dimension:** D6
- **Severity:** N/A (对比分析)

#### Evidence

**vul版本 (`IDOR1.java:31-43`):**
```java
@GetMapping("/vul/info")
public List<UserIDOR> vul(String name) {
    return userMapper.queryByUser2(name);  // 无权限检查
}
```

**safe版本 (`IDOR1.java:46-58`):**
```java
@GetMapping("/safe/info")
public Object safe(String name, HttpSession session) {
    String loginUser = (String) session.getAttribute("LoginUser");
    if (loginUser.equals(name)) {  // 验证会话身份
        m.put("user", loginUser);
    }
}
```

#### 分析结论
- vul版本缺少会话身份验证
- safe版本正确验证当前用户身份

---

### Finding 19: CSRF vul/safe实现差异分析

- **Dimension:** D6
- **Severity:** N/A (对比分析)

#### Evidence

**vul版本 (`CSRF.java:20-33`):**
```java
@GetMapping("/transfer/vul")  // GET方法
// 无CSRF Token
```

**safe版本 (`CSRF.java:65-87`):**
```java
@PostMapping("/transfer/doTransferToken")  // POST方法
if (!token.equals(sessionToken)) {  // CSRF Token验证
    result.put("success", false);
}
```

#### 分析结论
- vul版本使用GET方法且无保护
- safe版本使用POST方法并实施CSRF Token

---

### Finding 20: 敏感操作使用错误的HTTP方法

- **Dimension:** D6
- **Severity:** Medium
- **Confidence:** Confirmed

#### Entry Point
- `GET /vulnapi/CSRF/transfer/vul`
- `GET /vulnapi/CSRF/transfer/referer`

#### Protected Target
- 转账操作

#### Expected Control
- 状态变更操作应使用POST/PUT/DELETE

#### Actual Control
- 使用GET方法

#### Evidence
1. `CSRF.java:20,36`:
```java
@GetMapping("/transfer/vul")  // 应使用@PostMapping
@GetMapping("/transfer/referer")  // 应使用@PostMapping
```

#### Exploitability Reasoning
- GET请求更易受CSRF攻击，会被浏览器历史/代理记录

#### Verdict
- Confirmed

#### Recommended Fix
- 敏感状态变更操作使用POST/PUT/DELETE方法

---

## 漏洞统计汇总

### 按维度统计

| 维度 | Critical | High | Medium | Low | 合计 |
|------|----------|------|--------|-----|------|
| D1 认证边界 | 0 | 3 | 1 | 0 | 4 |
| D2 功能授权 | 1 | 2 | 1 | 1 | 5 |
| D3 对象授权 | 0 | 2 | 0 | 0 | 2 |
| D4 业务上下文 | 1 | 2 | 1 | 0 | 4 |
| D5 客户端授权 | 0 | 0 | 1 | 0 | 1 |
| D6 一致性检查 | 0 | 0 | 2 | 0 | 2 |
| **总计** | **2** | **9** | **7** | **2** | **20** |

### 按严重程度统计

```
Critical ████████████████████ 2
High     ████████████████████████████████████████████████████████████████████████████ 9
Medium   ██████████████████████████████████████████████████████████ 7
Low      ████████████ 2
```

---

## 审计结论

### 整体评估

本项目是一个**故意设计漏洞的安全训练应用**。识别出的漏洞均为预期的教学案例。

### 关键发现

1. **无框架级授权**: 项目仅依赖session认证，未使用Spring Security等授权框架，授权检查在各控制器中手动实现

2. **授权模式不一致**: 部分端点有正确实现(safe版本)，部分缺少保护(vul版本)，形成对比教学

3. **高价值攻击面**:
   - RCE端点无角色验证 (Critical)
   - 密码重置验证码泄露 (Critical)
   - IDOR越权访问 (High)
   - Actuator完全暴露 (High)

### 修复优先级建议

| 优先级 | 漏洞类型 | 数量 |
|--------|----------|------|
| P0 | Critical级别漏洞 | 2 |
| P1 | High级别漏洞 | 9 |
| P2 | Medium级别漏洞 | 7 |
| P3 | Low级别漏洞 | 2 |

---

## 附录: vul/safe对比表

| 功能 | vul端点 | safe端点 | 差异 |
|------|---------|----------|------|
| IDOR查询 | `/vul/info` | `/safe/info` | session身份验证 |
| ID枚举 | `/vul/userid` | `/safe/userid` | UUID替代整数ID |
| 垂直越权 | `/vul/admin` | `/safe/admin` | 角色验证 |
| CSRF转账 | `/transfer/vul` | `/doTransferToken` | POST+CSRF Token |
| 目录遍历 | `/download` | `/download/safe` | 黑名单过滤 |
| 重定向 | `/redirect/vul` | `/redirect/safe` | 白名单验证 |

---

*报告生成时间: 2026-04-16*
*审计工具: Access Control Check Skill v2*

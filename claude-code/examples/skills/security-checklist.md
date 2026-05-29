> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: security-checklist
description: Web 应用安全检查清单（全面版）
effort: medium
---

# 安全检查清单 Skill

## 快速安全审计

### 认证（Authentication）
- [ ] 密码使用 bcrypt/argon2 哈希（cost factor >= 10）
- [ ] 会话 token 由密码学安全随机数生成
- [ ] JWT token 有效期较短（访问令牌 15 分钟，刷新令牌 7 天）
- [ ] 登录端点启用速率限制
- [ ] 多次失败后触发账户锁定

### 授权（Authorization）
- [ ] 每个 API 端点均检查权限
- [ ] 无 IDOR（不安全直接对象引用）
- [ ] 已实现基于角色的访问控制
- [ ] 敏感操作要求重新认证

### 输入验证
- [ ] 所有用户输入在服务端进行验证
- [ ] 文件上传限制类型和大小
- [ ] SQL 查询使用参数化语句
- [ ] HTML 输出经过编码以防止 XSS

### 数据保护
- [ ] 敏感数据静态加密
- [ ] 全面强制使用 HTTPS
- [ ] 安全 Cookie（HttpOnly、Secure、SameSite）
- [ ] URL 和日志中不含敏感数据

### 响应头与 CORS
- [ ] 已设置 Content-Security-Policy 响应头
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options: DENY（或 SAMEORIGIN）
- [ ] 已启用 Strict-Transport-Security
- [ ] CORS 已正确限制

## 代码模式

### SQL 注入防御
```javascript
// 有漏洞的写法
db.query(`SELECT * FROM users WHERE id = ${userId}`);

// 安全写法
db.query('SELECT * FROM users WHERE id = $1', [userId]);
```

### XSS 防御
```javascript
// 有漏洞的写法
element.innerHTML = userInput;

// 安全写法
element.textContent = userInput;

// 安全写法（带净化）
element.innerHTML = DOMPurify.sanitize(userInput);
```

### CSRF 防护
```javascript
// 生成 token
const csrfToken = crypto.randomBytes(32).toString('hex');
session.csrfToken = csrfToken;

// POST 时验证
if (req.body.csrf !== session.csrfToken) {
  throw new ForbiddenError('Invalid CSRF token');
}
```

### 密钥管理
```javascript
// 绝对不要硬编码在代码中
const API_KEY = 'sk-abc123...';

// 使用环境变量
const API_KEY = process.env.API_KEY;

// 使用密钥管理器（生产环境）
const secret = await secretsManager.getSecret('api-key');
```

## 安全响应头示例

```javascript
// Express 中间件
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  next();
});
```

## 依赖安全

```bash
# 检查漏洞
npm audit

# 自动修复可修复的问题
npm audit fix

# 检查过时的包
npm outdated

# 更新依赖
npm update
```

## 安全事件日志

```javascript
// 需要记录的事件
logger.security({
  event: 'login_failed',
  ip: req.ip,
  email: req.body.email,
  reason: 'invalid_password',
  timestamp: new Date().toISOString()
});

// 绝对不要记录
// - 密码
// - 完整信用卡号
// - 会话 token
// - 个人数据（生产环境）
```

## 上线前检查清单

1. [ ] 运行 `npm audit` — 无严重漏洞
2. [ ] 所有密钥存放在环境变量中
3. [ ] 调试模式已关闭
4. [ ] 错误信息不暴露内部实现
5. [ ] 仅 HTTPS（HTTP 重定向至 HTTPS）
6. [ ] 数据库凭据已轮换
7. [ ] 日志已配置（无敏感数据）
8. [ ] 备份策略已测试

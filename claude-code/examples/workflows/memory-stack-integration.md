> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "内存栈集成：真实的多日工作流"
description: "5 天冲刺示例，结合 claude-mem、Serena、grepai 和 rg 进行认证重构"
tags: [workflow, memory, integration]
---

# 内存栈集成：真实的多日工作流

**场景**：在 5 天冲刺中将认证系统从基于会话迁移到 JWT

**使用工具**：claude-mem（自动捕获）+ Serena（手动决策）+ grepai（发现）+ rg（精确搜索）

**项目**：电商 API（Node.js/TypeScript，5 万行代码，200+ 文件）

---

## 第 1 天：发现与规划

### 上午：初步探索

**用户**："让我们探索一下当前的认证系统"

**Claude**（尚未安装 claude-mem 时）：
```bash
# 手动发现
rg "authenticate|session" --type ts | head -20
# 47 个匹配项分布在 12 个文件中
```

**问题**：文件太多，对架构决策没有上下文。

---

### 安装 claude-mem 后

```bash
# 安装 claude-mem
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
# 重启 Claude Code
```

**用户**："探索认证架构"

**Claude**（现已启用自动捕获）：
```bash
# 1. 语义发现（grepai）
grepai search "authentication and session management"
# → 找到：
#   - src/auth/auth.service.ts（主服务）
#   - src/middleware/session.middleware.ts（会话处理）
#   - src/controllers/user.controller.ts（端点）
#   - config/session.config.ts（会话配置）

# 2. 结构概览（Serena）
serena get_symbols_overview --file src/auth/auth.service.ts
# → AuthService 类：
#   - login(email, password)
#   - logout(sessionId)
#   - validateSession(sessionId)
#   - createSession(userId)

# 3. 调用图（grepai）
grepai trace callers "createSession"
# → 被以下调用：
#   - UserController.login()（47 次）
#   - AdminController.authenticate()（12 次）
#   - OAuthController.callback()（8 次）

# 4. 精确搜索（rg）
rg "express-session" --type ts
# → 找到于：
#   - package.json（依赖项）
#   - session.middleware.ts（导入）
#   - app.ts（配置）
```

**claude-mem** 自动捕获：
- 读取了 4 个文件（auth.service.ts、session.middleware.ts、user.controller.ts、session.config.ts）
- 67 次工具调用（读取、搜索、追踪）
- 关键发现："当前系统使用 express-session，存储在 Redis 中"

---

### 下午：架构决策

**用户**："我们应该使用 JWT 还是继续用会话？"

**讨论**：
- JWT：无状态、可扩展、无 Redis 依赖
- 会话：服务器端控制、易于撤销
- **决策**：API 使用 JWT，Web 管理后台保留会话

**记录决策（Serena）**：
```bash
serena write_memory "auth_architecture" "$(cat <<'EOF'
Decision: Migrate API to JWT, keep sessions for web dashboard.

Rationale:
- API clients (mobile, SPA) benefit from stateless JWT
- Web dashboard needs session revocation (admin logout)
- Hybrid approach: /api/* uses JWT, /admin/* uses sessions

Library choice: jose (vs jsonwebtoken)
- Lighter (40% fewer dependencies)
- Better TypeScript support
- Active maintenance (last update: 2 weeks ago)

Next steps:
1. Install jose
2. Create JwtService
3. Update UserController to use JWT for /api/* routes
4. Keep SessionMiddleware for /admin/* routes
EOF
)"
```

**claude-mem** 自动捕获：
- 决策讨论（压缩摘要）
- Serena write_memory 调用
- JWT + jose 库的选用理由

---

### 第 1 天结束

**Claude** 自动生成会话摘要：
```
会话摘要（第 1 天）：
- 探索了认证系统（识别出 4 个关键文件）
- 当前：express-session + Redis
- 决策：API 迁移到 JWT，管理后台保留会话
- 库：jose（更轻量，更好的 TypeScript 支持）
- 下一步：安装 jose，创建 JwtService
```

**claude-mem 仪表盘**（`http://localhost:37777`）：
- 时间线：67 条观察记录
- 关键文件：auth.service.ts、session.middleware.ts
- 已记录决策：auth_architecture

---

## 第 2 天：开始实现

### 上午：会话恢复

**Claude**（自动注入上下文）：
```
📋 上一会话（第 1 天）：
- 探索了认证系统（4 个文件：auth.service.ts、session.middleware.ts、user.controller.ts、session.config.ts）
- 决策：API 使用 JWT，管理后台使用会话
- 库：jose（比 jsonwebtoken 轻 40%）
- 下一步：安装 jose，创建 JwtService
```

**用户**："让我们实现 JWT 服务"

**Claude**（已有上下文，无需重新读取）：
```bash
# 安装 jose
npm install jose

# 创建 JWT 服务
```

**实现**：
```typescript
// src/auth/jwt.service.ts
import * as jose from 'jose';

export class JwtService {
  private secret: Uint8Array;

  constructor(secretKey: string) {
    this.secret = new TextEncoder().encode(secretKey);
  }

  async generateToken(userId: string, email: string): Promise<string> {
    const jwt = await new jose.SignJWT({ userId, email })
      .setProtectedHeader({ alg: 'HS256' })
      .setIssuedAt()
      .setExpirationTime('24h')
      .sign(this.secret);

    return jwt;
  }

  async verifyToken(token: string): Promise<{ userId: string; email: string }> {
    const { payload } = await jose.jwtVerify(token, this.secret);
    return payload as { userId: string; email: string };
  }
}
```

**claude-mem** 自动捕获：
- npm install jose
- JwtService 实现
- 方法签名（generateToken、verifyToken）

---

### 下午：决策检查点

**用户**："我们现在应该添加刷新令牌还是以后再加？"

**讨论**：
- 刷新令牌 = 更高安全性（短期访问令牌）
- 但：更复杂，需要数据库存储
- **决策**：推迟到第 2 阶段（等基础 JWT 工作正常后）

**记录决策（Serena）**：
```bash
serena write_memory "jwt_refresh_tokens" "$(cat <<'EOF'
Decision: Defer refresh tokens to Phase 2

Rationale:
- Phase 1: Basic JWT (access tokens only, 24h expiry)
- Phase 2: Add refresh tokens (1 week expiry, stored in DB)

Why defer:
- Validate JWT flow works first
- Avoid premature complexity
- Need to design refresh token storage schema

Reminder for Phase 2:
- Refresh tokens in separate table (id, userId, token, expiresAt)
- Rotation on refresh (invalidate old, issue new)
- Endpoint: POST /api/auth/refresh
EOF
)"
```

**claude-mem** 自动捕获：
- 刷新令牌讨论
- 推迟决策
- 第 2 阶段提醒

---

### 第 2 天结束

**Claude** 自动生成会话摘要：
```
会话摘要（第 2 天）：
- 安装了 jose 库
- 实现了 JwtService（generateToken、verifyToken）
- 决策：将刷新令牌推迟到第 2 阶段
- 下一步：更新 UserController 以使用 JwtService
```

**内存栈状态**：

| 层级 | 内容 | 工具 |
|-------|---------|------|
| **会话捕获** | 第 1-2 天观察记录（共 134 条） | claude-mem |
| **手动决策** | 2 条记忆（auth_architecture、jwt_refresh_tokens） | Serena |
| **代码发现** | 4 个文件已索引，调用图已映射 | grepai |

---

## 第 3 天：集成

### 上午：会话恢复

**Claude**（自动注入上下文）：
```
📋 前几次会话（第 1-2 天）：
- 第 1 天：探索认证，决定 API 使用 JWT
- 第 2 天：实现了 JwtService，将刷新令牌推迟到第 2 阶段
- 已修改文件：jwt.service.ts
- 下一步：更新 UserController 以使用 JwtService
```

**用户**："更新登录端点以返回 JWT"

**Claude**（已了解上下文）：
```bash
# 1. 查找依赖关系（grepai）
grepai trace callers "login"
# → UserController.login() 被 47 个路由调用

# 2. 读取当前实现（rg + Read）
rg "async login" src/controllers/user.controller.ts -A 10
```

**当前实现**：
```typescript
// src/controllers/user.controller.ts（修改前）
async login(req: Request, res: Response) {
  const { email, password } = req.body;
  const user = await this.authService.authenticate(email, password);

  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // 旧方式：创建会话
  const sessionId = await this.authService.createSession(user.id);
  req.session.userId = user.id;

  return res.json({ success: true, sessionId });
}
```

**更新为 JWT**：
```typescript
// src/controllers/user.controller.ts（修改后）
import { JwtService } from '../auth/jwt.service';

async login(req: Request, res: Response) {
  const { email, password } = req.body;
  const user = await this.authService.authenticate(email, password);

  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // 新方式：生成 JWT
  const token = await this.jwtService.generateToken(user.id, user.email);

  return res.json({ success: true, token });
}
```

**claude-mem** 自动捕获：
- UserController 修改
- 移除：createSession 调用、会话存储
- 添加：JWT 生成

---

### 下午：中间件更新

**用户**："创建 JWT 认证中间件"

**实现**：
```typescript
// src/middleware/jwt.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { JwtService } from '../auth/jwt.service';

export function jwtAuthMiddleware(jwtService: JwtService) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const authHeader = req.headers.authorization;

    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ error: 'Missing or invalid token' });
    }

    const token = authHeader.slice(7); // 移除 "Bearer "

    try {
      const payload = await jwtService.verifyToken(token);
      req.user = payload; // 将用户信息附加到请求
      next();
    } catch (error) {
      return res.status(401).json({ error: 'Invalid or expired token' });
    }
  };
}
```

**应用到 API 路由**：
```typescript
// src/app.ts
import { jwtAuthMiddleware } from './middleware/jwt.middleware';

// 将 JWT 中间件应用到 /api/* 路由
app.use('/api/*', jwtAuthMiddleware(jwtService));

// 保留 /admin/* 路由的会话中间件
app.use('/admin/*', sessionMiddleware);
```

**claude-mem** 自动捕获：
- 新中间件：jwt.middleware.ts
- 路由拆分：/api/*（JWT）vs /admin/*（会话）
- 保持向后兼容性

---

### 第 3 天结束

**用户**："记录迁移状态"

**Serena 内存更新**：
```bash
serena write_memory "jwt_migration_status" "$(cat <<'EOF'
JWT Migration Status (Day 3):

✅ Completed:
- JwtService implemented (jose library)
- UserController.login() updated to return JWT
- jwtAuthMiddleware created and applied to /api/* routes
- Admin routes still use sessions (/admin/*)

📝 Files Modified:
- src/auth/jwt.service.ts (new)
- src/middleware/jwt.middleware.ts (new)
- src/controllers/user.controller.ts (updated)
- src/app.ts (route split)

🧪 Next Steps (Day 4):
1. Write tests for JwtService
2. Test login flow end-to-end
3. Update API clients to use JWT (frontend, mobile)
4. Monitor logs for any session-related errors

⚠️ Rollback Plan:
- Keep session code intact (not deleted)
- Can revert by changing app.ts route config
- Database unchanged (no schema migration needed)
EOF
)"
```

**claude-mem 仪表盘** 显示：
- 3 天历史记录
- 共 217 条观察记录
- 3 条 Serena 记忆（架构、刷新令牌、迁移状态）
- 8 个文件已修改

---

## 第 4 天：测试与验证

### 上午：会话恢复

**Claude**（自动注入上下文）：
```
📋 前几次会话（第 1-3 天）：
- 第 1 天：探索认证，决定使用 JWT
- 第 2 天：实现了 JwtService
- 第 3 天：更新了登录端点，创建了 JWT 中间件
- 状态：/api/* 使用 JWT，/admin/* 使用会话
- 下一步：编写测试，端到端验证
```

**用户**："为 JwtService 编写测试"

**Claude**（已知第 2 天的实现）：
```typescript
// src/auth/__tests__/jwt.service.test.ts
import { JwtService } from '../jwt.service';

describe('JwtService', () => {
  let jwtService: JwtService;

  beforeEach(() => {
    jwtService = new JwtService('test-secret-key-32-chars-long!!');
  });

  describe('generateToken', () => {
    it('should generate valid JWT token', async () => {
      const token = await jwtService.generateToken('user-123', 'test@example.com');

      expect(token).toBeDefined();
      expect(typeof token).toBe('string');
      expect(token.split('.')).toHaveLength(3); // JWT format: header.payload.signature
    });

    it('should include userId and email in payload', async () => {
      const token = await jwtService.generateToken('user-456', 'user@example.com');
      const payload = await jwtService.verifyToken(token);

      expect(payload.userId).toBe('user-456');
      expect(payload.email).toBe('user@example.com');
    });

    it('should set 24h expiration', async () => {
      const token = await jwtService.generateToken('user-789', 'exp@example.com');
      const payload = await jwtService.verifyToken(token);

      // JWT exp is in seconds, not milliseconds
      const now = Math.floor(Date.now() / 1000);
      const expectedExp = now + (24 * 60 * 60); // 24 hours

      expect(payload.exp).toBeGreaterThan(now);
      expect(payload.exp).toBeLessThanOrEqual(expectedExp + 10); // 10s tolerance
    });
  });

  describe('verifyToken', () => {
    it('should verify valid token', async () => {
      const token = await jwtService.generateToken('user-abc', 'valid@example.com');
      const payload = await jwtService.verifyToken(token);

      expect(payload).toEqual({
        userId: 'user-abc',
        email: 'valid@example.com',
        iat: expect.any(Number),
        exp: expect.any(Number)
      });
    });

    it('should throw on invalid token', async () => {
      await expect(jwtService.verifyToken('invalid.token.here'))
        .rejects.toThrow();
    });

    it('should throw on expired token', async () => {
      // Create token with -1h expiration (already expired)
      const jwtService2 = new JwtService('test-secret');
      const expiredToken = await jwtService2.generateToken('user-old', 'old@example.com');

      // Wait 2 seconds (simulate expiration)
      await new Promise(resolve => setTimeout(resolve, 2000));

      await expect(jwtService2.verifyToken(expiredToken))
        .rejects.toThrow('expired');
    });

    it('should throw on token with wrong secret', async () => {
      const token = await jwtService.generateToken('user-123', 'test@example.com');

      // Try to verify with different secret
      const jwtService2 = new JwtService('different-secret-key-here!!!!');
      await expect(jwtService2.verifyToken(token))
        .rejects.toThrow();
    });
  });
});
```

**运行测试**：
```bash
npm test jwt.service.test.ts

# 输出：
# JwtService
#   generateToken
#     ✓ should generate valid JWT token (45ms)
#     ✓ should include userId and email in payload (32ms)
#     ✓ should set 24h expiration (28ms)
#   verifyToken
#     ✓ should verify valid token (21ms)
#     ✓ should throw on invalid token (15ms)
#     ✓ should throw on expired token (2018ms)
#     ✓ should throw on token with wrong secret (19ms)
#
# Tests: 7 passed, 7 total
```

**claude-mem** 自动捕获：
- 测试文件已创建（jwt.service.test.ts）
- 7 个测试用例（全部通过）
- 已测试边界情况（无效令牌、过期令牌、错误密钥）

---

### 下午：端到端测试

**用户**："测试完整的登录流程"

**手动端到端测试**：
```bash
# 1. 登录（获取 JWT）
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# 响应：
# {
#   "success": true,
#   "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiJ1c2VyLTEyMyIsImVtYWlsIjoidGVzdEBleGFtcGxlLmNvbSIsImlhdCI6MTcwNzU4MjAwMCwiZXhwIjoxNzA3NjY4NDAwfQ.xYz..."
# }

# 2. 访问受保护的 API 端点
curl -X GET http://localhost:3000/api/profile \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# 响应：
# {
#   "userId": "user-123",
#   "email": "test@example.com",
#   "profile": { ... }
# }

# 3. 不带令牌测试（应该失败）
curl -X GET http://localhost:3000/api/profile

# 响应：
# { "error": "Missing or invalid token" }

# 4. 测试管理员路由（应该仍使用会话）
curl -X GET http://localhost:3000/admin/dashboard \
  -H "Cookie: connect.sid=s%3A..."

# 响应：
# { "dashboard": "admin data" }
```

**所有测试通过！**

**记录验证结果（Serena）**：
```bash
serena write_memory "jwt_validation_results" "$(cat <<'EOF'
JWT Validation Results (Day 4):

✅ Unit Tests: 7/7 passed
- Token generation works
- Token verification works
- Expiration handling correct
- Error cases handled (invalid, expired, wrong secret)

✅ E2E Tests: 4/4 passed
- Login returns JWT
- Protected routes accept JWT
- Unauthorized requests rejected
- Admin routes still use sessions (backward compatibility)

📊 Performance:
- Token generation: ~30ms
- Token verification: ~20ms
- No Redis dependency (stateless)

⚠️ Observations:
- Frontend needs update to store JWT (localStorage)
- Mobile app needs update to send Authorization header
- Consider token refresh after 24h (Phase 2)

✅ Ready for deployment
EOF
)"
```

---

### 第 4 天结束

**内存搜索演示**：

```bash
# 用户搜索会话历史
"Search my memory for JWT decisions"
```

**claude-mem 返回**：
```
🔍 找到 3 个相关会话：

会话 1（第 1 天）：
- 决策：API 使用 JWT，管理后台使用会话
- 库：jose（轻 40%）

会话 2（第 2 天）：
- 实现了 JwtService
- 将刷新令牌推迟到第 2 阶段

会话 4（第 4 天）：
- 所有测试通过（7/7 单元测试，4/4 端到端测试）
- 已准备好部署
```

**内存栈最终状态**：

| 层级 | 内容 | 工具 |
|-------|---------|------|
| **会话捕获** | 4 天，312 条观察记录 | claude-mem |
| **手动决策** | 4 条记忆（架构、刷新令牌、迁移状态、验证结果） | Serena |
| **代码发现** | 8 个文件已修改，调用图已更新 | grepai |

---

## 第 5 天：部署

### 上午：部署前检查

**用户**："这周我们完成了什么？"

**Claude**（查询 claude-mem）：
```
"Show summary of JWT migration project"
```

**claude-mem 综合摘要**：
```
📋 JWT 迁移项目摘要（5 天）：

第 1 天：发现与规划
- 探索了认证系统（4 个关键文件）
- 决策：API 使用 JWT，管理后台使用会话
- 库：jose（轻 40%，更好的 TypeScript 支持）

第 2 天：实现
- 安装了 jose
- 创建了 JwtService（generateToken、verifyToken）
- 将刷新令牌推迟到第 2 阶段

第 3 天：集成
- 更新了 UserController.login()
- 创建了 jwtAuthMiddleware
- 应用于 /api/*，保留 /admin/* 的会话

第 4 天：测试
- 单元测试：7/7 通过
- 端到端测试：4/4 通过
- 验证完成

📊 影响：
- 已修改文件：8 个
- 工具调用：312 次
- 已移除依赖：express-session、connect-redis
- 已添加依赖：jose
- 性能：无状态（无 Redis 查询）

已准备好部署
```

**可用的 Serena 记忆**：
```bash
serena list_memories

# 输出：
# 1. auth_architecture（第 1 天）
# 2. jwt_refresh_tokens（第 2 天）
# 3. jwt_migration_status（第 3 天）
# 4. jwt_validation_results（第 4 天）
```

---

### 下午：部署与交接

**用户**："部署到预发布环境"

```bash
# 部署到预发布环境
git checkout -b feature/jwt-migration
git add .
git commit -m "feat: migrate API authentication to JWT

- Replace express-session with JWT (jose library)
- Apply JWT to /api/* routes
- Keep sessions for /admin/* routes
- Add comprehensive tests (7 unit, 4 E2E)
- All tests passing

Breaking changes: None (backward compatible)
Migration: None required (no DB changes)
Rollback: Change app.ts route config"

git push origin feature/jwt-migration

# 创建 PR
gh pr create --title "Migrate API to JWT authentication" \
  --body "$(serena read_memory jwt_migration_status)"
```

**交接文档**（从 Serena 记忆自动生成）：
```bash
# 从 Serena 记忆生成交接文档
serena read_memory auth_architecture > docs/jwt-migration.md
serena read_memory jwt_migration_status >> docs/jwt-migration.md
serena read_memory jwt_validation_results >> docs/jwt-migration.md
```

**结果**：完整记录了架构决策、实现细节和验证结果。

---

## 关键收获

### 内存栈的实际作用

**没有内存栈时**：
- 每天重新读取文件（约 200 个文件 × 5 天 = 1000 次读取）
- 会话之间丢失上下文
- 忘记架构决策
- 手动记录决策

**有了内存栈（claude-mem + Serena + grepai）后**：
- 每天早上自动注入上下文（0 次重新读取）
- 保留架构决策（4 条 Serena 记忆）
- 维护调用图（grepai 追踪）
- 完整的会话历史（312 条观察记录）
- **结果**：5 天项目以完整的上下文保留顺利完成

---

### Token 效率

| 指标 | 无内存栈 | 有内存栈 | 节省 |
|--------|---------|-------------------|---------|
| **文件读取** | 1000 次 | 150 次（减少 85%） | 850 次读取 |
| **输入 token** | 约 50 万 | 约 7.5 万（减少 85%） | 42.5 万 token |
| **上下文丢失** | 每次会话 | 从不 | 100% 保留 |
| **决策召回** | 手动笔记 | 4 条 Serena 记忆 | 即时获取 |

**渐进式披露的影响**：
- 第 5 天摘要查询：3 层（搜索 → 时间线 → 详情）
- 不使用时：加载所有 312 条观察记录（约 5 万 token）
- 使用时：加载摘要 → 5 个会话 → 1 个详情（约 5000 token）
- **节省**：90% 的 token

---

### 实用洞察

**各工具的适用场景**：

1. **claude-mem**（自动）：
   - 会话开始/结束（自动）
   - "我们昨天做了什么？"
   - "显示完整的项目历史"

2. **Serena**（手动，高价值）：
   - 架构决策
   - 库选择
   - 迁移状态
   - 回滚方案

3. **grepai**（语义发现）：
   - "找认证相关代码"
   - "谁调用了这个函数？"
   - "显示依赖关系图"

4. **rg**（精确搜索）：
   - "找导入语句"
   - "显示所有 TODO"
   - "搜索特定模式"

---

### 成本分析

**5 天项目**：

| 工具 | 费用 | 备注 |
|------|------|-------|
| **claude-mem** | $4.68 | 312 条观察记录 × $0.15/100 |
| **Serena** | 免费 | 本地存储 |
| **grepai** | 免费 | 本地 Ollama |
| **合计** | **$4.68** | 5 天完整记忆的成本 |

**投资回报**：850 次减少的文件读取 × 约 500 token/次 = 42.5 万 token 节省 = $4.25（按 $0.01/1k token 计算）

**净成本**：5 天完整记忆的净成本仅 $0.43

---

## 相关资源

**内存栈工具**：
- [claude-mem GitHub](https://github.com/thedotmack/claude-mem)
- [Serena GitHub](https://github.com/oraios/serena)
- [grepai GitHub](https://github.com/yoanbernabeu/grepai)

**指南章节**：
- 第 8.2.3 节：Serena（符号记忆）
- 第 8.2.4 节：grepai（语义搜索）
- 第 8.2.5 节：claude-mem（自动会话记忆）
- 内存工具决策矩阵：guide/ultimate-guide.md:8630

---

**工作流创建时间**：2026-02-10
**指南版本**：3.24.0
**作者**：Florian BRUNIAUX + Claude（Anthropic）

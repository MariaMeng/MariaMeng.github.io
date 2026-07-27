---
layout: page
title: "如何高质量完成一个 Git Commit 交付"
permalink: /best-practice/git-commit-quality-guide/
---

# 如何高质量完成一个 Git Commit 交付

> **调研日期**：2026-07-27
> **核心来源**：《Software Engineering at Google》(O'Reilly, 2020) + Google Engineering Practices Documentation
> **定位**：从 Google 顶级工程实践中提炼的 Git Commit 完整交付标准

---

## 一、为什么 Commit Message 质量如此重要？

Google 在《Software Engineering at Google》中提出了一个核心论断：

> **"Software engineering is programming integrated over time."**（软件工程 = 编程 × 时间的积分）

这意味着代码不是写完就结束的——它需要被阅读、理解、维护、演化数年甚至数十年。而 **Commit Message 是代码的"元数据"**，它构成了版本控制历史中最重要的知识库之一。

Google Engineering Practices 文档明确指出：

> **"A CL description is a public record of what change is being made and why it was made."**

每一次 commit 都是一笔**历史债务或历史资产**——好的 commit message 是资产，差的是债务。

### Chesterton's Fence 原则

《SWE Book》Chapter 3 引用了 Chesterton 的名言：

> **"Don't ever take a fence down until you know the reason why it was put up."**
> （在你理解围栏为什么被建起来之前，永远不要拆掉它。）

如果 commit message 只写了 "Fix bug"，未来的开发者在看到这段代码时：不知道修复的是什么 bug，不知道 refactor 的动机，不敢修改也不敢删除。**当围栏没有标签时，没人敢碰它。**

---

## 二、Commit Message 写作标准

### 2.1 标准结构

```
<类型>(作用域): <祈使语气的简短描述>    ← 首行，50字符以内

<正文：解释 What + Why>                 ← 空行后，每行72字符以内

<脚注：Bug/Issue 链接、Breaking Changes> ← 空行后
```

### 2.2 首行（Subject Line）规则

| 规则 | 说明 | 来源 |
|------|------|------|
| **祈使语气** | "Add feature" 而非 "Added feature" | Google Eng Practices |
| **完整的句子** | 能独立成立，不看 body 也能理解 | Google Eng Practices |
| **50 字符以内** | 强迫精炼表达 | 业界共识 |
| **说"做了什么"** | Why 放在 body 中 | Google Eng Practices |

**验证公式**：一条好的首行应该能完成这个句子——
> "If applied, this commit will **\[你的首行\]**"

- ✅ If applied, this commit will **add batch expiration notification**
- ❌ If applied, this commit will ~~added batch expiration notification~~

### 2.3 Type 类型定义（Conventional Commits）

| Type | 含义 | SemVer |
|------|------|--------|
| `feat` | 新功能 | MINOR |
| `fix` | Bug 修复 | PATCH |
| `refactor` | 重构（非修复非新功能） | - |
| `perf` | 性能优化 | - |
| `docs` | 文档变更 | - |
| `test` | 测试相关 | - |
| `build` | 构建系统/外部依赖 | - |
| `ci` | CI 配置 | - |
| `style` | 格式调整（不影响逻辑） | - |
| `chore` | 其他杂项 | - |

### 2.4 正文（Body）规则

正文必须回答以下问题：

1. **Why** — 为什么需要这个变更？之前的问题是什么？
2. **What**（补充） — 具体做了什么（对首行的展开）
3. **Trade-offs** — 方案的不足和权衡
4. **Context** — Bug 编号、benchmark 结果、设计文档链接

> ⚠️ 外部链接可能失效——关键信息要直接写在 body 中，不能只贴链接。

---

## 三、Commit 粒度标准：Atomic Commit

### 3.1 什么是好的 Atomic Commit？

| 特征 | 说明 |
|------|------|
| **单一目的** | 做且只做一件事 |
| **自包含** | 提交后系统仍能正常构建和运行 |
| **包含测试** | 生产代码和对应测试在同一个 commit |
| **可独立 review** | Reviewer 不需要额外信息就能理解 |
| **可独立 rollback** | 回滚不会影响其他功能 |
| **合理大小** | ~100 行合理，~1000 行过大 |

Google 明确指出：

> **"When in doubt, write CLs that are smaller than you think you need to write. Reviewers rarely complain about getting CLs that are too small."**

### 3.2 为什么小 Commit 更好？（Google 的 8 大理由）

| # | 优势 | 说明 |
|---|------|------|
| 1 | **Review 更快** | 找 5 分钟审小 CL 比腾出 30 分钟审大 CL 容易 |
| 2 | **Review 更彻底** | 大变更让 reviewer 疲惫，重要问题容易被遗漏 |
| 3 | **引入 bug 概率更低** | 变更越少，越容易推理影响 |
| 4 | **被拒绝时浪费更少** | 方向错了，小 CL 的沉没成本低 |
| 5 | **更容易 merge** | 大 CL 开发时间长，冲突更多 |
| 6 | **设计更精良** | 小变更更容易打磨设计 |
| 7 | **不阻塞开发** | 发送自包含部分后可继续编码 |
| 8 | **更简单的 rollback** | 大 CL 涉及的文件更可能被后续修改 |

> ⭐ **Reviewer 有权仅因 CL 太大就直接拒绝 review。**

### 3.3 拆分策略（面向 Java + Spring Boot）

| 策略 | 示例 |
|------|------|
| **重构与功能分离** | CL1: 提取 OrderQueryService; CL2: 添加缓存逻辑 |
| **按层级拆分** | CL1: DTO/Mapper → CL2: Repository → CL3: Service → CL4: Controller |
| **按功能拆分** | CL1: 创建订单; CL2: 取消订单; CL3: 查询订单 |
| **配置与代码分离** | CL1: application.yml 配置; CL2: 使用配置的业务代码 |
| **Migration 独立** | Flyway migration 作为独立 commit |

### 3.4 判断标准

- 如果 commit message 需要用 **"and"** 连接两个动作 → 考虑拆分
- 如果需要超过 **2 句话**解释"做了什么" → 可能太大了
- 如果改了超过 **50 个文件** → 一定太大了

---

## 四、好的 vs 差的 Commit Message 对比

### 4.1 Bug 修复

❌ **差的**：
```
Fix bug
```

✅ **好的**：
```
fix(order): prevent NPE when user has no default address

OrderService.createOrder() threw NPE when User.getDefaultAddress()
returned null. This happens for newly registered users who haven't
set a default address yet.

Add null check and fall back to the first available address.
If no address exists, throw a descriptive BusinessException.

Fixes #12345
```

### 4.2 新功能

❌ **差的**：
```
Add new feature
```

✅ **好的**：
```
feat(coupon): add expiration reminder push notification

Implement scheduled task to send push notifications to users
whose coupons expire within 24 hours. Expected to improve
redemption rate by ~15% based on pilot A/B test.

- Add CouponExpirationReminderJob scheduled at 10:00 daily
- Query expiring coupons via CouponRepository
- Send notification via MsgCenterPushService

Ticket: SAILOR-5678
```

### 4.3 重构

❌ **差的**：
```
Refactor code
```

✅ **好的**：
```
refactor(order): extract price calculation to PriceCalculator

OrderService has grown to 800+ lines with mixed responsibilities.
Extract all price-related logic (base price, discount, tax,
shipping fee) into dedicated PriceCalculator class.

Step 1 of OrderService decomposition plan (design doc: xxx).
No behavior change -- all existing tests pass without modification.
```

### 4.4 配置变更

❌ **差的**：
```
Update config
```

✅ **好的**：
```
perf(db): increase Redis connection pool from 10 to 50

Production monitoring shows pool exhaustion during peak hours
(11:30-13:00), causing ~2% request failures.

Calculation: peak QPS (3000) / avg query time (15ms) * safety
factor (1.5) = ~45, rounded to 50.
Memory impact: ~40MB additional (acceptable for 8GB instances).
```

### 4.5 性能优化

❌ **差的**：
```
Improve performance
```

✅ **好的**：
```
perf(db): add composite index on (user_id, status) to orders table

Order list query taking 200-500ms due to full table scan on
12M rows. Adding composite index reduces p99 from 480ms to 35ms.

Index size: ~180MB. Tested with EXPLAIN ANALYZE on prod-like data.
```

### 4.6 安全修复

❌ **差的**：
```
Security fix
```

✅ **好的**：
```
fix(coupon): sanitize input in code validation to prevent SQL injection

CouponService.validateCode() concatenated user input into native SQL,
creating a Critical SQL injection vulnerability.

Replace with parameterized @Query. Add input validation:
coupon codes must match [A-Z0-9]{8,16}.

Security-Review: approved by @security-team
```

### 4.7 依赖升级

❌ **差的**：
```
Upgrade dependencies
```

✅ **好的**：
```
build: upgrade Spring Boot from 2.7.x to 3.1.x

Spring Boot 2.7 reaches EOL Nov 2025. Key changes:
- Migrate javax.* to jakarta.* namespace
- Update Hibernate config for Hibernate 6
- Fix deprecated API usages

BREAKING CHANGE: Minimum Java version now 17 (was 11).
```

### Google 列出的反面教材清单

| 差的描述 | 问题 |
|----------|------|
| "Fix bug" | 什么 bug？怎么修的？ |
| "Fix build." | 什么构建问题？根因？ |
| "Add patch." | 什么补丁？为什么？ |
| "Moving code from A to B." | 为什么？有什么好处？ |
| "Phase 1." | Phase 1 of what？ |
| "Add convenience functions." | 什么函数？为谁便利？ |
| "kill weird URLs." | 什么 URL？为什么 weird？ |

---

## 五、提交前自检清单（15 项）

基于 Google Reviewer 检查清单反推的 Author 自检清单：

| # | 检查项 |
|---|--------|
| 1 | ✅ 首行是祈使语气的完整句子，能独立理解 |
| 2 | ✅ 首行 50 字符以内 |
| 3 | ✅ Body 解释了 Why（不只是 What） |
| 4 | ✅ CL 只做一件事（single purpose） |
| 5 | ✅ CL 大小合理（~100 行，不超过 ~500 行） |
| 6 | ✅ 重构和功能变更没有混在一起 |
| 7 | ✅ 包含了相关的测试代码 |
| 8 | ✅ 所有测试通过 |
| 9 | ✅ 命名清晰准确 |
| 10 | ✅ 注释解释了"为什么"而非"是什么" |
| 11 | ✅ 没有过度设计（YAGNI） |
| 12 | ✅ 遵循团队 Style Guide |
| 13 | ✅ 更新了相关文档 |
| 14 | ✅ 提交前 re-read 了自己的 diff |
| 15 | ✅ Description 仍反映 CL 的最终状态 |

---

## 六、Commit Message 与 Code Review 的关系

### Reviewer 的检查优先级（Google 标准）

| 顺序 | 检查项 | 与 Commit Message 的关系 |
|------|--------|------------------------|
| 1 | Design | commit message 应说明设计决策 |
| 2 | Functionality | commit message 应描述预期行为 |
| 3 | Complexity | 复杂变更需要在 message 中解释原因 |
| 4 | Tests | - |
| 5 | Naming | - |
| 6 | Comments | 注释应解释 why，与 commit message 互补 |
| 7 | Style | - |
| 8 | Documentation | commit message 本身就是文档 |

### 好的 Commit Message = Reviewer 的路线图

Google 优化的是**团队整体产出速度**，而非个人编码速度：

> **"At Google, we optimize for the speed at which a team of developers can produce a product together."**

好的 commit message 能：
- 让 reviewer 快速理解变更背景和动机
- 减少 review 轮次（round-trips）
- 缩短从提交到合入的时间

### 处理 Review 评论的正确姿势

当 reviewer 说看不懂代码时——**改代码，而不是写解释**：

> **"Your first response should be to clarify the code itself."**

优先级：
1. 让代码本身更清晰（最优先）
2. 添加代码注释
3. 在 review 工具中解释（最后手段）

---

## 七、工具链推荐

```
commitizen（写）→ commitlint + husky（验）→ release-please（发）
```

| 阶段 | 工具 | 作用 |
|------|------|------|
| **编写** | commitizen | 交互式引导，降低心智负担 |
| **校验** | commitlint + husky | Git hook 自动拦截不合规 commit |
| **发布** | release-please (Google) | 自动 CHANGELOG + 版本号 |

快速安装：
```bash
# 安装
npm install --save-dev @commitlint/{cli,config-conventional} husky commitizen

# 配置 commitlint
echo "module.exports = {extends: ['@commitlint/config-conventional']};" > commitlint.config.js

# 设置 husky hook
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

---

## 八、行动清单（10 条）

| # | 行动 | 优先级 | 来源 |
|---|------|--------|------|
| 1 | **首行使用祈使语气，50 字符以内** | 🔴 立即 | Google: CL Descriptions |
| 2 | **Body 必须回答 Why** | 🔴 立即 | Google: CL Descriptions + SWE Book Ch3 |
| 3 | **每个 Commit 只做一件事** | 🔴 立即 | Google: Small CLs + SWE Book Ch16 |
| 4 | **采用 Conventional Commits 类型前缀** | 🔴 立即 | Conventional Commits 1.0.0 |
| 5 | **控制大小：~100 行目标，~500 行警戒** | 🟡 逐步 | Google: Small CLs |
| 6 | **重构和功能变更永远分开提交** | 🟡 逐步 | Google: Small CLs |
| 7 | **测试代码与生产代码同一个 Commit** | 🟡 逐步 | Google: Small CLs |
| 8 | **提交前 re-read diff + description** | 🟡 每次 | Google: CL Descriptions |
| 9 | **拒绝 "Fix bug" / "WIP" 类 message** | 🟢 规范 | Google: Bad CL Descriptions |
| 10 | **安装 commitlint + husky 工具链** | 🟢 一次 | 工具保障 |

---

## 参考资料

| 来源 | URL |
|------|-----|
| Software Engineering at Google (在线) | [abseil.io/resources/swe-book](https://abseil.io/resources/swe-book) |
| Google CL Description 指南 | [CL Descriptions](https://google.github.io/eng-practices/review/developer/cl-descriptions.html) |
| Google Small CLs 指南 | [Small CLs](https://google.github.io/eng-practices/review/developer/small-cls.html) |
| Code Review Standard | [Standard](https://google.github.io/eng-practices/review/reviewer/standard.html) |
| What to Look For | [Looking For](https://google.github.io/eng-practices/review/reviewer/looking-for.html) |
| Handling Comments | [Handling Comments](https://google.github.io/eng-practices/review/developer/handling-comments.html) |
| Review Speed | [Speed](https://google.github.io/eng-practices/review/reviewer/speed.html) |
| Conventional Commits 1.0.0 | [conventionalcommits.org](https://www.conventionalcommits.org/en/v1.0.0/) |
| Angular Commit Guidelines | [Angular Contributing](https://github.com/angular/angular/blob/main/contributing-docs/commit-message-guidelines.md) |
| Chris Beams - Git Commit | [cbea.ms/git-commit](https://cbea.ms/git-commit/) |

---

> **一句话总结**：代码展示了 How，Commit Message 解释了 What 和 Why。一个优秀的工程师不仅写出好的代码，还要写出好的 Commit Message——因为这两者共同构成了代码库的长期资产。

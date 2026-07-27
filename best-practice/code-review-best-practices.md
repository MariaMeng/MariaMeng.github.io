---
layout: page
title: "如何做好高质量的 Code Review —— Reviewer 视角"
permalink: /best-practice/code-review-best-practices/
---

# 如何做好高质量的 Code Review —— Reviewer 视角深度调研报告

> 📅 调研日期：2026-07-27
>
> 📚 核心来源：Google Engineering Practices Documentation、《Software Engineering at Google》Chapter 9、SmartBear Best Practices、Stack Overflow Blog
>
> 🎯 目标读者：Java 后端工程师（Reviewer 角色）

---

## 目录

- [Part 1: Google 的 Code Review 哲学](#part-1-google-的-code-review-哲学)
- [Part 2: Reviewer 应该看什么？（What to Look For）](#part-2-reviewer-应该看什么what-to-look-for)
- [Part 3: 如何高效 Navigate 一个 CL？](#part-3-如何高效-navigate-一个-cl)
- [Part 4: Code Review 的速度与节奏](#part-4-code-review-的速度与节奏)
- [Part 5: 如何写好 Review Comments](#part-5-如何写好-review-comments)
- [Part 6: 处理分歧与 Pushback](#part-6-处理分歧与-pushback)
- [Part 7: 常见反模式（Anti-patterns）](#part-7-常见反模式anti-patterns)
- [Part 8: 面向 Java + Spring Boot 的 CR 实战要点](#part-8-面向-java--spring-boot-的-cr-实战要点)
- [Part 9: 行动清单](#part-9-行动清单)
- [参考资料](#参考资料)

---

## Part 1: Google 的 Code Review 哲学

### 1.1 Code Review 的核心目的

Code Review 的目的远不止"找 bug"。根据 Google 的工程实践文档和《Software Engineering at Google》Chapter 9，Code Review 服务于以下多重目标：

| 目的 | 说明 |
|------|------|
| **代码库健康度（Code Health）** | 确保代码库的整体质量随时间持续改善，而非逐渐退化 |
| **知识共享（Knowledge Sharing）** | 让团队成员了解彼此的代码，避免知识孤岛 |
| **一致性（Consistency）** | 确保代码风格、设计模式在整个代码库中保持一致 |
| **教育与指导（Mentoring）** | 帮助开发者学习新的语言特性、框架用法、设计原则 |
| **守门人（Gatekeeping）** | 确保变更符合团队的设计方向和技术决策 |
| **文档化决策** | Review 中的讨论记录了设计决策的上下文和理由 |

> **核心洞见**：Google 将 Code Review 定位为**代码库健康度的守护机制**，而非单纯的缺陷检测工具。

来源：[Google Engineering Practices - The Standard of Code Review](https://google.github.io/eng-practices/review/reviewer/standard.html)

### 1.2 The Standard of Code Review —— Google 的审批标准

Google 对 Code Review 的核心审批标准可以用一句话概括：

> **"In general, reviewers should favor approving a CL once it is in a state where it definitely improves the overall code health of the system being worked on, even if the CL isn't perfect."**
>
> — Google Engineering Practices

这句话包含几个关键信息：

1. **没有"完美"的代码** —— 只有"更好"的代码（*There is no such thing as "perfect" code — there is only better code.*）
2. **持续改进优于追求完美** —— Reviewer 应该追求的是 continuous improvement，而非 perfection
3. **不应因为代码不完美就延迟合入** —— 一个整体上提升了系统可维护性、可读性、可理解性的 CL，不应该因为不"完美"而被拖延数天
4. **但底线不可放弃** —— 绝不允许合入明确降低代码库整体健康度的 CL（除非是 [Emergency](https://google.github.io/eng-practices/review/emergencies.html)）

**什么时候该 Approve？**

| ✅ 应该 Approve | ❌ 不应该 Approve |
|-----------------|-------------------|
| CL 整体上改善了代码健康度 | CL 引入了不必要的复杂性 |
| 虽有小瑕疵但不影响代码质量 | CL 明确降低了代码健康度 |
| 留下 "Nit:" 前缀的建议后 approve | CL 方向与团队技术决策冲突 |
| Author 展示了多种方案都同样合理 | CL 缺少必要的测试 |

### 1.3 Google CR 的类型分类

根据《Software Engineering at Google》Chapter 9，Google 的 Code Review 可以分为以下几类：

| 类型 | 描述 | Review 重点 |
|------|------|-------------|
| **Greenfield Review** | 全新功能/模块的代码 | 重点关注设计、架构是否合理；是否过度工程化 |
| **Bug Fix Review** | 修复缺陷 | 关注修复是否正确、是否引入回归；测试是否覆盖该 bug |
| **Refactoring Review** | 重构代码 | 确认行为不变、测试充分、重构方向合理 |
| **Small Change / Cleanup** | 小规模清理和优化 | 快速审查，确认改进方向正确 |
| **Behavioral Changes** | 改变已有行为的变更 | 重点关注向后兼容性、影响范围 |

来源：《Software Engineering at Google》Chapter 9 — Code Review（O'Reilly, 2020）

### 1.4 Google CR 文化中的核心原则

根据 Google 的 Reviewer Guide 中明确阐述的 **Principles**：

> 1. **Technical facts and data overrule opinions and personal preferences.**（技术事实和数据优先于个人意见和偏好）
> 2. **On matters of style, the style guide is the absolute authority.**（在风格问题上，Style Guide 是绝对权威）
> 3. **Aspects of software design are almost never a pure style issue or just a personal preference.**（软件设计几乎从来不是纯粹的风格问题）
> 4. **If no other rule applies, then the reviewer may ask the author to be consistent with what is in the current codebase.**（如果没有其他规则适用，保持与现有代码一致）

来源：[Google Engineering Practices - Standard](https://google.github.io/eng-practices/review/reviewer/standard.html)

《Software Engineering at Google》进一步阐述了 Google CR 文化的基石：

- **强制性**：Google 几乎所有代码变更都必须经过 Code Review，这不是可选的。
- **工具支持**：Google 使用 Critique 工具进行 Code Review，与内部代码搜索、构建系统深度集成。
- **Readability 制度**：Google 有专门的 "Readability" 认证，通过认证的工程师可以 approve 特定语言的代码。
- **LGTM 文化**：Reviewer 通过 "LGTM"（Looks Good to Me）来批准变更。

---

## Part 2: Reviewer 应该看什么？（What to Look For）

### 2.1 完整检查清单

Google 在 [What to Look For in a Code Review](https://google.github.io/eng-practices/review/reviewer/looking-for.html) 中列出了 Reviewer 应该关注的所有维度。以下是每个维度的详细检查要点：

---

#### ① Design（设计） —— 最重要的维度

> **"The most important thing to cover in a review is the overall design of the CL."**

**检查要点：**

- CL 中各个代码片段的交互是否合理？
- 这个变更是否应该属于当前代码库，还是应该放在一个独立的库中？
- 它与系统的其他部分是否良好集成？
- 现在是添加这个功能的合适时机吗？

**判断标准：**

| ✅ 好的设计 | ❌ 差的设计 |
|-------------|-------------|
| 职责清晰，单一职责原则 | 一个类/方法做了太多事情 |
| 抽象层次合理 | 过度抽象或抽象不足 |
| 解决当前已知的问题 | 过度工程化，解决"可能"需要的未来问题 |
| 接口清晰稳定 | 接口暴露了实现细节 |

> **关于过度工程化（Over-engineering）的特别警告：**
>
> "Reviewers should be especially vigilant about over-engineering. Encourage developers to solve the problem they know needs to be solved now, not the problem that the developer speculates might need to be solved in the future."
>
> — [Google: What to Look For](https://google.github.io/eng-practices/review/reviewer/looking-for.html)

---

#### ② Functionality（功能正确性）

**检查要点：**

- CL 是否实现了开发者的意图？
- 开发者的意图对用户来说是否正确？（"用户"包括终端用户和未来的开发者）
- 边界情况（edge cases）是否处理？
- 是否存在并发问题（concurrency problems）？
- 是否有通过阅读代码就能发现的 bug？

**特别注意的场景：**

- **用户可见的变更（UI 变更）**：纯读代码很难判断效果，可以要求 Author 做 demo
- **并行编程**：死锁（deadlock）和竞态条件（race condition）仅靠运行代码很难发现，需要仔细思考

---

#### ③ Complexity（复杂度）

**检查要点（在每个层次上检查）：**

- **行级别**：单行代码是否太复杂？
- **函数级别**：函数是否太长/太复杂？
- **类级别**：类的职责是否太多？

**判断标准：**

> "Too complex" usually means **"can't be understood quickly by code readers"**. It can also mean **"developers are likely to introduce bugs when they try to call or modify this code."**

**必须 Block 的情况：**
- 代码对读者来说难以理解
- 添加了当前不需要的通用化设计（YAGNI 原则）

---

#### ④ Tests（测试）

**检查要点：**

- 是否有对应的单元测试、集成测试或端到端测试？
- 测试是否正确、合理、有用？
- 当代码被破坏时，测试是否真的会失败？
- 是否会产生误报（false positives）？
- 每个测试是否做了简单且有用的断言？
- 测试方法之间是否有适当的分离？
- 测试代码的复杂度是否合理？

> **"Tests do not test themselves, and we rarely write tests for our tests — a human must ensure that tests are valid."**
>
> — [Google: What to Look For](https://google.github.io/eng-practices/review/reviewer/looking-for.html)

**关键提醒**：测试也是需要维护的代码。不要因为"只是测试"就接受不必要的复杂度。

---

#### ⑤ Naming（命名）

**检查要点：**

- 名称是否足够长以完整表达其含义？
- 名称是否过长以至于难以阅读？
- 命名是否准确反映了变量/函数/类的用途？

---

#### ⑥ Comments（代码注释）

**检查要点：**

- 注释是否用清晰的语言编写？
- 注释是否真正必要？
- 注释是否在解释 **why**（为什么这样做），而非 **what**（代码在做什么）？
- 是否有可以删除的过时 TODO？
- 是否有之前的注释建议不应该做当前这个修改？

> **核心原则**：如果代码不够清晰，首先应该简化代码，而不是添加注释来解释复杂的代码。

**好的注释 vs 差的注释：**

| ✅ 好的注释 | ❌ 差的注释 |
|-------------|-------------|
| `// 使用LRU缓存因为查询结果有时间局部性且DB查询成本高` | `// 获取用户信息` (代码已经自解释了) |
| `// 这里必须先释放锁再发送通知，否则会导致死锁(详见ISSUE-1234)` | `// 设置 i = 0` |
| `// 正则说明: 匹配 IPv6 地址的简写形式` | `// 遍历列表` |

---

#### ⑦ Style（代码风格）

**检查要点：**

- CL 是否遵循了团队的 Style Guide？
- 个人风格偏好 vs 规范要求 —— 如果 Style Guide 未覆盖，用 "Nit:" 前缀标注

> **重要规则**：不要仅因为个人风格偏好就 block 一个 CL。

**特别注意**：不应在同一个 CL 中混合大规模风格改动和功能改动。如果需要重新格式化整个文件，应该作为独立的 CL 提交。

---

#### ⑧ Consistency（一致性）

- Style Guide 是绝对权威
- Style Guide 未覆盖的情况：保持与周围代码一致
- 鼓励 Author 提交 TODO 来清理已有的不一致代码

---

#### ⑨ Documentation（文档）

**检查要点：**

- CL 是否影响了构建、测试、交互或发布方式？
- 关联的文档（README、API 文档等）是否已更新？
- 是否需要删除已废弃代码的文档？

---

#### ⑩ Every Line（逐行审查）

> **"Look at every line of code that you have been assigned to review."**

**可以快速扫过的：**
- 数据文件、生成的代码、大型数据结构

**必须仔细阅读的：**
- 人类编写的类、函数、代码块

**一个重要推论：**

> "If you can't understand the code, it's very likely that other developers won't either. So you're also helping future developers understand this code, when you ask the developer to clarify it."

如果你读不懂代码，不是你的问题 —— 是代码需要改进的信号。

---

#### ⑪ Context（上下文）

- 不要只看变更的几行代码，要看整个文件的上下文
- 例如：添加的 4 行代码可能使一个方法膨胀到 50 行，需要拆分
- 从系统整体角度思考：这个 CL 是在改善还是在恶化系统？

---

#### ⑫ Good Things（发现亮点要表扬）

> **"If you see something nice in the CL, tell the developer, especially when they addressed one of your comments in a great way."**

来源：[Google: What to Look For](https://google.github.io/eng-practices/review/reviewer/looking-for.html)

### 2.2 什么必须 Block？什么可以放过？

| 🚫 必须 Block（Required） | ✅ 可以放过（Nit/Optional） |
|----------------------------|---------------------------|
| 明确的 bug 或逻辑错误 | 更好的变量命名建议 |
| 缺少必要的测试 | import 排序 |
| 设计层面的严重问题 | 微小的代码风格偏好 |
| 安全漏洞（SQL 注入、XSS 等） | 更优雅的写法（现有写法也正确） |
| 引入了不可接受的复杂度 | 文档中的小瑕疵 |
| 违反 Style Guide 的强制要求 | 额外的优化（非性能关键路径） |
| 并发安全问题 | 附近已有代码的改进建议 |
| 降低代码库整体健康度 | 纯教育性质的建议 |

---

## Part 3: 如何高效 Navigate 一个 CL？

来源：[Google: Navigating a CL in Review](https://google.github.io/eng-practices/review/reviewer/navigate.html)

### 3.1 三步法

Google 推荐的 CL 阅读方法分为三步：

#### Step 1: 俯瞰全局（Take a Broad View）

**做什么：**
1. 阅读 CL Description —— 这个变更要做什么？为什么？
2. 判断这个变更是否应该存在 —— 方向是否正确？

**如果方向不对，立即反馈。** 但要注意措辞：

> **❌ 差的反馈：**
> "这个 CL 不应该做。"
>
> **✅ 好的反馈：**
> "Looks like you put some good work into this, thanks! However, we're actually going in the direction of removing the FooWidget system that you're modifying here, and so we don't want to make any new modifications to it right now. How about instead you refactor our new BarWidget class?"

来源：[Google: Navigating a CL](https://google.github.io/eng-practices/review/reviewer/navigate.html)

#### Step 2: 审查核心部分（Examine the Main Parts）

**做什么：**
1. 找到逻辑变更最多的文件 —— 这通常是 CL 的核心
2. 先看核心部分，为理解其他部分提供上下文
3. **如果发现重大设计问题，立即反馈**，不要等看完所有代码

**为什么要立即反馈设计问题？**
- Author 可能已经基于这个 CL 开始了新的工作
- 重大设计改动比小改动需要更长时间

#### Step 3: 按合理顺序审查剩余部分（Look Through the Rest）

**做什么：**
1. 确认没有遗漏任何文件
2. 可以按代码审查工具的默认顺序
3. 有时候先读测试代码会更有帮助 —— 可以更快理解变更的意图

### 3.2 大 CL 的处理策略

当收到一个很大的 CL 时：

1. **首选方案**：要求 Author 拆分成多个小 CL
2. **无法拆分时**：至少给出整体设计层面的评论，让 Author 可以开始改进
3. **核心原则**：Reviewer 的目标之一是始终 unblock Author，让他们能够快速采取下一步行动

> **"One of your goals as a reviewer should be to always unblock the developer or enable them to take some sort of further action quickly, without sacrificing code health to do so."**
>
> — [Google: Speed of Code Reviews](https://google.github.io/eng-practices/review/reviewer/speed.html)

### 3.3 快速抓住核心变更的技巧

| 技巧 | 说明 |
|------|------|
| **先读 CL Description** | 好的 description 会告诉你变更的 what 和 why |
| **看文件变更量排序** | 变更行数最多的文件通常是核心 |
| **先读测试** | 测试代码往往最直接地展示了预期行为 |
| **关注新增的类/接口** | 新增的抽象通常是设计的关键 |
| **看 import 变化** | 新依赖暗示了设计方向 |

---

## Part 4: Code Review 的速度与节奏

来源：[Google: Speed of Code Reviews](https://google.github.io/eng-practices/review/reviewer/speed.html)

### 4.1 Google 对 Review 速度的要求

> **"One business day is the maximum time it should take to respond to a code review request (i.e., first thing the next morning)."**

注意：这说的是**响应时间（response time）**，不是整个 review 完成的时间。

- **如果没在做集中任务**：收到 review 请求后应尽快处理
- **如果在做集中任务**：等到一个断点再处理（完成当前编码任务、午餐后、会议结束后）
- **跨时区场景**：在对方下班前完成 review；如果来不及，至少在对方第二天上班前完成

### 4.2 为什么 Review 速度很重要？

Google 的文档列出了 review 太慢导致的三个后果：

| 后果 | 说明 |
|------|------|
| **团队整体速度下降** | 新功能和 bug 修复被延迟，虽然 reviewer 个人在做其他事，但团队整体被阻塞 |
| **开发者开始抵触 CR 流程** | 如果 reviewer 每次都隔几天才回复且提出大量修改意见，开发者会抱怨 reviewer "太严格"。**但如果 reviewer 回复很快，即使要求同样严格，抱怨也会消失** |
| **代码健康度受损** | Review 太慢会让开发者产生压力，倾向于提交质量不够好的 CL |

> **核心洞见**：大多数对 Code Review 流程的抱怨，实际上可以通过加快流程来解决。
>
> *"Most complaints about the code review process are actually resolved by making the process faster."*

### 4.3 速度 vs 质量的平衡

| 场景 | 建议 |
|------|------|
| 正在集中编码 | 不要中断自己，等到一个自然断点 |
| 太忙无法完整 review | 发一条消息告知何时可以 review，或建议其他 reviewer |
| 能快速完成 | 立即 review |
| CL 太大没时间完整看 | 至少给出整体设计评论 |

### 4.4 LGTM With Comments

为了加速 review 流程，Google 允许在以下情况下**带评论 Approve**：

1. Reviewer 确信 Author 会正确处理所有剩余评论
2. 剩余评论不需要 Author 处理
3. 建议很小（如排序 import、修复附近的 typo、使用建议的修复等）

> **跨时区场景**：LGTM With Comments 特别有用。否则 Author 可能仅仅为了一个 "LGTM, Approval" 就要等整整一天。

### 4.5 中断与批处理的策略

| 策略 | 说明 |
|------|------|
| **自然断点法** | 完成当前任务 → Review → 下一个任务 |
| **固定时间段** | 每天设置 2-3 个固定的 review 时间段（如早上、午餐后、下午） |
| **批处理** | 积累到一个断点时集中处理多个 review 请求 |
| **优先级排序** | 紧急/小 CL 先处理，大 CL 安排到有整块时间时处理 |

### 4.6 紧急情况（Emergencies）

来源：[Google: Emergencies](https://google.github.io/eng-practices/review/emergencies.html)

**什么算紧急情况：**
- 阻止重大发布的回滚的小修改
- 修复线上严重影响用户的 bug
- 紧迫的法律问题
- 关闭重大安全漏洞

**什么不算紧急情况：**
- 想本周而不是下周上线
- 开发者已经做了很久很想合入
- Reviewer 在不同时区
- 周五下班前想把 CL 合入
- 经理说今天必须完成（soft deadline）

> 紧急 CL 合入后，应该回过头来对这些 CL 做更彻底的 review。

---

## Part 5: 如何写好 Review Comments

来源：[Google: How to Write Code Review Comments](https://google.github.io/eng-practices/review/reviewer/comments.html)

### 5.1 四项基本原则

Google 总结的 Review Comment 写作原则：

1. **Be kind（友善）**
2. **Explain your reasoning（解释你的理由）**
3. **Balance giving explicit directions with just pointing out problems（平衡给出方向和指出问题）**
4. **Encourage developers to simplify code or add code comments（鼓励简化代码或添加注释）**

### 5.2 Courtesy（礼貌）原则

**核心规则：评论代码，不评论人。**

> **❌ 差的评论（评论人）：**
> "Why did you use threads here when there's obviously no benefit to be gained from concurrency?"
>
> **✅ 好的评论（评论代码）：**
> "The concurrency model here is adding complexity to the system without any actual performance benefit that I can see. Because there's no performance benefit, it's best for this code to be single-threaded instead of using multiple threads."

来源：[Google: Comments - Courtesy](https://google.github.io/eng-practices/review/reviewer/comments.html)

### 5.3 Explain Why（解释为什么）

不要只说"这不对"或"改成这样"，要解释**为什么**：

| ❌ 差的评论 | ✅ 好的评论 |
|-------------|-------------|
| "这个方法太长了" | "这个方法有 80 行，包含了数据校验、业务逻辑和持久化三个关注点。拆分成三个方法可以提高可读性和可测试性" |
| "不要用 synchronized" | "这里使用 synchronized 会锁住整个对象，在高并发场景下会成为瓶颈。考虑使用 ConcurrentHashMap 或 ReentrantReadWriteLock 来细化锁粒度" |
| "加个测试" | "这个边界条件（amount = 0）没有被测试覆盖，而且从 calculateDiscount 的逻辑看，amount = 0 时可能会触发除零异常" |

### 5.4 Nit vs Required 的区分

Google 推荐使用标签来明确评论的严重程度：

| 标签 | 含义 | 是否必须修改 |
|------|------|-------------|
| **（无标签）** | 必须修改 | ✅ 是 |
| **Nit:** | 小问题，技术上应该改，但影响不大 | ❌ 可选 |
| **Optional:** / **Consider:** | 建议，不是必须 | ❌ 可选 |
| **FYI:** | 信息性，不需要在这个 CL 中处理 | ❌ 不需要 |

**示例：**
- `Nit: 这里的变量名 d 不太清晰，建议改为 duration`
- `Optional: 这个逻辑可以用 Stream API 简化，但当前写法也没问题`
- `FYI: Java 17 引入了 sealed classes，未来重构时可以考虑用在这个继承体系上`

### 5.5 好的评论 vs 差的评论（更多示例）

**场景 1：发现潜在的 NPE**

| ❌ 差的评论 | ✅ 好的评论 |
|-------------|-------------|
| "这里会 NPE" | "当 user.getAddress() 返回 null 时，第 42 行的 .getCity() 调用会抛出 NPE。建议加 null check 或使用 Optional：`Optional.ofNullable(user.getAddress()).map(Address::getCity).orElse(DEFAULT_CITY)`" |

**场景 2：代码风格**

| ❌ 差的评论 | ✅ 好的评论 |
|-------------|-------------|
| "不要这样写" | "Nit: 根据我们的 style guide，常量应该用 UPPER_SNAKE_CASE。这里 `maxRetryCount` 建议改为 `MAX_RETRY_COUNT`" |

**场景 3：设计问题**

| ❌ 差的评论 | ✅ 好的评论 |
|-------------|-------------|
| "这个设计不好" | "这个类同时负责了数据校验、业务计算和数据库操作，违反了单一职责原则。建议拆分为 Validator、Calculator 和 Repository 三个类，这样每个类都可以独立测试，未来修改也更容易定位" |

**场景 4：表扬好的实践**

| ❌ 差的评论 | ✅ 好的评论 |
|-------------|-------------|
| （什么都不说） | "👍 这里使用 Builder 模式构造复杂对象的做法很好，比之前的 telescoping constructor 可读性好多了" |

### 5.6 Accepting Explanations

当你让开发者解释一段代码时，这通常应该导致他们**重写代码使其更清晰**，而非仅仅在 review 工具中留下解释。

> **核心观点**：只写在 Code Review 工具中的解释对未来的代码阅读者没有帮助。如果代码需要解释，应该改进代码本身或在代码中添加注释。

来源：[Google: Comments - Accepting Explanations](https://google.github.io/eng-practices/review/reviewer/comments.html)

---

## Part 6: 处理分歧与 Pushback

来源：[Google: Handling Pushback in Code Reviews](https://google.github.io/eng-practices/review/reviewer/pushback.html)

### 6.1 当 Author 不同意你的意见

**首先，认真考虑对方是否正确：**

> "When a developer disagrees with your suggestion, first take a moment to consider if they are correct. Often, they are closer to the code than you are, and so they might really have a better insight about certain aspects of it."

**决策流程：**

```
Author 不同意我的建议
      │
      ▼
他们的论点是否有道理？── 是 ──→ 承认对方正确，放弃建议
      │
      否
      │
      ▼
进一步解释你的理由
（展示你理解了对方的观点 + 提供额外信息）
      │
      ▼
是否关系到代码健康度？── 是 ──→ 坚持你的建议
      │
      否
      │
      ▼
    可以让步
```

### 6.2 什么时候应该坚持？什么时候应该让步？

| 应该坚持 | 可以让步 |
|----------|----------|
| 改善代码健康度的建议 | 纯风格偏好（Style Guide 未覆盖的） |
| 安全性/正确性问题 | 两种方案都同样合理时 |
| 违反设计原则 | 改进虽好但投入产出比低 |
| 引入技术债 | Author 用数据/工程原理证明了其方案等效 |

### 6.3 "我后面会清理" 的陷阱

> **"Experience shows that as more time passes after a developer writes the original CL, the less likely this clean up is to happen. In fact, usually unless the developer does the clean up immediately after the present CL, it never happens."**

**应对策略：**

- 对于 CL 自身引入的新复杂性 → **坚持在合入前修复**
- 对于 CL 暴露出的周围代码问题 → 接受，但要求 Author：
  1. 创建一个 bug/issue 来跟踪清理工作
  2. 将 issue 分配给自己
  3. 在代码中添加 TODO 注释并引用 issue 编号

### 6.4 升级决策（Escalation）的时机和方式

来源：[Google: Standard - Resolving Conflicts](https://google.github.io/eng-practices/review/reviewer/standard.html)

**升级路径：**

1. **第一步**：Reviewer 和 Author 基于 Code Review 指南尝试达成共识
2. **第二步**：面对面讨论或视频会议（讨论结果必须记录在 CL 评论中）
3. **第三步**：扩大到团队讨论
4. **第四步**：请 Tech Lead 介入
5. **第五步**：请代码的 Maintainer 做决定
6. **第六步**：请 Engineering Manager 协助

> **关键原则：不要让 CL 因为 Author 和 Reviewer 无法达成一致而长期搁置。**

### 6.5 如何在 CR 中保持团队和谐

| 做法 | 说明 |
|------|------|
| **始终保持礼貌** | 评论代码，不评论人 |
| **承认好的实践** | 看到好的做法要表扬 |
| **解释理由** | 不要只说"不行"，解释为什么 |
| **倾听对方** | 即使不同意，也展示你理解了对方的观点 |
| **区分严重程度** | 用 Nit/Optional/FYI 标签避免所有评论都看起来像"必须改" |
| **面对面沟通** | 文字容易产生误解，复杂分歧用面谈解决 |

根据《Software Engineering at Google》Chapter 9 的观点，Google 的 CR 文化强调：Code Review 是一种**对等协作**（peer review），而非**上下级审查**。Reviewer 和 Author 是共同为代码质量负责的伙伴关系。

---

## Part 7: 常见反模式（Anti-patterns）

### 7.1 Reviewer 常犯的错误

综合 Google Engineering Practices、SmartBear Code Review Best Practices（https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/）和 Stack Overflow Blog（https://stackoverflow.blog/2019/09/30/how-to-make-good-code-reviews-better/）的内容，以下是 Reviewer 最常犯的错误：

#### 反模式 1: 吹毛求疵（Nitpicking to Death）

**表现：**
- 对每一行代码都提出修改意见
- 将个人风格偏好当作必须修改的问题
- 要求代码达到"完美"才能合入

**后果：**
- 开发者沮丧，抵触 CR 流程
- Review 周期大幅延长
- 开发者害怕提交 CL

**解决方法：**

> Google 的建议："Reviewers should not require the author to polish every tiny piece of a CL before granting approval. Rather, the reviewer should balance out the need to make forward progress compared to the importance of the changes they are suggesting."

- 使用 "Nit:" 前缀标注非必须修改
- 记住核心标准：CL 是否整体上改善了代码健康度？

#### 反模式 2: 橡皮图章（Rubber Stamping）

**表现：**
- 几秒钟就 LGTM
- 只看 CL Description 不看代码
- "我信任这个人，不用看了"

**后果：**
- bug 和设计问题流入代码库
- 代码库健康度逐渐下降
- CR 流程失去意义

**解决方法：**
- 逐行审查每一行被分配给你 review 的代码
- 即使是高级工程师的代码也需要认真 review

> SmartBear 的研究表明：每次 review 的代码量不应超过 200-400 行，超过这个量后发现缺陷的能力急剧下降。

来源：SmartBear — *Best Practices for Peer Code Review*（https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/）

#### 反模式 3: 风格偏好 vs 真正的问题

**表现：**
- 花大量时间讨论大括号位置、空格数量
- 忽视设计和功能问题，纠结于格式

**解决方法：**
- 风格问题交给自动化工具（linter、formatter）
- Review 时聚焦于设计、逻辑、测试
- 记住 Google 的原则：*Style Guide 是风格问题的绝对权威，Style Guide 未覆盖的是个人偏好*

#### 反模式 4: 阻塞式 Review（Review 太慢）

**表现：**
- 收到 review 请求后数天不响应
- 每一轮都隔很久才回复
- 积压大量 review 请求

**后果：**
- 团队速度下降
- 开发者对 CR 流程产生怨言
- 代码质量反而下降（因为压力导致降低标准）

**解决方法：**
- 一个工作日内必须响应
- 如果太忙，至少发消息告知预计时间或建议其他 reviewer

#### 反模式 5: 只关注负面，从不表扬

**表现：**
- Review 只指出问题，从不认可好的做法
- 让 Author 觉得 CR 只是批评大会

**解决方法：**

> "Code reviews often just focus on mistakes, but they should offer encouragement and appreciation for good practices, as well. It's sometimes even more valuable, in terms of mentoring, to tell a developer what they did right than to tell them what they did wrong."
>
> — [Google: What to Look For - Good Things](https://google.github.io/eng-practices/review/reviewer/looking-for.html)

#### 反模式 6: 重写偏好（The "I Would Have Done It Differently" Trap）

**表现：**
- Author 的实现方式虽然可行，但 Reviewer 更喜欢另一种方式
- 要求 Author 按自己的方式重写

**解决方法：**
- 问自己：两种方式是否都同样合理？
- 如果是，接受 Author 的方式（Google 原则："If the author can demonstrate that several approaches are equally valid, then the reviewer should accept the preference of the author."）

#### 反模式 7: 一次性倾泻大量评论

**表现：**
- 看完整个 CL 后一次性提出 50+ 条评论
- 不区分优先级，Author 不知道从何改起

**解决方法：**
- 发现重大设计问题时立即反馈，不要等看完所有代码
- 使用严重程度标签区分优先级
- 如果设计层面有问题，其他细节可以暂时不提

---

## Part 8: 面向 Java + Spring Boot 的 CR 实战要点

### 8.1 Java 后端代码 CR 的特定关注点

#### 线程安全

| 检查项 | 具体内容 |
|--------|----------|
| **共享可变状态** | 类的成员变量是否在多线程环境下被安全访问？是否需要 `volatile`、`synchronized` 或并发容器？ |
| **竞态条件** | check-then-act 模式是否存在竞态？例如 `if (map.containsKey(key)) { map.get(key).doSomething(); }` |
| **SimpleDateFormat** | 是否作为共享变量使用？（非线程安全，应使用 DateTimeFormatter 或 ThreadLocal） |
| **HashMap** | 多线程环境下是否使用了 HashMap 而非 ConcurrentHashMap？ |
| **线程池配置** | 线程池参数是否合理？是否有无界队列导致 OOM 的风险？ |

**示例：**

```java
// ❌ 非线程安全 —— 必须 Block
private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

// ✅ 线程安全
private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
```

#### NPE（NullPointerException）防护

| 检查项 | 具体内容 |
|--------|----------|
| **返回值可能为 null** | 外部方法调用、数据库查询、Map.get() 等返回值是否做了 null check？ |
| **链式调用** | `user.getAddress().getCity().toLowerCase()` 这种链式调用是否有 NPE 风险？ |
| **集合操作** | 对集合操作前是否检查了 null 和 empty？ |
| **Optional 使用** | 是否合理使用 Optional？是否滥用 `Optional.get()` 而不检查 `isPresent()`？ |
| **@Nullable/@NonNull** | 关键接口的参数和返回值是否标注了可空性？ |

#### 资源泄漏

| 检查项 | 具体内容 |
|--------|----------|
| **IO 流** | InputStream/OutputStream/Connection 是否在 finally 或 try-with-resources 中关闭？ |
| **数据库连接** | 连接/Statement/ResultSet 是否正确释放？ |
| **线程** | 是否有线程被创建但从未关闭？ExecutorService 是否在不需要时 shutdown？ |
| **锁** | ReentrantLock 是否在 finally 中 unlock？ |

```java
// ❌ 资源泄漏 —— 必须 Block
InputStream is = new FileInputStream(file);
String content = IOUtils.toString(is);
// is 没有关闭

// ✅ 正确做法
try (InputStream is = new FileInputStream(file)) {
    String content = IOUtils.toString(is);
}
```

#### SQL 注入与安全

| 检查项 | 具体内容 |
|--------|----------|
| **SQL 拼接** | 是否有字符串拼接 SQL？必须使用参数化查询（PreparedStatement / MyBatis #{} ） |
| **MyBatis ${}** | `${}` 是直接拼接，有 SQL 注入风险；应使用 `#{}` |
| **敏感数据** | 密码、token 等是否明文存储？日志中是否打印了敏感信息？ |
| **权限校验** | 接口是否做了权限校验？是否存在越权访问（水平/垂直越权）？ |

```xml
<!-- ❌ SQL 注入风险 —— 必须 Block -->
<select id="findUser">
    SELECT * FROM user WHERE name = '${name}'
</select>

<!-- ✅ 安全的参数化查询 -->
<select id="findUser">
    SELECT * FROM user WHERE name = #{name}
</select>
```

#### 异常处理

| 检查项 | 具体内容 |
|--------|----------|
| **空 catch** | 是否有 `catch (Exception e) {}` 吞掉异常？ |
| **过于宽泛的 catch** | 是否 catch 了 Exception 而非具体异常？ |
| **异常信息** | 异常日志是否包含了足够的上下文信息？ |
| **checked vs unchecked** | 异常类型选择是否合理？ |

### 8.2 Spring Boot 特有的 Review 要点

#### Bean 生命周期

| 检查项 | 具体内容 |
|--------|----------|
| **单例 Bean 中的可变状态** | Spring Bean 默认是 singleton，在 Bean 中存储请求级别的可变状态会导致线程安全问题 |
| **循环依赖** | 是否存在 Bean 之间的循环依赖？Spring Boot 2.6+ 默认禁止循环依赖 |
| **Bean 的初始化顺序** | `@PostConstruct` 中访问的依赖是否已经初始化？ |
| **@Scope** | 是否需要 prototype scope 而误用了 singleton？ |

```java
// ❌ 单例 Bean 中的线程不安全状态 —— 必须 Block
@Service
public class OrderService {
    private Order currentOrder; // 多线程共享！
    
    public void process(Order order) {
        this.currentOrder = order; // 竞态条件
        // ...
    }
}
```

#### 事务管理

| 检查项 | 具体内容 |
|--------|----------|
| **@Transactional 失效** | 自调用（同一个类中的方法调用）不会触发 AOP 代理，事务不生效 |
| **事务传播级别** | propagation 设置是否合理？嵌套事务的行为是否符合预期？ |
| **长事务** | 事务中是否包含了 RPC 调用、文件操作等耗时操作？ |
| **异常回滚** | `@Transactional` 默认只对 RuntimeException 回滚，checked exception 不回滚 |
| **只读事务** | 查询操作是否标记了 `readOnly = true`？ |

```java
// ❌ @Transactional 自调用失效 —— 必须指出
@Service
public class UserService {
    public void createUser(User user) {
        saveUser(user); // 自调用，事务不生效！
    }
    
    @Transactional
    public void saveUser(User user) {
        userMapper.insert(user);
    }
}

// ❌ 长事务 —— 需要关注
@Transactional
public void processOrder(Order order) {
    orderMapper.insert(order);
    // 事务中调用外部 RPC —— 可能导致长事务和连接池耗尽
    inventoryService.deductStock(order.getItems()); 
    paymentService.charge(order);
}
```

#### 配置管理

| 检查项 | 具体内容 |
|--------|----------|
| **硬编码配置** | URL、端口、超时时间等是否硬编码？应放到配置文件中 |
| **@Value 默认值** | `@Value("${xxx}")` 是否设置了合理的默认值？缺少配置时是否会导致启动失败？ |
| **配置分环境** | 测试/预发/生产的配置是否隔离？ |
| **敏感配置** | 密码、密钥等是否使用了加密存储（如配置中心的加密功能）？ |

#### 其他 Spring Boot 关注点

| 检查项 | 具体内容 |
|--------|----------|
| **API 设计** | RESTful 接口设计是否合理？HTTP 方法使用是否正确？ |
| **参数校验** | 是否使用了 `@Valid` / `@Validated` 进行参数校验？ |
| **统一异常处理** | 是否通过 `@ControllerAdvice` 统一处理异常？ |
| **日志规范** | 日志级别是否合理？是否使用了参数化日志（`log.info("user: {}", userId)` 而非字符串拼接）？ |
| **接口幂等性** | 写接口是否保证了幂等性？ |

### 8.3 后端场景下的 CR 重点

基于后端微服务架构的常见关注点：

| 维度 | 关注点 |
|------|--------|
| **性能** | 是否有 N+1 查询？是否需要分页？大数据量查询是否有超时风险？ |
| **缓存** | 缓存使用是否合理？是否有缓存穿透/击穿/雪崩的风险？缓存一致性如何保证？ |
| **分布式问题** | 分布式锁使用是否正确？分布式事务策略是否合理？ |
| **监控告警** | 关键路径是否添加了监控打点？异常是否有告警？ |
| **降级容错** | 外部依赖是否有降级方案？超时设置是否合理？ |
| **数据一致性** | 多个数据源更新是否有一致性保证？消息消费是否幂等？ |
| **向后兼容** | 接口变更是否向后兼容？数据库 DDL 变更是否需要分步发布？ |
| **灰度发布** | 是否需要灰度策略？开关/Feature Flag 是否设置？ |

---

## Part 9: 行动清单

以下是 12 条可直接落地的实践建议，每条附上来源：

### ✅ 1. 记住核心标准：CL 是否改善了代码健康度？

不追求完美，追求持续改进。如果 CL 整体上改善了代码健康度，即使不完美也应该 approve。

> 来源：[Google - The Standard of Code Review](https://google.github.io/eng-practices/review/reviewer/standard.html)

### ✅ 2. 使用三步法阅读 CL

**Step 1**: 读 Description，判断方向 → **Step 2**: 看核心文件，确认设计 → **Step 3**: 按序审查剩余部分。发现重大设计问题时立即反馈。

> 来源：[Google - Navigating a CL in Review](https://google.github.io/eng-practices/review/reviewer/navigate.html)

### ✅ 3. 一个工作日内响应 Review 请求

如果在集中编码，等到一个自然断点再处理。如果太忙，至少发消息告知时间或推荐其他 reviewer。

> 来源：[Google - Speed of Code Reviews](https://google.github.io/eng-practices/review/reviewer/speed.html)

### ✅ 4. 评论代码，不评论人

用客观的语言描述代码的问题，解释原因和建议方案，不要用质问的语气或暗示开发者能力不足。

> 来源：[Google - How to Write Code Review Comments](https://google.github.io/eng-practices/review/reviewer/comments.html)

### ✅ 5. 使用严重程度标签

区分必须修改、建议修改和信息性评论。使用 `Nit:`、`Optional:`、`FYI:` 等前缀，避免所有评论都看起来像"必须改"。

> 来源：[Google - Comments: Label comment severity](https://google.github.io/eng-practices/review/reviewer/comments.html)

### ✅ 6. 解释 Why，不只说 What

每条修改建议都应该说明**为什么**需要改，而不仅仅是**改成什么**。帮助 Author 理解背后的原则，而非机械地执行修改。

> 来源：[Google - Comments: Explain Why](https://google.github.io/eng-practices/review/reviewer/comments.html)

### ✅ 7. 看到好的实践要表扬

不要只关注问题。当 Author 写了好的代码、做了好的设计、或者很好地回应了你的评论时，给予正面反馈。

> 来源：[Google - What to Look For: Good Things](https://google.github.io/eng-practices/review/reviewer/looking-for.html)

### ✅ 8. 在 Java CR 中建立安全检查清单

每次 review Java 代码时，系统性地检查：线程安全、NPE 防护、资源泄漏、SQL 注入、异常处理、事务管理。将这些检查项内化为习惯。

> 来源：综合 Google 实践 + Java/Spring Boot 最佳实践

### ✅ 9. 不接受"后面再清理"的承诺

如果 CL 引入了新的复杂性，坚持在合入前修复。对于暴露出的已有问题，要求创建跟踪 issue 并添加 TODO。

> 来源：[Google - Handling Pushback: Cleaning It Up Later](https://google.github.io/eng-practices/review/reviewer/pushback.html)

### ✅ 10. 将风格检查交给自动化工具

使用 Checkstyle、SpotBugs、SonarQube 等工具自动检测风格和常见缺陷，Reviewer 聚焦于工具无法检测的设计、逻辑、业务理解。

> 来源：SmartBear — *Best Practices for Peer Code Review*（https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/）

### ✅ 11. 控制单次 Review 的代码量

单次 review 的代码量建议控制在 200-400 行以内。对于大 CL，要求 Author 拆分。如果无法拆分，分多个 session 完成 review。

> 来源：SmartBear 研究数据 + [Google - Speed: Large CLs](https://google.github.io/eng-practices/review/reviewer/speed.html)

### ✅ 12. 善用 LGTM With Comments 加速流程

对于非关键的小建议，可以 approve 的同时留下评论。特别是在跨时区协作时，避免 Author 仅为了一个 LGTM 等一整天。

> 来源：[Google - Speed: LGTM With Comments](https://google.github.io/eng-practices/review/reviewer/speed.html)

---

## 参考资料

| # | 来源 | URL |
|---|------|-----|
| 1 | Google Engineering Practices - Reviewer Guide 首页 | https://google.github.io/eng-practices/review/reviewer/ |
| 2 | Google - The Standard of Code Review | https://google.github.io/eng-practices/review/reviewer/standard.html |
| 3 | Google - What to Look For in a Code Review | https://google.github.io/eng-practices/review/reviewer/looking-for.html |
| 4 | Google - Navigating a CL in Review | https://google.github.io/eng-practices/review/reviewer/navigate.html |
| 5 | Google - Speed of Code Reviews | https://google.github.io/eng-practices/review/reviewer/speed.html |
| 6 | Google - How to Write Code Review Comments | https://google.github.io/eng-practices/review/reviewer/comments.html |
| 7 | Google - Handling Pushback in Code Reviews | https://google.github.io/eng-practices/review/reviewer/pushback.html |
| 8 | Google - Emergencies | https://google.github.io/eng-practices/review/emergencies.html |
| 9 | Google Engineering Practices 总览 | https://google.github.io/eng-practices/ |
| 10 | 《Software Engineering at Google》Chapter 9: Code Review (O'Reilly, 2020) | https://abseil.io/resources/swe-book/html/ch09.html |
| 11 | SmartBear - Best Practices for Peer Code Review | https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/ |
| 12 | Stack Overflow Blog - How to Make Good Code Reviews Better | https://stackoverflow.blog/2019/09/30/how-to-make-good-code-reviews-better/ |

---

> 📝 **调研说明**：本报告的核心内容主要基于 Google Engineering Practices Documentation（已成功全量抓取 8 个页面的完整内容）。《Software Engineering at Google》Chapter 9、SmartBear 和 Stack Overflow Blog 的内容基于这些广泛引用的公开资料和业界共识进行了补充。Java/Spring Boot 部分基于后端工程实践经验编写。

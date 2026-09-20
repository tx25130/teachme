# Harness 模板与使用指南

Harness 是可执行的验证脚本（或固定题库），用于锁定核心业务逻辑。**只要 Harness 通过，就代表核心契约未被破坏。**

## 何时使用 Harness？

当一块逻辑满足以下**全部**条件时：
1. 已验收通过，确定短期内不变。
2. 属于核心业务边界（你不希望 AI 在后续迭代中“偷偷改掉”的部分）。

## 目录结构建议

./harness/
├── code/ # 代码型验证（单元测试）
│ └── test_core.py (或 .js)
│ └── test_core.py（或.js）
└── concept/ # 概念型验证（固定 Flashcard 题库）
└── core_principles.md


---

## 类型一：代码型 Harness（编程/配置类）

**适用场景**：锁定函数输入输出、算法边界、配置规则。
**写法**：使用通用的测试框架（`pytest`、`Jest` 等），AI 可以自动生成。

### 示例（Python + Pytest）

```python
# harness/code/test_core_logic.py
import pytest
from src.pet import Pet  # 假设这是核心模块

class TestPetEatLogic:
    # 锁定 ADR-002 中的进食逻辑
    def test_auto_eat_below_threshold(self):
        pet = Pet(satiety=25)
        pet.check_and_eat()
        assert pet.is_eating is True, "饱腹度低于30必须触发进食"

    def test_stop_eat_at_threshold(self):
        pet = Pet(satiety=60)
        pet.check_and_eat()
        assert pet.is_eating is False, "饱腹度达到60必须停止进食"

### 示例（JavaScript / Jest）

// harness/code/core.spec.js
const { Pet } = require('../../src/pet');

describe('Pet Eating Harness (Locked via ADR-002)', () => {
  it('should eat when satiety < 30', () => {
    const pet = new Pet(25);
    pet.checkAndEat();
    expect(pet.isEating).toBe(true);
  });

  it('should stop when satiety >= 60', () => {
    const pet = new Pet(60);
    pet.checkAndEat();
    expect(pet.isEating).toBe(false);
  });
});

### 执行规则

- AI 在修改涉及该逻辑的代码后，必须运行此 Harness。
- 若失败，AI 必须暂停，并提示：“Harness 校验失败，当前修改违反了 ADR-XXX 的历史决策，请确认是否要废弃旧决策。”


## 类型二：概念型 Harness（知识/非编程类）
适用场景：锁定核心定义、不可变的历史事实、公式边界。
写法：一份固定的 Q&A 清单，AI 在生成新内容时不得违背。

### 示例（核心原则核验卡）

- harness/concept/core_principles.md

- 核验清单（Harness Check）

> 当 AI 在后续会话中解释这些概念时，其核心含义**不得**偏离以下标准答案。若偏离，视同 Harness 失败。

**Q1: 什么是“合意难度”？**
- **锁定答案**：指在学习中故意引入的困难（如检索练习、间隔效应），这些困难短期内降低流畅度，但能显著增强长期记忆（存储强度）。
- **禁止的表述**：单纯的难题、随便增加作业量。

**Q2: 本工作区中“Spec”和“Harness”的关系是什么？**
- **锁定答案**：Spec 是自然语言的因果契约（给 AI 理解背景），Harness 是可执行的强制验证（给机器判定通过/失败）。Spec 覆盖所有决策，Harness 仅锁定核心不可变逻辑。
- **禁止的表述**：Harness 是 Spec 的详细写法、两者可以互相替代。

### 使用流程
当 AI 需要撰写涉及这些概念的新 Lesson 时，必须优先查阅此清单。如果写出的定义与清单冲突，用户可直接判定该 Lesson 不合格，要求重写。

### 反模式（禁止）
- ❌ 不要把正在频繁变动的 UI 样式写入 Harness（会导致维护成本剧增）。
- ❌ 不要把 Harness 当作普通单元测试覆盖所有代码（只锁核心边界，不锁边缘细节）。
- ❌ 不要手动编写大量 Harness —— 在验收通过后，让 AI 自动生成 Harness 脚本，人工只负责确认锁定的范围是否正确。
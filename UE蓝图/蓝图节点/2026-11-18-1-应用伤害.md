---
date: 2026-11-18
type: 蓝图节点
tags: [UE, 蓝图]
---

# 应用伤害（Apply Damage）

把一次伤害数值交给目标 Actor，真正扣不扣血由目标自己的事件决定。

## 干什么用的

`Apply Damage` 是 `UGameplayStatics::ApplyDamage` 的蓝图节点。它只负责**传递**伤害和伤害来源，不做扣血、不做死亡判断——目标蓝图里绑了 `Event AnyDamage` 才会响应。

所以它更像"敲门"，不是"动手"。

## 输入 / 输出

| 引脚 | 类型 | 说明 |
|---|---|---|
| （执行输入） | Exec | 触发一次伤害 |
| Damaged Actor | Actor | 被打的目标 |
| Base Damage | Float | 伤害数值，传正数 |
| Event Instigator | Controller | 发起者（一般是玩家控制器），用于击杀归属、仇恨 |
| Damage Causer | Actor | 实际造成伤害的 Actor，比如那颗子弹、那个陷阱 |
| Damage Type Class | Class（DamageType 子类） | 伤害类型，留空走默认类型 |
| Return Value | Float | 目标 `TakeDamage` 返回的值，一般就是传进去的数值 |

## 典型场景

- 子弹命中判定 → `Apply Damage` → 目标 `AnyDamage` 里扣血、播受击动画、飘伤害数字
- 毒圈、火焰地板这类周期性伤害：用 Timer 每 0.5 秒对范围内 Actor 各 Apply 一次
- 击杀统计：目标收到伤害时把 `Event Instigator` 存进自己的 `LastInstigator`，死亡时再拿它结算

## 坑

- **目标没响应的最常见原因**：Actor 上 `Can be Damaged` 被关掉了，或者根本没写 `Event AnyDamage`。这时候节点执行成功、什么都不发生、也不报错，很容易以为伤害算错了。
- **只在服务器调用**。客户端调一次不会让血量同步下去，表现就是"打了没反应"。伤害判定老老实实放服务器。
- `Damage Type Class` 留空时会用默认类型。做元素抗性、护甲减伤时，每一发都得带上对应类型，不然所有伤害都长一个样。
- `Event Instigator` 传空，目标死亡时找不到凶手，击杀数、掉落归属、仇恨全会丢。
- 别传负数想做治疗。要治疗就单独写函数或走 `Apply Healing` 那条路，负数伤害在减伤计算里容易翻车。

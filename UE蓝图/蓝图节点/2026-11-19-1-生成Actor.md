---
date: 2026-11-19
type: 蓝图节点
tags: [UE, 蓝图, 生成]
---

# 生成 Actor（Spawn Actor from Class）

在运行时按指定的类实例化一个 Actor，放回一个引用给你用。

## 干什么用的

把"类"变成场景里真实存在的对象。子弹、掉落物、敌人、特效触发器基本都是它生出来的。
和「在编辑器里摆一个」不同，这个是运行时的动态生成。

## 输入 / 输出

| 引脚 | 类型 | 说明 |
|---|---|---|
| 执行 | Exec | 进 / 出 |
| Class | Class（Actor 子类） | 要生成什么，默认是 Actor 基类，必须换成具体蓝图类 |
| Spawn Transform | Transform | 生成的位置、旋转、缩放 |
| Collision Handling Override | 枚举 | 出生点被占了怎么办（见下） |
| Owner | Actor | 谁"拥有"它，用于网络归属和生命周期管理 |
| Instigator | Pawn | 谁"导致"它出现，做伤害归因时用 |
| Return Value | Actor 引用 | 生成出来的那个对象，失败时是空 |

Collision Handling Override 的几个值，按严格程度排：

- `Always Spawn, Ignore Collisions` —— 不管撞不撞都生成，且忽略碰撞
- `Try To Adjust Location, But Always Spawn` —— 先试着挪开，挪不开也硬生成
- `Try To Adjust Location, Don't Spawn If Still Colliding` —— 挪不开就不生成
- `Do Not Spawn` —— 只要撞了就放弃

## 典型场景

- 开火：从枪口 Socket 的 Transform 生成一发子弹，Instigator 设成开火的 Pawn
- 拾取物：怪物死亡时在尸体位置生成掉落
- 波次刷怪：循环里按刷新点数组批量生成

## 坑

- **返回值可能是空**。用了 `Don't Spawn` 这类策略而出生点被占，节点不生成东西，
  后面直接拿返回值调函数就是空引用。生成完先判 Is Valid 再往下走。
- **想"先生成再设参数"不能用它**。这个节点在生成的那一刻就跑了 BeginPlay，
  你还没来得及改血量/队伍它就已经初始化完了。要先改属性的场景用
  `Spawn Actor Deferred`，设完参数再调 `Finish Spawning Actor`。
- **别在 Tick 里生成**。每帧 Spawn 一个 Actor 的分配和注册开销很实在，几百个就掉帧。
  该用对象池就用对象池，或者改成事件驱动。
- **Class 引脚是硬引用**。在蓝图里直接填一个类，会把这个类连同它的资源全部打进包，
  哪怕你只是偶尔生成一次。种类多的话考虑软引用加载后再生成。
- **缩放传递**。Spawn Transform 的 Scale 会作用到 Actor 上，但很多 Actor 的碰撞体
  在构造时就定了大小，缩放后碰撞和表现可能对不上，优先在类里做尺寸而不是靠缩放。

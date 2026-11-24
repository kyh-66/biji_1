---
date: 2026-11-24
type: 蓝图节点
tags: [UE, 蓝图, 碰撞]
---

# 组件开始重叠（On Component Begin Overlap）

挂在某个组件的碰撞上：这个组件和别的物体**开始相交**的那一刻，触发一次。

## 干什么用的

检测「进去了」这件事。触发器、拾取物、伤害区域、门禁、检查点，全靠它。

它来自 **Collision 分区的组件绑定事件**——在组件详情面板的碰撞设置下方，点 `+` 选 `On Component Begin Overlap`，
蓝图里就会出现一个同名红色事件节点。**不是**靠拖一个「节点」出来的，新手最容易在这里找不到。

## 输入 / 输出

事件节点本身没有输入引脚（由引擎在碰撞发生时调用），输出如下：

| 引脚 | 类型 | 说明 |
|---|---|---|
| Overlapped Component | Primitive Component | 发生重叠的组件（也就是挂事件的那个自身组件） |
| Other Actor | Actor | 撞进来的那个 Actor |
| Other Comp | Primitive Component | 对方具体的组件（胶囊体？网格体？） |
| Other Body Index | Integer | 对方是多体（比如骨骼网格的物理体）时，第几个体，一般忽略 |
| bFrom Sweep | Boolean | 这次重叠是否由移动时的 Sweep 造成 |
| Sweep Result | Hit Result | Sweep 的命中信息，仅 bFromSweep 为真时有意义 |

配套的还有 `On Component End Overlap`（离开）和 `On Component Hit`（撞击，只有模拟物理的物体才会触发）。

## 典型场景

- **拾取物**：Begin Overlap → Cast 到玩家 → 加血/加弹药 → `Destroy Actor`
- **伤害区域**：Begin Overlap 开始扣血，End Overlap 停止扣血（两个事件成对用）
- **检查点 / 触发器**：玩家经过就 `Set Checkpoint`，记得加个 Bool 防重复触发
- **门禁**：只有特定 Tag 的 Actor 进来才开门

## 坑

1. **事件不触发，九成是碰撞设置问题**，不是蓝图写错：
   - 组件的 **Collision Presets** 要选能产生重叠的（如 `OverlapAllDynamic`），
     `NoCollision` 和 `BlockAll` 都不产生 Overlap 事件。
   - 两个物体都要 **Generate Overlap Events 打勾**，少一个都不行。
   - 查询和物理要都设为 Overlap，一方 Block 一方 Overlap 时行为不一致。
2. **只有响应通道开了才算**。`OverlapAllDynamic` 忽略了 `WorldStatic` 之外的很多通道，
   自定义通道要去项目设置里给两边都配好。
3. **每帧都在触发**的错觉：Begin 只在「进入」那一瞬触发一次，如果你看到刷屏，
   通常是把逻辑写进了 `Tick` 或者 End/Begin 在边界反复抖——检查物体是否卡在边界上。
4. **用 Other Actor 前一定要判空并 Cast 失败即返回**，因为世界里的子弹、掉落物、特效碰撞盒都会撞进来。
5. **别在 Begin Overlap 里 Spawn 带碰撞的东西**，新生成的物体可能立刻再触发一次，形成连锁。
   要么延迟一帧 `Set Timer by Event`，要么生成时把碰撞先关掉。
6. 需要「持续重叠」时不要在本事件里做每帧逻辑，用 `Is Overlapping Actor` 在 Tick 里判，或者用 Begin/End 两个事件维护一个状态。

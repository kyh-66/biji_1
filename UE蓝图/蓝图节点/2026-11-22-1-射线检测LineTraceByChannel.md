---
date: 2026-11-22
type: 蓝图节点
tags: [UE, 蓝图, 碰撞, 射线]
---

# 射线检测（Line Trace By Channel）

从起点到终点拉一条看不见的线，返回碰到的第一个挡住这条线的东西。

## 干什么用的

问你一句「这两点之间的直线上有没有东西？是什么？」。它不生成任何物体，只是查询碰撞世界，
所以每帧调用也不会留下痕迹——但每帧调用很贵，这点后面讲。

## 输入 / 输出

| 引脚 | 类型 | 说明 |
|---|---|---|
| Start | Vector | 线的起点，一般填摄像机位置或枪口位置的世界坐标 |
| End | Vector | 线的终点。通常写成「起点 + 朝向 × 长度」，长度就是射程 |
| Trace Channel | TraceTypeQuery | 走哪条碰撞通道（Visibility / Camera / 项目自定义的）。**只检测该通道响应为 Block 的对象** |
| Trace Complex | Boolean | 是否用精确三角形碰撞。默认关；打开才能拿到骨骼网格上准确的点 |
| Actors to Ignore | Actor 数组 | 忽略名单。施法者、自己的武器都塞这里 |
| Draw Debug Type | Enum | None / For Duration / Persistent，调试画线用 |
| Ignore Self | Boolean | 是否忽略调用者自己。默认勾着 |
| Trace Color / Trace Hit Color | Linear Color | 调试线的颜色（没命中 / 命中） |
| Draw Time | Float | 调试线停留秒数 |
| **Out Hit** | Hit Result | 命中信息结构体，没命中时里面的值不可靠 |
| **Return Value** | Boolean | 有没有挡住。**先看这个，再看 Out Hit** |

Hit Result 里常用字段：

- `bBlockingHit` 是否挡住
- `bStartPenetrating` 起手就在物体内部（比如贴着墙出生），这时 `Location` 不能当命中点用
- `Location` / `Impact Point` / `Impact Normal` 命中点与法线
- `Distance` 起点到命中点的距离
- `Hit Actor` / `Hit Component` 命中的是谁、哪个组件
- `Bone Name` 命中的骨骼名（打骨骼网格时才有意义）

## 典型场景

- **射击**：从摄像机中心往前打，命中点做弹孔，命中 Actor 上找可接收伤害的组件
- **交互**：玩家视线往前 200 单位，打中可交互物就显示「按 F」提示
- **地面检测**：脚底往下一小段距离打，判断站的地方是什么材质（草地 / 金属 / 水）
- **摆放建筑**：从鼠标世界坐标往下打，拿到地形高度再放 Ghost 模型
- **AI 视线**：AI 眼睛到玩家头颅打一条，中间被墙挡住就看不见

## 坑

- **打不到人，先怀疑通道。** 目标物在自己那条通道上要是 Ignore，线直接穿过去，看起来像没检测。
- **Ignore Self 不够用。** 你想忽略的是「自己 + 队友 + 自己的武器」，只勾 Ignore Self 挡不住别人的碰撞体，得用 Actors to Ignore 明确列出来。
- **BeginPlay 里打空。** 关卡切换、碰撞体还没注册的瞬间，射线常常什么都打不到。差一帧或者用 Timer 延迟 0.1 秒再看。
- **Trace Complex 别乱开。** 打开后走三角形级碰撞，长距离高频检测会明显掉帧，而且对网格资产有额外要求。
- **单线只给最近的一个。** 要一次拿到穿透路径上的所有对象，用 Multi Line Trace By Channel，它返回的是 Hit Result 数组，记得自己按 Distance 排序。
- **Out Hit 在没命中时是脏数据。** 一定先判断 Return Value，否则会拿一堆零当成世界原点用。
- **调试线记得关。** Draw Debug Type 留成 Persistent 又不关的线，会在正式包里一直挂着刷屏（编辑器里尤其难排查是谁画的）。

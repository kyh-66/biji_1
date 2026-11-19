---
date: 2026-11-18
type: 蓝图节点
tags: [UE, 蓝图, UI, 坐标变换]
---

# 世界坐标转屏幕坐标（Project World Location to Screen）

把三维世界里的一个点，换算成玩家屏幕上的二维像素坐标。

## 干什么用的

它做两件事：把世界位置乘上摄像机的 View/Projection 矩阵，再映射到玩家视口的像素空间。
返回值是个 `Vector2D`，单位是**视口像素**，原点在左上角。

它和「Get Screen Position」之类的纯数学节点不一样的地方在于：它需要**一个 PlayerController**，
因为屏幕尺寸、分屏视口、宽高比都存在 PlayerController 身上。

## 输入 / 输出

| 引脚 | 类型 | 说明 |
|---|---|---|
| Player Controller | Player Controller 对象引用 | 用当前本地玩家；分屏时不同视口要传各自那个 |
| World Location | Vector | 要投影的世界坐标（一般是组件世界位置） |
| Player Viewport Relative | Boolean（勾选项） | 分屏时相对自己那块视口算；全屏一般不用勾 |
| Screen Position | Vector2D | 输出，视口像素坐标（左上原点） |
| Return Value | Boolean | **点是否落在屏幕前方且在视锥内**，false 时上面的坐标不可信 |

## 典型场景

- **头顶血条 / 名牌**：每帧取角色骨骼的头顶位置投影，再把 UMG 控件挪过去
- **任务指引箭头**：投影目标点，判断在屏幕内就贴边显示，在屏幕外就用它的方向做罗盘箭头
- **3D 选中框**：做即时战略的框选，鼠标框和单位投影包围盒求交
- **伤害数字飘字**：把命中点投影成 UMG 位置，再用动画往上飞

## 坑

1. **Return Value 是 false 的时候别用输出坐标。** 点在摄像机背后时，投影结果会翻到屏幕另一侧，
   看起来像"东西穿屏了"。必须先用 bool 分流；背后的话改成"贴边 + 方向"的处理。
2. **UMG 的坐标不是屏幕像素。** UMG 有自己的 DPI 缩放和画布尺寸，直接把 Screen Position 当
   Canvas 位置喂进去，非 100% DPI 或非设计分辨率下会偏得离谱。正确做法是走 WidgetLayoutLibrary
   的 Screen to Widget 系列做换算（Local / Absolute 看你挂在哪层），而不是硬乘一个 magic number。
3. **分辨率 / 窗口尺寸变化会立刻失效。** 任何缓存的投影结果都要在 Viewport Resized 之后重算，
   否则拖动窗口后 UI 全歪。
4. **PlayerController 可能为空。** 在专用服务器、或者 GameInstance 里拿早期引用时它是 null，
   节点会直接返回 false；用之前判一下有效。
5. **分屏必须勾 Player Viewport Relative。** 不勾的话两个玩家共用主视口的屏幕坐标，UI 会重叠到另一块屏。
6. **别每帧每个单位都投影。** 一百个单位每帧调一次还行，但再加上 UMG 的 SetPosition 就顶不住了。
   常用做法是只处理视野内的、做距离剔除，或者干脆用 Niagara 的屏幕空间粒子代替 UMG。

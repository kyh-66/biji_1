Linear Interpolate，线性插值


它的作用是在两个数值、向量或颜色之间，根据一个比例系数平滑地计算出过渡值。

### 数学原理与公式

Lerp 的底层逻辑就是一维线性方程：

$$\text{Result} = A + (B - A) \times \text{Alpha}$$

- **$\text{Alpha} = 0.0$**：输出完全等于 **$A$**。
    
- **$\text{Alpha} = 1.0$**：输出完全等于 **$B$**。
    
- **$\text{Alpha} = 0.5$**：输出正好处于 **$A$ 和 $B$ 的正中中点**。
    
- **$\text{Alpha} > 1.0$ 或 $< 0.0$**：若未开启 Clamp，会向外延展（外插外推）。


根据输入引脚数据类型的不同，UE 提供了对应维度的 Lerp 节点：

|**节点类型**|**输入 A & B**|**Alpha 类型**|**典型应用场景**|
|---|---|---|---|
|**Lerp (Float)**|单个浮点数|Float|血条过渡、摄像机 FOV 缩放、速度过渡|
|**Lerp (Vector)**|FVector (X, Y, Z)|Float|角色/物体在两点之间的平移、位移轨迹计算|
|**Lerp (Rotator)**|FRotator (R, P, Y)|Float / Shortest Path|视角旋转、朝向对齐（注意：更推荐四元数插值）|
|**Lerp (Linear Color)**|FLinearColor (R, G, B, A)|Float|材质变色、角色受击变红闪烁、昼夜天空盒颜色渐变|
|**Lerp (Vector4)**|FVector4|Float|复杂着色器数据通道混合|



#### 配合 Timeline（匀速/曲线过渡）

这是最安全、最精准的实现方式。Timeline 提供一个从 `0.0` 到 `1.0` 随时间变化的 Alpha 曲线，直接连入 Lerp 的 Alpha 引脚。

- **典型应用**：开门动画、UI 进度条动画、电梯升降。
    
- **特点**：总耗时固定，过渡曲线可以通过 Timeline 的曲线编辑器自由控制（如缓入缓出 Ease In/Out）。
    

#### 2. 在 Event Tick 中逐帧插值（阻尼平滑）

将当前值连给 $A$，目标值连给 $B$，Alpha 给一个与 `Delta Seconds` 相关的较小浮点数。

- **公式范式**：`Current = Lerp(Current, Target, Alpha)`
    
- **注意陷阱**：直接写固定 Alpha 会导致帧率高时变快、帧率低时变慢（帧率不独立）。在 UE 中，此类逐帧逼近逻辑**更推荐直接使用 `FInterp To` / `VInterp To` / `RInterp To` 节点**，它们内部自带 `DeltaTime` 与 `InterpSpeed` 修正。
    

#### 3. 在材质编辑器（Material Editor）中混合贴图

在材质中，Lerp 快捷键为 **`L + 鼠标左键`**。

- **典型应用**：地表材质混合（泥土与草地）、破损贴图叠加、根据遮罩贴图（Mask）控制粗糙度或发光区域。
    
- **Alpha 驱动源**：通常由黑白蒙版贴图（R 通道）、Fresnel 菲涅尔节点、或是 Vertex Color（顶点色）驱动。
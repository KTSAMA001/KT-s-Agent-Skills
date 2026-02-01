# Unity 物理系统知识

<!-- 
修改记录：
- [2026-02-01] 创建文件，记录 3D Collider 类型性能消耗对比知识。
-->

本文档记录 Unity 物理系统的核心概念与原理，偏"知识 + 原理"。

---

## 3D Collider 类型性能消耗对比

**分类**：Unity > 物理 > Collider  
**关键词**：#Collider #Physics #性能 #BoxCollider #SphereCollider #CapsuleCollider #MeshCollider  
**来源**：Unity 2022.3 官方文档 - Collider types and performance  
**来源日期**：2026-01-31  
**收录日期**：2026-02-01  
**可信度**：⭐⭐⭐⭐⭐ (官方文档)

### 性能排序（从低到高）

| 碰撞体类型 | 性能消耗 | 说明 |
|-----------|---------|------|
| **SphereCollider** | 🟢 最低 | 最简单高效，适用于圆形物体和通用交互 |
| **CapsuleCollider** | 🟢 较低 | 比 Sphere 稍复杂，但仍高效。适合角色、柱状物体 |
| **BoxCollider** | 🟡 中等偏低 | 高效灵活，适合方形/块状物体。比 Sphere/Capsule 略耗资源 |
| **Convex MeshCollider** | 🟠 较高 | 比 Primitive 碰撞体耗资源多。需凸面网格，可附加到非 Kinematic Rigidbody |
| **Non-Convex MeshCollider** | 🔴 最高 | 最耗资源。仅用于静态、不移动且需精确碰撞面的几何体 |

### 核心要点

1. **Primitive Colliders (Sphere/Capsule/Box)** 是最高效的类型
   - 由简单几何形状定义，多边形数量极少
   - 物理引擎使用数学公式直接计算，而非三角形遍历

2. **Compound Collider（复合碰撞体）**
   - 由多个 Primitive Collider 组合而成
   - 比单个 MeshCollider 更高效
   - 适合复杂动态物体（如车辆、机器人）

3. **MeshCollider 特性**
   - 需要 **mesh cooking** 预处理，将几何体转换为优化的物理格式
   - 运行时 cooking 会造成 CPU 峰值
   - Non-Convex MeshCollider **不能**附加到非 Kinematic Rigidbody
   - 建议启用 **Prebake Collision Meshes**（Player Settings）避免运行时 cooking

### 使用场景建议

| 场景 | 推荐碰撞体 |
|------|-----------|
| 子弹、球体、简单触发器 | SphereCollider |
| 角色控制器、人形 hitbox | CapsuleCollider |
| 箱子、墙壁、平台 | BoxCollider |
| 复杂动态物体（车辆） | Compound Collider（多个 Primitive 组合） |
| 静态环境（地形、建筑） | MeshCollider |

### 参考链接

- [Collider types and performance](https://docs.unity3d.com/2022.3/Documentation/Manual/physics-optimization-cpu-collider-types.html) - 官方性能对比文档
- [Introduction to primitive collider shapes](https://docs.unity3d.com/2022.3/Documentation/Manual/primitive-colliders-introduction.html) - Primitive Collider 说明
- [Optimize physics performance](https://docs.unity3d.com/2022.3/Documentation/Manual/physics-optimization.html) - 物理系统优化总览

---

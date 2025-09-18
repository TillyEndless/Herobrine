---
title: 【Blender】3D建模测量不规则物体转动惯量
published: 2024-11-20T00:00:00.000Z
description: 会不断更新
image: ''
tags:
  - Blender
category: 技术杂项
draft: true
lang: ''
---
不断更新。

---
***目录***

<!-- toc -->

- [新手入门：](#%E6%96%B0%E6%89%8B%E5%85%A5%E9%97%A8)
- [GPT问答：如何使用blender进行不规则物体转动惯量测量实验：](#gpt%E9%97%AE%E7%AD%94%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8blender%E8%BF%9B%E8%A1%8C%E4%B8%8D%E8%A7%84%E5%88%99%E7%89%A9%E4%BD%93%E8%BD%AC%E5%8A%A8%E6%83%AF%E9%87%8F%E6%B5%8B%E9%87%8F%E5%AE%9E%E9%AA%8C)
  * [实验原理](#%E5%AE%9E%E9%AA%8C%E5%8E%9F%E7%90%86)
  * [高级方法：使用Blender Python脚本](#%E9%AB%98%E7%BA%A7%E6%96%B9%E6%B3%95%E4%BD%BF%E7%94%A8blender-python%E8%84%9A%E6%9C%AC)
  * [注意事项](#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)

<!-- tocstop -->

# 新手入门：
> Link:
> - [3D神器Blender教程来了！从入门到跨过门槛](https://www.uisdc.com/blender-introductory-tutorial)
> - [官方文档-刚体世界环境](https://docs.blender.org/manual/zh-hans/2.83/physics/rigid_body/world.html)

# GPT问答：如何使用blender进行不规则物体转动惯量测量实验：

在Blender中可以通过物理模拟和数学计算来测量不规则物体的转动惯量。以下是一个完整的实验步骤：

## 实验原理

转动惯量（Moment of Inertia）描述物体绕轴旋转时的惯性
￼
Blender可以通过物理引擎和物体属性计算出模拟的转动惯量。

具体步骤

1. 创建不规则物体
	1. 打开Blender，删除默认立方体（X键删除）。
	2. 添加一个不规则形状的物体，例如：
	* 使用 Shift+A > Mesh > UV Sphere 或 Icosphere，然后用 Sculpt Mode 改变形状。
	* 或导入一个自定义的 3D 模型。
2. 设置旋转轴
	1. 调整物体原点：
	* 选择物体，按 Tab 进入编辑模式。
	* 按 A 全选，调整物体使其绕某点旋转（例如放在原点）。
	* 退出编辑模式（Tab），在右侧属性面板设置原点为物体几何中心（Object > Set Origin > Origin to Geometry）。
	2. 添加旋转轴参考物体：
	* 创建一个细长圆柱或其他物体，作为旋转轴标记。
3. 启用物理引擎
	1.	在右侧属性面板，选择物体，启用刚体模拟：
	* Physics Properties（物理属性） > Rigid Body。
	* 设置：
	    - Type: Active
	    - Mass: 设置物体质量（假设为$m$）。
	    - Shape: Mesh
	2.	固定旋转轴：
	    - 对旋转轴设置为 Rigid Body > Type: Passive，并设置位置锁定。
4. 模拟旋转运动
	1. 给物体一个初始旋转：
	    - 选择物体，按 N 打开右侧面板，在 Rigid Body > Angular Velocity 中输入一个初始角速度（例如绕$Z$轴设置为 5 rad/s）。
	2. 播放动画（Space 键），观察物体的旋转运动。
5. 记录数据
	1. 在 Scene Properties 中启用 Rigid Body World，并设置时间步长为较小值（例如 0.01 秒）以提高精度。
	2. 用 Timeline 或 Graph Editor 记录角速度和力矩的变化。

6. 计算转动惯量
根据力矩和角速度的关系计算转动惯量：$ I = \tau / \alpha $
* $\tau$：力矩（在 Blender 中可以通过脚本测量）
* $\alpha$：角加速度（角速度变化率，可以从模拟中记录）

## 高级方法：使用Blender Python脚本
1. 打开 Scripting 界面，创建新脚本。
2. 使用以下代码计算转动惯量：
```py
import bpy
import mathutils

# 获取物体
obj = bpy.context.active_object

# 质量和角速度
mass = obj.rigid_body.mass
angular_velocity = obj.rigid_body.angular_velocity

# 计算转动惯量（假设绕Z轴）
inertia_z = mass * (angular_velocity.x**2 + angular_velocity.y**2)
print(f"转动惯量 (绕Z轴): {inertia_z}")
```
## 注意事项
1.	模型精度：确保不规则物体的网格足够细密以提高计算精度。
2.	物理参数：正确设置物体质量和旋转初始条件。
3.	单位一致性：Blender默认单位为米、千克和秒，确保输入符合单位制。


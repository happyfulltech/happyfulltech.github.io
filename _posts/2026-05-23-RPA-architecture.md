---
title: "RPA: 递归特权架构——可扩展隔离的统一模型"
date: 2026-05-23 12:00:00 +0800
categories: [research]
tags: [architecture, privilege, virtualization, security, risc-v]
description: 将特权配置从硬件固定转变为内存可配置，统一虚拟化、机密计算与沙箱场景
image: /assets/img/rpa-cover.png
license: false
---

## 摘要

现代处理器特权架构面临一个根本性限制：特权等级由硬件固定，无法按需扩展。x86提供四个固定的特权环，ARM 提供4个EL，RISC-V 定义三种特权模式。每种新需求——虚拟化、可信执行、内存保护——都需要独立的硬件扩展，导致架构碎片化。

本文提出**递归特权架构（Recursive Privilege Architecture, RPA）**，将特权配置从硬件固定转变为内存可配置。RPA 引入两个对称原语（`descend`、`ascend`），支持由系统资源而非硬件约束决定的理论无限特权层级。

### 核心特性

| 特性 | 实现方式 | 效果 |
|------|----------|------|
| 嵌套虚拟化 | 页表叠加 | 原生支持 |
| 系统调用优化 | INHERIT 模式 | 接近函数调用延迟 |
| 机密计算 | 安全域机制 | 管理权与访问权分离 |

## Index Terms

特权架构，虚拟化，机密计算，内存保护，处理器设计

---

## 论文在线阅读
 Copyright © 2025 Yongkang Liu

This is the submitted manuscript version. The final published
version will be available at IEEE Computer Architecture Letters.

[<i class="fas fa-file-pdf"></i> 在线阅读 PDF 全文](/assets/files/rpa-architecture.pdf)

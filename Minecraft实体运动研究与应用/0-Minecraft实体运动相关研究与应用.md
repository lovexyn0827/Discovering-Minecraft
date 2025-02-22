---
title: Minecraft实体运动研究与应用
date: false
keywords: Minecraft, 实体, 运动, 机制, lovexyn0827
desc: Minecraft实体运动研究与应用
id: motion_of_entities:0
draft: false
---

# Minecraft实体运动研究与应用

## 20250222 lovexyn0827

该报告的Word（并非最新）/PDF格式以及配套存档下载：

> https://wwz.lanzoui.com/b02iez40j 密码5a3e

Github仓库：

> https://github.com/lovexyn0827/Discovering-Minecraft

在线版本及更多文章

> [https://lovexyn0827.space/mcdocs/docs/](https://lovexyn0827.space/mcdocs/docs/)

## 目录

1 [绪论](1-Chapter-1.md)

   1.1 研究背景与意义

   1.2 本文主要研究思路 

   1.3 本文所需知识基础 

   1.4 内容概要

   1.5 一些特殊约定 

2 [实体运动基础](2-Chapter-2.md)

   2.1 实体运动基本属性及设定

   2.2 实体运算位置及实体间运算顺序

   2.3 Minecraft实体分类 

   2.4 Entity类定义的实体模型

   2.5 误差和一个重要Bug

3 [实体自由运动相关公式推导](3-Chapter-3.md)

   3.1 符号定义及公式推导

   3.2 公式的直接及拓展应用

4 [实体移动过程及碰撞机制](4-Chapter-4.md)

   4.1 基于Entity.move()方法

   4.2 基于raycast的弹射物自主移动

   4.3 两种移动过程的对比

5 [外界因素对运动的影响](5-Chapter-5.md)

   5.1 蛛网和浆果丛的减速作用

   5.2 粘液块和床的回弹

   5.3 蜂蜜块和灵魂沙的减速作用

   5.4 滑度机制

   5.5 气泡柱的变速作用

   5.6 粘液块的减速作用

   5.7 流体的变速作用

   5.8 活塞的推动和变速作用

   5.9 潜影盒和潜影贝的推动作用

   5.10 实体挤压（推动）

   5.11 骑乘

   5.12 鱼竿浮标的拉动

   5.13 击退

   5.14 方块对某些实体的推出作用

   5.15 爆炸的变速作用

   5.16 区块行为对运动的影响

   5.17 TP/折跃门传送

   5.18 跨维度运动

   5.19 拴绳机制

   5.20 蜜块侧面的减速作用

6 [LivingEntity运动机制](6-Chapter-6.md)

   6.1 LivingEntity的自由运动

   6.2 AI及属性对运动的影响

   6.3 鞘翅飞行

7 [几类具体实体运动分析](7-Chapter-7.md)

   7.1 TNT

   7.2 下落的方块

   7.3 物品

   7.4 经验球

   7.5 船

   7.6 矿车

   7.7 箭矢

   7.8 雪球

   7.9 恶魂火球

   7.10 玩家

8 [实体运动应用举例](8-Chapter-8.md)

   8.1 改造一种TNT大炮

   8.2 科学地使用弹射物

​		8.2.1 珍珠投掷技术

​		8.2.2 珍珠炮

​		8.2.3 杀凋大炮

   8.3 解释一些奇葩现象

[参考文献](9-References.md)

[附录A常见实体运动属性](10-Appendix.md)

[附录B完整公式](10-Appendix.md)

[附录C研究环境及辅助性工具](10-Appendix.md)

[附录D主要符号及单位](10-Appendix.md)

[附录E 定义索引](10-Appendix.md)

## 致谢

感谢 B 站用户**粗缯大布裹生涯**帮助优化该报告的Word格式的排版，并指出文中几处重要错误。

感谢 B 站用户**Yellowxuu**提出拴绳机制这一研究方向及几个关键问题，并探讨了船的摔落后的破碎问题。


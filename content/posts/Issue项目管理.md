# 使用GitLab管理项目

## 为什么是 GitLab？

GitLab 不仅可以托管代码，它还支持敏捷项目管理。无论是从非常简单的问题跟踪到看板和 Scrum 风格的敏捷交付。在 GitLab 上，您可以像使用专门的项目管理应用程序一样有效地管理问题、可视化整个项目的工作状态并跟踪工作量和解决优先级。相比于其他过于复杂而无法有效使用的应用程序更适合开发人员。

#### 实现

1. 每组通过独立项目统一管理issue，在**Readme**中描述使用方式及定义
2. 通过**milestone**管理项目版本，对齐目标。节奏很重要
3. 通过**label**管理优先级(`P0`|`P1`|`P2`)、类型(`bug`|`feature`)
4. 通过**board**查看**issue**进度状态，配合`To Do`、`Doing`、`Verify`等**label**，定义**issue生命周期**
5. 通过模板定义**author**需要提供的信息

## Milestone

通过 **milestone** 管理项目版本，对齐目标。

#### 短期里程碑

短期内需要达到的一个目标。

#### 季度里程碑

每个季度发布软件的多个版本。

#### 发布版本里程碑

一个可发布的版本，项目所有需求都已开发完成并测试通过，可交付给客户。

## Issue

为问题添加一些标签以快速识别涉及应用程序的哪个组件以及它是什么类型的问题（错误、功能请求等）非常重要。GitLab issue boards具有强大的过滤功能，通过过滤 Label 确定优先级并使团队与业务需求保持一致。

#### 标签示例

| 分类               | Label                | 简化版Label    | 说明                         |
| ------------------ | -------------------- | -------------- | ---------------------------- |
| 类型（type）       | 🟥type: bug           | 🟥bug           | 一个bug                      |
|                    | 🟦type: feature       | 🟦feature       | 新增功能的需求               |
|                    | 🟩type: enhancement   | 🟩enhancement   | 增强                         |
|                    | 🟪type: discussion    | 🟪discussion    | 问题讨论会议                 |
|                    | 🟨type: docs          | 🟨documentation | 文档相关的问题               |
|                    | 🟥type: security      | 🟥security      | 安全性相关的问题             |
|                    | 🟨type: test          | 🟨test          | 测试相关的问题               |
| 优先级（priority） | 🟥priority: critical  | 🟥P0            | 放下正在做的任何事情并处理它 |
|                    | 🟧priority: high      | 🟧P1            | 需要尽快处理                 |
|                    | 🟨priority: medium    | 🟨P2            | 按计划进行                   |
|                    | ⬜priority: low       | ⬜P3            | 当无事可做时处理             |
| 状态(state)        | 🟩state: approved     | 🟩approved      | 问题已接收但未处理           |
|                    | 🟥state: blocked      | 🟥blocked       | 问题由于其他事情阻塞无法进行 |
|                    | 🟨state: in progress  | 🟨in progress   | 正在处理中                   |
|                    | 🟦state: completed    | 🟦completed     | 已完成                       |
|                    | ⬜state: inactive     | ⬜inactive      | 问题已结束                   |
| 组件（component ） | 🟩component: frontend | 🟩frontend      | 前端                         |
|                    | 🟦component: backend  | 🟦backend       | 后端                         |
| 项目（project）    | 🟧计划                | 🟧计划          |                              |
|                    | 🟩开发中              | 🟩开发中        |                              |
|                    | ⬜已完成              | ⬜已完成        |                              |

## Issue template

##### Bug.md

```
## 概述

## Bug行为

## 期望行为

## 重现步骤

## 附件
<!-- URL/相关信息-id 号/截图 -->

```

##### Feature.md

```
## 背景
<!-- 为何要做，不做会有怎样的问题，做了会有怎样的收益 -->

## 需求说明

## 方案
<!-- 思路或模型 -->

## 验证
<!-- 如何验证，预期的标准是什么 -->

```

## 提交约定

建议应用此提交消息格式：

```
<type>(<scope>): <subject>
```

**type**可以在哪里：

- **feat** （新特性）
- **fix** （bug修复）
- **docs** （文档改动）
- **style** （格式化, 缺失分号等; 不包括生产代码变动）
- **refactor** （重构代码）
- **test** （添加缺失的测试, 重构测试, 不包括生产代码变动）
- **chore** （更新grunt任务等; 不包括生产代码变动）

**scope** 可能：

- **api**
- **front**
- **internal** 等等。

并且**subject**必须使用命令式，现在时。

```
feat(api): add users profile endpoint
```

完整提交消息示例：

```bash
<type>(<scope>): <subject>

<body>

<footer>
```

```
fix(middleware): ensure Range headers adhere more closely to RFC 2616

Add one new dependency, use `range-parser` (Express dependency) to compute
range. It is more well-tested in the wild.

Fixes #2310
```
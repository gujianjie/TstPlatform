# TstPlatform - Claude 协作记录

## 项目概述

TstPlatform 是一个测试平台项目，主要用于汽车电子测试领域，可能与TSMaster等工具集成。

---

## 2026-03-11 会话记录

### 本次目标
设计测试用例管理界面原型，作为示例供后续详细讨论。

### 完成工作

#### 1. Figma 设计创建
- **设计文件**: https://www.figma.com/design/IzInbpk8dr1n4YKQxEyaAc
- **File Key**: `IzInbpk8dr1n4YKQxEyaAc`
- **创建方式**: 通过HTML原型 + Figma MCP捕获工作流生成

#### 2. 本地文件结构
```
TstPlatform/
├── designs/
│   └── test-case-manager.html      # HTML原型文件
├── docs/
│   └── design/
│       └── test-case-manager-spec.html  # 设计规范文档
└── CLAUDE.md                       # 本文件
```

#### 3. 设计规范摘要

**布局结构**: 经典三栏布局
- **顶部导航栏**: 56px高度，包含品牌、主导航、操作按钮
- **左侧边栏**: 260px宽度，项目列表 + 用例集导航
- **中间内容区**: flex自适应，搜索/筛选 + 测试用例表格
- **右侧面板**: 360px宽度，用例详情 + 测试步骤

**颜色系统**:
- 主色: `#1677ff` (蓝色)
- 成功: `#52c41a` (绿色)
- 警告: `#fa8c16` (橙色)
- 错误: `#ff4d4f` (红色)
- 背景: `#f5f6f7` (浅灰)

**核心功能展示**:
- 项目管理: BCM控制器、PEPS系统、DCU测试项目
- 测试用例列表: ID、名称、类型、优先级、状态、日期、操作
- 用例详情: 基本信息、测试步骤、关联需求追溯
- 示例数据: 发动机启动测试、故障码读取、CAN报文周期等汽车电子测试用例

### 关键技术点

#### Figma + Claude Code 工作流
1. 创建HTML原型文件 (`designs/test-case-manager.html`)
2. 启动本地HTTP服务器 (`python -m http.server 8888`)
3. 使用Figma MCP工具捕获设计
4. 在Figma桌面应用或网页中查看和编辑

**注意**: Figma设计存储在云端，本地工程目录保存设计规范文档而非.figma源文件。

### 待讨论事项

用户明确表示当前设计"只是个示例，还需要详细讨论"。待讨论内容可能包括：

1. **功能细节**: 与TSMaster工具集成的具体界面需求
2. **交互流程**: 测试执行、报告生成等完整工作流
3. **数据模型**: 测试用例、步骤、需求的数据结构设计
4. **参考对比**: 是否需要对比Storm UTP等现有工具的功能
5. **技术栈**: 前端框架选择（React/Vue/Electron等）

### 相关资源

- **Figma设计**: https://www.figma.com/design/IzInbpk8dr1n4YKQxEyaAc
- **设计规范文档**: `docs/design/test-case-manager-spec.html`
- **HTML原型**: `designs/test-case-manager.html`

---

## 项目上下文

### 背景知识
- 项目涉及汽车电子测试领域
- 可能与TSMaster（汽车总线分析工具）集成
- 参考过Storm UTP（统一测试平台）的相关资料

### 开发环境
- 工作目录: `/Users/gujianjie/Documents/Projects/TstPlatform`
- 系统: macOS (Darwin 25.3.0)
- 工具: Figma MCP, Pencil MCP

---

*最后更新: 2026-03-11*

# 测试平台技术调研报告

## 一、工具集成技术分析

### 1. TSMaster（优先绑定工具）

#### 集成方式对比

| 集成方式 | 技术原理 | 优点 | 缺点 | 适用场景 |
|---------|---------|------|------|---------|
| **TSMaster API (COM)** | Windows COM自动化技术 | 功能最全，官方推荐，支持多语言 | 仅Windows，需安装TSMaster | 桌面应用集成 |
| **libTSCAN API** | 底层C库封装 | 跨平台(Windows/Linux)，轻量级 | 功能相对有限 | 嵌入式/边缘部署 |
| **小程序库(MPC)** | TSMaster内部扩展 | 可独立运行，可被外部调用 | 依赖TSMaster环境 | 特定功能扩展 |
| **RPC接口** | 远程过程调用 | 支持远程测试，分布式部署 | 网络依赖，配置复杂 | 远程/云端测试 |
| **图形化编程** | 可视化流程设计 | 无需编程，Excel转测试流程 | 灵活性受限 | 简单自动化场景 |

#### 推荐技术方案

**阶段一（MVP）**：TSMaster COM API + Python/C#
- 理由：功能完整，开发效率高，官方文档齐全
- 技术栈：Python 3.11+ 或 .NET 8 + C#
- 集成点：
  - 测试执行控制（启动/停止/监控）
  - 信号/变量读写
  - 测试报告获取
  - 自动化流程编排

**阶段二（扩展）**：libTSCAN API
- 用于：Linux环境部署、轻量级数据采集、边缘计算场景

#### TSMaster API 核心能力

```python
# 典型集成示例
import win32com.client

# 连接TSMaster
tsmaster = win32com.client.Dispatch("TSMaster.Application")

# 打开工程
tsmaster.open_project("test.cfg")

# 控制测试执行
tsmaster.start_measurement()
tsmaster.set_signal_value("EngineSpeed", 3000)
result = tsmaster.get_test_result()
tsmaster.stop_measurement()
```

### 2. CANoe（第二优先级）

#### 集成方式

| 方式 | 说明 | 复杂度 |
|-----|------|-------|
| **COM API** | 通过Python/C#调用CANoe对象 | 中等 |
| **vTESTstudio** | 配合测试设计工具，生成VTUEXE | 较高 |
| **FDX协议** | 高性能数据交换，适合HIL | 高 |

#### COM API 核心对象

- `Application` - CANoe应用实例
- `Configuration` - 工程配置
- `Measurement` - 测量控制
- `TestSetup` - 测试环境
- `TestConfiguration` - 测试配置执行

```python
# CANoe COM 示例
import win32com.client

canoe = win32com.client.Dispatch("CANoe.Application")
canoe.Open(r"test.cfg")
canoe.Measurement.Start()

# 执行测试单元
test_env = canoe.Configuration.TestSetup.TestEnvironments.Item(1)
test_module = test_env.TestModules.Item("TestModule")
test_module.Start()
```

### 3. ECU-TEST（第三优先级）

#### 特点
- 图形化测试用例设计
- 原生支持Python扩展
- 强大的跨工具集成能力
- 符合ASPICE流程规范

#### 集成方式
- 标准：XIL API（ASAM标准）
- 自定义：Python测试步骤
- 报告：XML/JSON格式导出

---

## 二、技术架构方案对比

### 方案A：桌面应用（C# WPF/WinUI）

```
┌─────────────────────────────────────┐
│           桌面客户端 (C#)            │
│  ┌──────────┐ ┌──────────┐         │
│  │ 测试管理  │ │ 用例设计  │         │
│  └──────────┘ └──────────┘         │
└──────────────┬──────────────────────┘
               │ COM API
┌──────────────▼──────────────────────┐
│         工具适配层 (C#/Python)       │
│  ┌─────────┐ ┌─────────┐ ┌────────┐│
│  │ TSMaster│ │  CANoe  │ │ECU-TEST││
│  │ COM API │ │ COM API │ │ XIL API││
│  └─────────┘ └─────────┘ └────────┘│
└─────────────────────────────────────┘
```

**优点**：
- 与Windows工具生态集成好
- UI交互体验佳
- 开发工具成熟（Visual Studio）

**缺点**：
- 仅Windows平台
- 部署需安装客户端

### 方案B：Web应用（B/S架构）

```
┌─────────────────────────────────────┐
│           浏览器 (前端)              │
│        React/Vue + TypeScript       │
└──────────────┬──────────────────────┘
               │ HTTP/WebSocket
┌──────────────▼──────────────────────┐
│         后端服务 (Python/FastAPI)    │
│  ┌──────────┐ ┌──────────┐         │
│  │ 测试管理  │ │ 工具代理  │         │
│  └──────────┘ └──────────┘         │
└──────────────┬──────────────────────┘
               │ COM API / RPC
┌──────────────▼──────────────────────┐
│         工具适配服务 (Windows)       │
│  ┌─────────┐ ┌─────────┐ ┌────────┐│
│  │ TSMaster│ │  CANoe  │ │ECU-TEST││
│  └─────────┘ └─────────┘ └────────┘│
└─────────────────────────────────────┘
```

**优点**：
- 跨平台访问
- 易于部署和更新
- 天然支持远程协作

**缺点**：
- 需要工具代理服务层
- 实时性不如桌面应用
- 复杂度更高

### 方案C：混合架构（推荐）

```
┌─────────────────────────────────────┐
│  桌面客户端 (轻量)  │  Web管理端    │
│    (C# WinUI)    │  (React)      │
└────────┬──────────┴────────┬───────┘
         │                   │
         │    ┌─────────┐    │
         └───►│ 后端API │◄───┘
              │ (Python)│
              └────┬────┘
                   │
         ┌─────────┼─────────┐
         ▼         ▼         ▼
    ┌────────┐ ┌────────┐ ┌────────┐
    │TSMaster│ │ CANoe  │ │ECU-TEST│
    │适配器  │ │适配器  │ │适配器  │
    └────────┘ └────────┘ └────────┘
```

**核心思想**：
- 桌面端：测试执行（需要与工具紧耦合）
- Web端：用例管理、报告查看、协同工作
- 后端：数据管理、任务调度

---

## 三、推荐技术栈

### 首选方案：桌面应用（阶段一）

| 层级 | 技术选型 | 理由 |
|-----|---------|------|
| **UI框架** | WPF 或 WinUI 3 | Windows原生，工具集成友好 |
| **后端逻辑** | C# (.NET 8) | 与COM工具集成最佳 |
| **脚本扩展** | Python 3.11 | 测试工程师熟悉，生态丰富 |
| **数据存储** | SQLite (本地) + PostgreSQL (团队) | 轻量到企业级渐进 |
| **报告生成** | PDF (iText/QuestPDF) + HTML | 满足交付需求 |

### 备选方案：Web优先（阶段二）

| 层级 | 技术选型 | 理由 |
|-----|---------|------|
| **前端** | React + TypeScript | 生态成熟，团队易招聘 |
| **后端** | Python (FastAPI) | 与测试工具生态一致 |
| **工具代理** | Python COM桥接 | 复用桌面端适配器逻辑 |
| **数据库** | PostgreSQL | 开源可靠，扩展性好 |
| **消息队列** | Redis/RabbitMQ | 异步任务、实时通知 |

---

## 四、关键实现要点

### 1. 与TSMaster紧耦合设计

```csharp
// 示例：工具适配器接口
public interface ITSMasterAdapter
{
    // 连接管理
    bool Connect();
    void Disconnect();

    // 工程管理
    bool OpenProject(string path);
    void CloseProject();

    // 测试执行
    void StartMeasurement();
    void StopMeasurement();
    TestResult RunTestCase(TestCase tc);

    // 信号操作
    void SetSignal(string name, object value);
    object GetSignal(string name);

    // 事件监听
    event EventHandler<TestEvent> OnTestEvent;
}
```

### 2. 测试用例存储格式

考虑到工具紧耦合，用例存储可以采用：
- **元数据**：JSON/YAML（用例描述、参数、预期结果）
- **执行脚本**：Python/C#（与工具特定的执行逻辑）

```yaml
# 用例元数据示例
test_case:
  id: TC001
  name: 发动机启动测试
  tool: TSMaster

  parameters:
    - name: EngineSpeed
      type: signal
      value: 3000

  steps:
    - action: set_signal
      target: Ignition
      value: 1
    - action: wait
      duration: 2000
    - action: check_signal
      target: EngineSpeed
      condition: "> 2800"
```

### 3. 报告系统集成

TSMaster和CANoe都支持：
- 自动生成测试报告
- 自定义报告模板
- 结果导出（XML/JSON）

建议：复用工具原生报告 + 平台层汇总分析

---

## 五、与Storm UTP的技术差异

| 维度 | Storm UTP | 我们的方案（建议） |
|-----|-----------|------------------|
| **架构** | B/S架构 | 桌面为主，可选Web |
| **工具集成** | 通用接口，多工具兼容 | 紧耦合，TSMaster优先 |
| **智能化** | 智能推荐算法 | 初期规则驱动，后期扩展 |
| **部署** | 私有化部署+云服务 | 本地桌面+可选服务器 |
| **技术栈** | 未知（可能是Java/Web） | C#/Python/.NET |
| **扩展性** | 强（通用设计） | 针对特定工具优化 |

---

## 六、技术风险与应对

| 风险 | 影响 | 应对措施 |
|-----|------|---------|
| TSMaster API变更 | 高 | 封装适配器层，隔离变化 |
| 工具授权限制 | 中 | 设计上支持多工具切换 |
| 性能瓶颈 | 中 | 异步执行，避免UI阻塞 |
| 跨团队使用 | 低 | 后期增加Web管理层 |

---

## 七、建议实施路线

### 阶段一（2-3个月）：核心功能
- C#桌面应用框架
- TSMaster基础集成（执行、信号、报告）
- 本地用例管理（SQLite）

### 阶段二（1-2个月）：功能扩展
- CANoe集成
- 用例编辑器（可视化）
- 报告分析与展示

### 阶段三（可选）：协作能力
- Web管理端
- 团队数据同步
- CI/CD集成

---

*调研日期：2026-03-11*
*调研工具：Tavily API*

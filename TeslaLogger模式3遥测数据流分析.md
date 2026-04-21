# TeslaLogger 模式3：本地 Fleet Telemetry 数据流分析

> 分析时间：2026-04-21
> 源码路径：/Users/zhangcheng/TeslaLogger

---

## 一、架构概览

```
特斯拉车辆
    │
    │ TLS / HTTPS (Fleet Telemetry 协议)
    ▼
fleet-telemetry 服务器（Tesla 官方开源，Go 语言）
    │  github.com/teslamotors/fleet-telemetry
    │  部署在局域网或公网（需有效 TLS 证书）
    │
    │ ZeroMQ PUB-SUB 协议
    │ tcp://192.x.x.x:5284
    ▼
TeslaLogger（C# / Mono）
    │
    ├─ TelemetryConnectionZMQ.cs   ← ZMQ 订阅者，接收原始消息
    ├─ TelemetryParser.cs           ← 消息解析 + 数据分类路由
    │
    ▼  按字段类型分类写入不同数据表
  ┌──────────┬──────────┬────────────┬──────────┬──────────┬──────────────┐
  │   pos    │  battery │ cruisestate│ charging │  alerts  │  CurrentJSON │
  │ （位置）  │ （电池）  │ （巡航）   │ （充电）  │ （警报）  │  （实时状态） │
  └──────────┴──────────┴────────────┴──────────┴──────────┴──────────────┘
```

**核心特点：**
- 数据完全不经过任何第三方服务器，100% 本地化
- 采样频率最高可达 100ms（视字段配置）
- 需要在车辆中安装 Virtual Key 并授权自定义服务器域名

---

## 二、先决条件配置

### 2.1 fleet-telemetry 服务器配置

Tesla 官方 fleet-telemetry 服务器需要开放 ZeroMQ 端口对本地 TeslaLogger 进行数据转发。
服务器启动后在 `tcp://0.0.0.0:5284` 上监听 PUB socket。

### 2.2 TeslaLogger 应用设置

在 `ApplicationSettings` 中配置两个关键参数：

```
TelemetryServerType = "ZMQ"              # 选择本地模式
TelemetryServerURL  = "tcp://192.168.x.x:5284"   # ZMQ 服务地址
```

### 2.3 遥测字段订阅配置（LocalTelemetry.md）

TeslaLogger 提供了约 50 个字段的订阅配置模板，推送给 fleet-telemetry 服务器，控制每个字段的上报频率：

| 字段 | 采样间隔 | 重发间隔 | 最小变化量 | 说明 |
|------|---------|---------|----------|------|
| `Location` | 1s | — | 3m | GPS 坐标 |
| `VehicleSpeed` | 1s | — | — | 车速 |
| `DCChargingPower` | 1s | — | — | 直流充电功率 |
| `ACChargingPower` | 1s | — | — | 交流充电功率 |
| `ACChargingEnergyIn` | 1s | — | — | 交流充电累计能量 |
| `DCChargingEnergyIn` | 1s | — | — | 直流充电累计能量 |
| `Gear` | 5s | — | — | 档位（P/D/R/N） |
| `ChargeState` | 5s | — | — | 充电状态 |
| `PackVoltage` | 10s | — | — | 电池组电压 |
| `PackCurrent` | 10s | — | — | 电池组电流 |
| `BrickVoltageMin/Max` | 10s | — | — | 单体电压极值 |
| `ModuleTempMin/Max` | 10s | — | — | 模组温度极值 |
| `TpmsPressure*` | 10s | 300s | 0.01 | 轮胎气压（四轮） |
| `Soc` | 15s | 300s | 0.1 | 电池电量 |
| `IdealBatteryRange` | 15s | — | — | 理想续航 |
| `RatedRange` | 15s | — | — | 标定续航 |
| `OutsideTemp` | 60s | — | 0.1 | 外部温度 |
| `InsideTemp` | 60s | — | 0.1 | 车内温度 |
| `SentryMode` | 60s | — | — | 哨兵模式状态 |
| `Locked` | 60s | — | — | 车门锁状态 |
| `FastChargerPresent` | 30s | — | — | 是否接入直流桩 |
| `Version` | 600s | — | — | 固件版本号 |
| `CarType` | 600s | — | — | 车型 |
| `VehicleName` | 600s | — | — | 车辆昵称 |
| `Trim` | 600s | — | — | 配置级别 |

---

## 三、ZMQ 连接层（TelemetryConnectionZMQ.cs）

### 3.1 类结构

```csharp
class TelemetryConnectionZMQ : TelemetryConnection
{
    private Car car;
    private TelemetryParser parser;
    
    void Run()        // 主线程入口
    void Connect()    // 建立 ZMQ SUB 连接
}
```

### 3.2 连接建立流程

```
Run()
  └─ Connect()
       ├─ new SubscriberSocket()
       ├─ socket.Connect("tcp://192.x.x.x:5284")
       ├─ socket.Subscribe("")    // 订阅所有 topic
       └─ 进入消息循环
```

### 3.3 消息接收循环

```csharp
while (true)
{
    Thread.Sleep(100);   // 100ms 轮询间隔
    
    var msg = zmqsubscriber.ReceiveMultipartMessage(timeout: ...);
    // msg[0] = topic（车辆 VIN 等）
    // msg[1] = JSON 消息体
    
    string json = msg[1].ConvertToString();
    parser.handleMessage(json);   // 交给解析器处理
}
```

### 3.4 错误处理与重连

```
异常发生
  └─ 关闭当前 socket
  └─ Random.Next(30000, 60000) ms 随机等待
  └─ 重新 Connect()
```

---

## 四、消息解析与路由（TelemetryParser.cs）

### 4.1 消息格式

fleet-telemetry 推送的 JSON 消息结构：

```json
// 遥测数据消息
{
  "vin": "LRWXXXXXXXXXXX",
  "data": [
    { "key": "Location", "value": { "locationValue": { "latitude": 39.9, "longitude": 116.4 } } },
    { "key": "VehicleSpeed", "value": { "doubleValue": 65.3 } },
    { "key": "Gear", "value": { "stringValue": "D" } },
    { "key": "Soc", "value": { "doubleValue": 78.5 } }
    // ... 更多字段
  ],
  "createdAt": "2024-01-01T12:00:00Z"
}

// 警报消息
{
  "vin": "LRWXXXXXXXXXXX",
  "alerts": [
    { "name": "ALERT_NAME", "startedAt": "...", "audiences": ["Driver"] }
  ]
}

// 登录响应
{
  "Teslalogger": { ... }
}
```

### 4.2 入口方法 handleMessage()

```
handleMessage(string json)
  │
  ├─ 解析 JSON，读取顶层 key
  │
  ├─ key == "data"
  │    └─ 依次调用 6 个处理方法（见第五章）
  │
  ├─ key == "alerts"
  │    └─ InsertAlert()
  │
  └─ key == "Teslalogger"
       └─ handleLoginResponse()
```

---

## 五、数据分类写入详解

接收到 `"data"` 消息后，TelemetryParser 依次调用以下 6 个方法，每个方法负责处理特定字段并写入对应数据表：

### 5.1 InsertBatteryTable → `battery` 表

**触发字段：**

| 字段 Key | 含义 |
|---------|------|
| `PackVoltage` | 电池组总电压（V） |
| `PackCurrent` | 电池组电流（A，正=放电，负=充电） |
| `BrickVoltageMin` | 单体电池最低电压 |
| `BrickVoltageMax` | 单体电池最高电压 |
| `ModuleTempMin` | 模组最低温度 |
| `ModuleTempMax` | 模组最高温度 |
| `IsolationResistance` | 绝缘电阻 |
| `LifetimeEnergyUsed` | 累计使用能量 |

**写入逻辑：**

```
接收到以上任一字段
  └─ 更新内存中对应变量
  └─ 当 PackVoltage AND PackCurrent 均已获取
       └─ 动态构造 INSERT SQL
       └─ INSERT INTO battery (CarID, Datum, ...) VALUES (...)
```

---

### 5.2 InsertCruiseStateTable → `cruisestate` 表

**触发字段：**

| 字段 Key | 原始值 | 存储值 |
|---------|-------|-------|
| `CruiseState` | Off | 0 |
| `CruiseState` | On | 1 |
| `CruiseState` | Override | 2 |
| `CruiseState` | Standby | -1 |
| `CruiseState` | Standstill | -2 |

**写入逻辑：**

```
接收到 CruiseState 字段
  └─ 字符串映射为整数
  └─ INSERT INTO cruisestate (CarID, Datum, state) VALUES (...)
```

---

### 5.3 handleStatemachine → 状态机控制（不直接写 DB，触发其他写入）

这是最核心的路由方法，根据多个字段综合判断车辆状态，控制 `Driving`、`acCharging`、`dcCharging` 三个布尔标志，从而触发行程/充电的开始与结束。

**监控字段：**

| 字段 | 用途 |
|-----|-----|
| `Gear` | 档位（P/D/R/N） |
| `ChargeState` | 充电状态字符串 |
| `VehicleSpeed` | 车速（m/s 或 km/h） |
| `Location` | 当前 GPS 坐标 |
| `FastChargerPresent` | 是否接入直流快充桩 |
| `DCChargingPower` | 直流充电功率 |
| `PackCurrent` | 电池电流（判断实际充电中） |

**状态转换逻辑：**

```
行驶状态检测：
  Gear != "P" AND Gear != null
    → Driving 未开启  → InsertFirstPos() → 开始行程 → Driving = true
  Gear == "P"
    → Driving 已开启  → StopDriving()  → Driving = false

交流充电检测：
  ChargeState == "Enable" AND PackCurrent > 2A
    → acCharging 未开启 → StartACCharging() → acCharging = true
  DetailedChargeState == "DetailedChargeStateStopped"
    → acCharging 已开启 → StopACCharging()  → acCharging = false

直流充电（超充）检测：
  FastChargerPresent == true AND PackCurrent > 5A
    → dcCharging 未开启 → StartDCCharging() → dcCharging = true
  FastChargerPresent == false OR PackCurrent < 2A
    → dcCharging 已开启 → StopDCCharging()  → dcCharging = false
```

---

### 5.4 InsertLocation → `pos` 表

**触发字段：**

| 字段 Key | 含义 |
|---------|------|
| `Location` | GPS 坐标（locationValue 或 "lat,lng" 字符串） |
| `Odometer` | 里程表（km） |
| `VehicleSpeed` | 车速 |

**写入逻辑：**

```
接收到 Location 字段
  └─ 解析经纬度
  └─ InsertLastLocation(lat, lng, odometer, speed)
       └─ 更新 car.CurrentJSON 实时状态
       └─ 如果 Driving == true
            └─ car.DbHelper.InsertPos(lat, lng, speed, odometer, ...)
                 └─ INSERT INTO pos (CarID, Datum, lat, lng, speed, odometer, ...) VALUES (...)
```

> 注意：仅在行驶状态（Driving=true）下才向 `pos` 表写入轨迹数据，静止时只更新内存状态。

---

### 5.5 InsertCharging → `charging` 表

分为两层：**字段解析层** 和 **数据库写入层**。

**触发字段：**

| 字段 Key | 含义 |
|---------|------|
| `ACChargingEnergyIn` | 交流充电累计能量（kWh） |
| `ACChargingPower` | 交流充电功率（kW） |
| `DCChargingEnergyIn` | 直流充电累计能量（kWh） |
| `DCChargingPower` | 直流充电功率（kW） |
| `Soc` | 当前电量（%） |
| `IdealBatteryRange` | 理想续航（km） |
| `RatedRange` | 标定续航（km） |

**写入逻辑：**

```
接收到充电相关字段
  └─ 更新内存中对应变量
  └─ 当 IsCharging == true（acCharging 或 dcCharging 为 true）
       └─ 构造动态 SQL
       └─ INSERT INTO charging (CarID, Datum, charger_power, charge_energy_added, 
                               battery_level, ideal_battery_range_km, rated_range_km, ...)
          ON DUPLICATE KEY UPDATE ...
```

---

### 5.6 InsertStates → 多目标写入

负责处理所有状态类字段，写入多个目标：

**目标1：`pos` 表（温度数据补充）**

| 字段 Key | 含义 |
|---------|------|
| `OutsideTemp` | 外部气温 |
| `InsideTemp` | 车内气温 |

```
接收到温度字段
  └─ InsertLastLocation(with temperature)
       └─ INSERT INTO pos (..., outside_temp, inside_temp) ...（当行驶时）
```

**目标2：`TPMS` 表（轮胎气压）**

| 字段 Key | 含义 |
|---------|------|
| `TpmsPressureFl` | 左前轮胎气压（bar） |
| `TpmsPressureFr` | 右前轮胎气压（bar） |
| `TpmsPressureRl` | 左后轮胎气压（bar） |
| `TpmsPressureRr` | 右后轮胎气压（bar） |

```
四轮气压均已接收
  └─ InsertTPMS()
       └─ INSERT INTO TPMS (CarID, Datum, fl, fr, rl, rr) VALUES (...)
```

**目标3：`CurrentJSON` 实时状态（内存，不写 DB）**

以下字段仅更新内存中的实时状态对象，用于 Web UI 展示：

| 字段 Key | 说明 |
|---------|------|
| `SentryMode` | 哨兵模式开/关 |
| `PreconditioningEnabled` | 预热/预冷状态 |
| `DoorState` | 车门开关状态 |
| `Locked` | 车门锁状态 |
| `*Window` | 各车窗开关状态 |
| `BatteryHeaterOn` | 电池加热器状态 |
| `VehicleName` | 车辆昵称 |
| `Trim` | 配置级别 |
| `CarType` | 车型 |
| `Version` | 固件版本 |
| `SoftwareUpdateInstallationPercentComplete` | OTA 安装进度 |

```
接收到以上字段
  └─ car.CurrentJSON.xxx = value
  └─ 通过 MQTT 广播实时状态更新（如果 MQTT 已配置）
```

---

### 5.7 InsertAlert → `alerts` 表

**数据结构：**

```json
{
  "alerts": [
    {
      "name": "VCFRONT_a184",
      "startedAt": "2024-01-01T10:00:00Z",
      "endedAt":   "2024-01-01T10:05:00Z",
      "audiences": ["Driver", "Mechanic"]
    }
  ]
}
```

**写入逻辑：**

```
接收到 alerts 消息
  └─ 遍历每个 alert
       ├─ 查找或创建 alert_names 记录（nameID）
       ├─ INSERT INTO alerts (CarID, startedAt, nameID, endedAt)
       │    ON DUPLICATE KEY UPDATE endedAt = ...
       └─ 遍历 audiences
            └─ INSERT INTO alert_audiences (alertID, audience)
                 ON DUPLICATE KEY IGNORE
```

---

## 六、完整数据流时序图

```
特斯拉车辆
    │  Fleet Telemetry 推送（TLS）
    ▼
fleet-telemetry 服务器（Go）
    │  ZeroMQ PUB  tcp://192.x.x.x:5284
    │  消息格式：[topic][JSON body]
    ▼
TelemetryConnectionZMQ.Run()
    │  Thread.Sleep(100ms)
    │  ReceiveMultipartMessage()
    ▼
TelemetryParser.handleMessage(json)
    │
    ├─────────────────────────────────────────────────────────────┐
    │  解析 data[] 数组中每个 {key, value} 对                      │
    │                                                             │
    ├─ 遍历字段 key                                               │
    │    ├─ PackVoltage / PackCurrent / ...  → InsertBatteryTable │
    │    ├─ CruiseState                      → InsertCruiseState  │
    │    ├─ Gear / ChargeState / ...         → handleStatemachine │
    │    ├─ Location / Odometer / Speed      → InsertLocation     │
    │    ├─ ACChargingPower / Soc / ...      → InsertCharging     │
    │    └─ OutsideTemp / TPMS / Version ... → InsertStates      │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
         │              │              │              │
         ▼              ▼              ▼              ▼
     battery        cruisestate       pos           charging
      table           table          table           table
                                      ↑
                                   TPMS table
                                   alerts table
                                   CurrentJSON (内存)
```

---

## 七、与模式1（REST API 轮询）的对比

| 对比项 | 模式1：REST API 轮询 | 模式3：本地 Fleet Telemetry |
|-------|-------------------|--------------------------|
| 数据方向 | TeslaLogger 主动拉取 | 车辆主动推送 |
| 最高频率 | 行驶中 5s/次 | Location/Speed 1s，可降至 100ms |
| 字段数量 | ~50 个（固定组合） | 100+ 个，可自定义订阅 |
| 网络依赖 | 需持续访问 Tesla 服务器 | 仅局域网，完全离线可用 |
| 数据隐私 | 数据经过 Tesla 服务器 | 数据不离开本地网络 |
| 唤醒影响 | 频繁轮询可能阻止车辆休眠 | 推送模式，不影响休眠 |
| 部署复杂度 | 简单，只需 OAuth Token | 需要配置 fleet-telemetry 服务器 + Virtual Key + TLS |
| 数据完整性 | 可能因网络延迟丢失数据点 | 车辆本地缓存，断网后自动补发 |
| 电池相关字段 | 无单体电压、模组温度等底层数据 | 包含 BrickVoltage、ModuleTemp 等详细数据 |

---

## 八、特殊机制说明

### 8.1 数据写入的条件性

**并非所有接收到的遥测字段都会立即写入数据库**，写入条件由状态机控制：

- `pos` 表写入：**仅当 `Driving == true` 时**
- `charging` 表写入：**仅当 `IsCharging == true` 时**（`acCharging || dcCharging`）
- `battery` 表写入：需要 PackVoltage AND PackCurrent **同时到达**后才触发
- `TPMS` 表写入：需要**四轮气压全部到达**后才触发一次写入

### 8.2 内存聚合与批量写入

各方法先将字段值聚合在内存中，满足条件后一次性写入，避免单字段到达就频繁写 DB（因为字段间会有时间差）。

### 8.3 ON DUPLICATE KEY UPDATE

`charging` 表和 `alerts` 表使用 `ON DUPLICATE KEY UPDATE` 语义，确保幂等性 — 同一时刻的数据重复推送不会产生重复行。

### 8.4 ZMQ 100ms 轮询

ZMQ 接收循环每 100ms 主动尝试读取一次消息，这并非阻塞式等待。如果没有新消息则跳过，保持 CPU 占用可控。实际数据频率由 fleet-telemetry 服务器侧的字段配置决定（最快 1s）。

---

## 九、总结

TeslaLogger 模式3 的核心设计是：

1. **分层解耦**：ZMQ 传输层（TelemetryConnectionZMQ）与业务解析层（TelemetryParser）分离
2. **字段路由**：按 JSON key 分类路由，同一消息的不同字段写入不同数据表
3. **状态驱动写入**：以车辆状态机（行驶/充电）为门控条件，而非所有字段盲目写入
4. **完全本地化**：整条数据链路（车辆 → fleet-telemetry → ZMQ → TeslaLogger → DB）均在本地网络内，零第三方依赖
5. **高频底层数据**：获得 REST API 无法提供的电池单体电压、模组温度、CAN 级别数据

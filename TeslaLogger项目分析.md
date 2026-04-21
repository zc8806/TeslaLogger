# TeslaLogger 项目分析

> 分析时间：2026-04-21
> 项目地址：https://github.com/bassmaster187/TeslaLogger
> 本地路径：/Users/zhangcheng/TeslaLogger

---

## 一、项目概述

TeslaLogger 是一个开源的自托管 Tesla 车辆数据记录系统，由德国开发者 bassmaster187 维护。
与 TeslaMate 定位相同，但技术栈不同，且已完整支持 Tesla 新版 **Fleet Telemetry** 数据推送接口。

---

## 二、技术栈

| 层级 | 技术 |
|-----|-----|
| 后端核心 | C# / .NET Framework 4.8（Mono 6.12 运行时） |
| 数据库 | MySQL / MariaDB 10.4.7 |
| Web 前端 | PHP 7.3 + Apache |
| 可视化 | Grafana 10.0.1 |
| 消息队列 | MQTT（M2Mqtt 4.3.0）、ZeroMQ（NetMQ 4.0.1） |
| 序列化 | Google.Protobuf 3.25.1（Fleet Telemetry 数据解析） |
| 加密 | BouncyCastle 2.2.1 |
| JSON | Newtonsoft.Json 13.0.3 |
| 错误追踪 | Exceptionless 6.0.3 |

---

## 三、项目目录结构

```
TeslaLogger/
├── TeslaLogger/              # 核心 C# 应用（50 个类）
│   ├── www/                  # PHP Web 管理界面
│   ├── GrafanaPlugins/       # 自定义 Grafana 插件
│   ├── GrafanaDashboards/    # 预配置 Grafana 仪表板
│   ├── bin/                  # 编译输出（约 256MB，不上传）
│   ├── screenshots/          # 截图文档
│   └── sqlschema.sql         # 数据库建表 SQL
├── docker/                   # Docker 配置文件
│   ├── teslalogger/          # C# 应用 Dockerfile（Mono 环境）
│   └── webserver/            # PHP/Apache Dockerfile
├── MQTTClient/               # MQTT 集成库
├── Logfile/                  # 日志管理库
├── UnitTestsTeslalogger/     # 单元测试
├── Teslamate-Import/         # 从 TeslaMate 迁移数据工具
├── TeslaFi-Import/           # 从 TeslaFi 迁移数据工具
├── OSMMapGenerator/          # OpenStreetMap 瓦片生成
├── TLUpdate/                 # 自动更新模块
├── KML_Import/               # KML 地理数据导入
├── srtm/                     # SRTM 高程数据处理库
├── docs/                     # 文档
└── docker-compose.yml        # 容器编排配置
```

---

## 四、核心模块说明

| 类文件 | 代码量 | 职责 |
|-------|-------|-----|
| `Car.cs` | ~2245 行 | 车辆主控制器，状态机管理 |
| `WebHelper.cs` | ~5436 行 | Tesla API 通信、Token 管理、数据采集 |
| `DBHelper.cs` | ~7507 行 | 数据库操作层 |
| `TelemetryParser.cs` | ~1500 行 | Fleet Telemetry 数据解析处理 |
| `WebServer.cs` | ~2000 行 | 内置 HTTP 服务器、REST API |
| `TelemetryConnectionWS.cs` | ~229 行 | WebSocket 连接管理（Fleet Telemetry 云端） |
| `TelemetryConnectionZMQ.cs` | ~200 行 | ZeroMQ 连接管理（Fleet Telemetry 本地） |
| `MQTT.cs` | — | MQTT 消息发布 |
| `Geofence.cs` | — | 地理围栏与 POI 管理 |
| `CurrentJSON.cs` | — | 实时状态数据模型 |

---

## 五、数据采集方式（★ 核心亮点）

TeslaLogger 支持三种数据采集模式，可按需切换：

### 模式 1：REST API 轮询（传统方式）

与 TeslaMate 相同，定期调用 Tesla Owner API：

```
GET /api/1/vehicles/{id}/vehicle_data
    ?endpoints=drive_state;location_data;climate_state;vehicle_state;charge_state;vehicle_config
```

| 车辆状态 | 轮询间隔 |
|---------|---------|
| 行驶中 | 5 秒 |
| 充电中 | 10 秒 |
| 在线静止 | 30 秒 |
| 休眠 | 10 秒（检查是否唤醒） |

### 模式 2：Fleet Telemetry 云端中转

```
车辆 → Tesla 服务器 → teslalogger.de:4501（作者运营）→ 本地 TeslaLogger
```

- 协议：WebSocket，数据格式：Protobuf
- 需要在 Tesla 账户中授权并配置 Virtual Key
- 数据字段更丰富，延迟更低

### 模式 3：本地 Fleet Telemetry（完全自托管）

```
车辆 → 本地 fleet-telemetry 服务器（Tesla 官方开源）
         ↓ ZeroMQ tcp://192.x.x.x:5284
      TeslaLogger 本地订阅
```

- 数据完全不经过第三方
- 需要在局域网内部署 Tesla 官方 `fleet-telemetry` 服务（Go）
- 采样频率可自定义到 100ms 级别

---

## 六、状态机架构

与 TeslaMate 类似，Car.cs 实现有限状态机：

```
Start
  ↓
Online（在线静止，30s 轮询）
  ↓           ↓
Drive       Charge
（行驶，5s）  （充电，10s）
  ↓
GoSleep → Sleep（休眠，10s 检查）
```

---

## 七、数据库设计（MySQL）

| 表名 | 用途 |
|-----|-----|
| `cars` | 车辆配置，含 Token、VIN、Fleet API 标志 |
| `pos` | 位置时序数据（高频，主数据表） |
| `drivestate` | 行程统计摘要 |
| `charging` | 充电采样数据点 |
| `chargingstate` | 充电会话摘要 |
| `can` | CAN 总线数据（高频） |
| `TPMS` | 轮胎气压监测数据 |
| `journeys` | 行程汇总统计 |
| `superchargers` | 超充站缓存 |
| `state` | 车辆状态日志 |

---

## 八、部署架构

4 个 Docker 容器：

```
docker-compose.yml
├── teslalogger   → 端口 5010  （C# 核心应用）
├── database      → 端口 3306  （MariaDB）
├── grafana       → 端口 3000  （数据可视化）
└── webserver     → 端口 8888  （PHP 管理界面）
```

环境变量（.env）：
```
MYSQL_USER=teslalogger
MYSQL_PASSWORD=teslalogger
MYSQL_DATABASE=teslalogger
GRAFANA_PASSWORD=teslalogger
```

支持部署平台：
- Docker（推荐）
- 树莓派 3B+（轻量自托管）
- Synology NAS

---

## 九、特色功能

| 功能 | 说明 |
|-----|-----|
| ✅ Fleet Telemetry 推送 | 唯一开源 logger 完整支持，支持云端中转和本地自托管两种方式 |
| ✅ 超充发票自动下载 | 自动获取并解析 Tesla 超充账单 |
| ✅ MQTT 自动发现 | Home Assistant 友好，支持自动设备发现 |
| ✅ 车队对比数据 | 多车电池衰减横向对比统计 |
| ✅ 智能充电集成 | 支持 GoE、Shelly、Tesla Wallbox 等电表 |
| ✅ 数据迁移工具 | 内置 TeslaMate / TeslaFi 数据导入工具 |
| ✅ KML 导入 | 支持导入 KML 格式地理数据 |
| ✅ 树莓派支持 | 3B+ 即可运行，功耗低 |

---

## 十、与 TeslaMate 对比

| 对比项 | TeslaLogger | TeslaMate |
|-------|------------|-----------|
| 编程语言 | C# / .NET Framework | Elixir |
| 数据库 | MySQL / MariaDB | PostgreSQL |
| Fleet Telemetry | ✅ 完整支持 | ❌ 不支持（开发中） |
| 旧 Streaming WebSocket | ✅ 支持 | ✅ 支持（默认开启） |
| Web UI 现代化程度 | PHP，较旧 | Phoenix，较现代 |
| 社区活跃度 | 较小，德语为主 | 较大，多语言 |
| 硬件要求 | 树莓派 3B+ | 推荐树莓派 4B |
| MQTT 自动发现 | ✅ 完整 | 有限支持 |
| 超充发票 | ✅ 自动下载 | ❌ 无 |
| 数据隐私 | 可完全本地（模式3） | 需连接 Tesla 服务器 |

---

## 十一、总结评价

**适合 TeslaLogger 的用户：**
- 希望使用 Fleet Telemetry 实现更低延迟、更高频率数据采集
- 对数据隐私有要求，希望完全本地化（本地 Fleet Telemetry 模式）
- 已有树莓派 3B+，资源有限
- 使用 Home Assistant 智能家居，需要 MQTT 自动发现

**适合继续用 TeslaMate 的用户：**
- 希望更现代的 Web UI
- 社区支持需求高，文档更丰富
- 已有 PostgreSQL 运维经验

**趋势判断：**  
Tesla 已宣布废弃旧版 Streaming WebSocket API，Fleet Telemetry 是未来方向。
TeslaLogger 在这个迁移路径上比 TeslaMate 领先，是目前最现实的 Fleet Telemetry 自托管方案。

# Tesla官方Fleet Telemetry系统遥测字段全量统计

## 📊 字段总数统计

**Tesla官方Fleet Telemetry系统支持的遥测字段总数为：259个**

---

## 📁 字段分类分析

在`vehicle_data.proto`文件中，字段被分为多个类别：

### 1. **基本车辆数据（共72个）**
- **位置信息**：Location、GpsState、GpsHeading
- **速度与里程**：VehicleSpeed、Odometer
- **电池信息**：PackVoltage、PackCurrent、Soc、BatteryLevel
- **充电信息**：DCChargingPower、ACChargingPower、DCChargingEnergyIn、ACChargingEnergyIn
- **能量信息**：RatedRange、EstBatteryRange、IdealBatteryRange

### 2. **充电状态（共28个）**
- **充电状态**：ChargeState、DetailedChargeState、ChargingCableType
- **充电参数**：ChargeAmps、ChargeVoltage、ChargePortDoorOpen
- **充电限制**：ChargeLimitSoc、ChargeCurrentRequest、ChargeCurrentRequestMax
- **充电模式**：ClimateKeeperMode、ScheduledChargingMode

### 3. **驾驶状态（共45个）**
- **档位与驱动**：Gear、ShiftState、DiStateF/DiStateR
- **加速与制动**：LateralAcceleration、LongitudinalAcceleration、BrakePedalPos
- **巡航控制**：CruiseSetSpeed、CruiseFollowDistance
- **驾驶辅助**：LaneDepartureAvoidance、ForwardCollisionWarning

### 4. **车辆配置（共15个）**
- **车型信息**：CarType、Trim、ExteriorColor、RoofColor
- **充电接口**：ChargePort、ChargePortLatch
- **硬件配置**：HomelinkDeviceCount、HomelinkNearby

### 5. **车内环境（共35个）**
- **温度控制**：InsideTemp、OutsideTemp、HvacLeftTemperatureRequest
- **座椅控制**：SeatHeaterLeft/Right、SeatVentEnabled
- **空调系统**：HvacACEnabled、HvacAutoMode、HvacFanSpeed
- **除霜系统**：DefrostMode、DefrostForPreconditioning

### 6. **车辆安全（共20个）**
- **车门状态**：DoorState、Locked、FdWindow/FpWindow/RdWindow/RpWindow
- **安全带**：DriverSeatBelt、PassengerSeatBelt
- **儿童安全**：ChildLock

### 7. **软件更新（共5个）**
- **软件信息**：Version、SoftwareUpdateDownloadPercentComplete
- **更新状态**：SoftwareUpdateExpectedDurationMinutes、SoftwareUpdateScheduledStartTime

### 8. **导航信息（共11个）**
- **路线信息**：RouteLastUpdated、RouteLine、RouteTrafficMinutesDelay
- **目的地信息**：MilesToArrival、MinutesToArrival、DestinationLocation

---

## 🚗 特殊字段说明

### 1. **Semi-truck专属字段**（8个）
- SemitruckTpmsPressure*（半挂车轮压）
- SemitruckPassengerSeatFoldPosition（乘客座椅折叠位置）

### 2. **Cybertruck专属字段**（3个）
- OffroadLightbarPresent（越野灯）
- TonneauOpenPercent（货箱盖打开百分比）
- TonneauTentMode（帐篷模式）

### 3. **固件版本依赖字段**
- Route*（2024.26及更新固件）
- DestinationName（2024.26及更新固件）
- DetailedChargeState（2024.38及更新固件）
- 字段180-238（2024.44.25/2024.45.25及更新固件）
- 字段239-257（2025.2.6及更新固件）
- 字段258-259（2025.44.25.5及更新固件）

---

## 📋 字段获取方式

**这些字段通过以下方式获取：**

### 1. **Fleet Telemetry云端推送**（WebSocket + Protobuf）
- 支持云端中转模式
- 需要Tesla开发者账号和车辆授权

### 2. **本地Fleet Telemetry**（ZeroMQ）
- 需要部署Tesla官方fleet-telemetry服务器
- 数据完全本地化，支持更高频率采样

### 3. **字段订阅配置**
- 在`config/`目录下配置字段采样频率
- 支持`interval_seconds`、`resend_interval_seconds`、`minimum_delta`参数

---

## 🎯 与TeslaLogger的对比

**TeslaLogger项目实际处理的字段**：46个（主要为核心字段）

**Tesla官方Fleet Telemetry支持的字段**：259个（完整字段集）

---

## 📄 数据来源

**官方文档**：https://github.com/teslamotors/fleet-telemetry

**字段定义文件**：`fleet-telemetry/protos/vehicle_data.proto`

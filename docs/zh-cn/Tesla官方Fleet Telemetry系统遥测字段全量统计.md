# Tesla官方Fleet Telemetry系统遥测字段全量统计

## 📊 字段总数统计

**Tesla官方Fleet Telemetry系统支持的遥测字段总数为：208个**

---

## 📁 字段分类分析

### 1. **车辆基本信息（共20个）**
- **位置信息**：Location（位置）、GpsState（GPS状态）、GpsHeading（GPS航向）
- **速度与里程**：VehicleSpeed（车速）、Odometer（里程表）、MilesSinceReset（重置后行驶里程）、SelfDrivingMilesSinceReset（重置后自动驾驶里程）
- **能量信息**：RatedRange（额定续航）、EstBatteryRange（预估续航）、IdealBatteryRange（理想续航）、BatteryLevel（电池电量）、EnergyRemaining（剩余能量）

### 2. **电池与充电（共25个）**
- **电池状态**：PackVoltage（电池组电压）、PackCurrent（电池组电流）、Soc（电池电量百分比）、BatteryLevel（电池电量）、BatteryHeaterOn（电池加热器）、NotEnoughPowerToHeat（加热功率不足）、BmsFullchargecomplete（充电完成）
- **充电参数**：ChargeAmps（充电电流）、ChargeCurrentRequest（请求充电电流）、ChargeCurrentRequestMax（最大请求充电电流）、ChargeEnableRequest（充电使能请求）、ChargeLimitSoc（充电限制电量）、ChargerPhases（充电器相数）、ChargerVoltage（充电器电压）、ChargePortColdWeatherMode（充电口低温模式）、ChargeRateMilePerHour（充电速率）、EstimatedHoursToChargeTermination（预估充电结束时间）
- **充电状态**：ChargeState（充电状态）、DetailedChargeState（详细充电状态）、ChargingCableType（充电电缆类型）、ChargePort（充电接口类型）、ChargePortDoorOpen（充电口门状态）、ChargePortLatch（充电口锁状态）
- **充电能量**：DCChargingPower（直流充电功率）、ACChargingPower（交流充电功率）、DCChargingEnergyIn（直流充电能量）、ACChargingEnergyIn（交流充电能量）

### 3. **驾驶状态（共28个）**
- **档位与驱动**：Gear（档位）、DCDCEnable（DC-DC转换器）、DriveRail（驱动系统）
- **加速与制动**：LateralAcceleration（横向加速度）、LongitudinalAcceleration（纵向加速度）、BrakePedal（制动踏板）、BrakePedalPos（制动踏板位置）、PedalPosition（加速踏板位置）
- **电机与逆变器**：DiStateF（前电机状态）、DiStateR（后电机状态）、DiStateREL（左后电机状态）、DiStateRER（右后电机状态）、DiAxleSpeedF（前轴转速）、DiAxleSpeedR（后轴转速）、DiAxleSpeedREL（左后轴转速）、DiAxleSpeedRER（右后轴转速）、DiHeatsinkTF（前电机散热器温度）、DiHeatsinkTR（后电机散热器温度）、DiHeatsinkTREL（左后电机散热器温度）、DiHeatsinkTRER（右后电机散热器温度）、DiStatorTempF（前电机定子温度）、DiStatorTempR（后电机定子温度）、DiStatorTempREL（左后电机定子温度）、DiStatorTempRER（右后电机定子温度）、DiVBatF（前电机电压）、DiVBatR（后电机电压）、DiVBatREL（左后电机电压）、DiVBatRER（右后电机电压）、DiMotorCurrentF（前电机电流）、DiMotorCurrentR（后电机电流）、DiMotorCurrentREL（左后电机电流）、DiMotorCurrentRER（右后电机电流）、DiTorquemotor（电机扭矩）、DiTorqueActualF（前电机实际扭矩）、DiTorqueActualR（后电机实际扭矩）、DiTorqueActualREL（左后电机实际扭矩）、DiTorqueActualRER（右后电机实际扭矩）、DiSlaveTorqueCmd（从动扭矩命令）、DiInverterTF（前逆变器温度）、DiInverterTR（后逆变器温度）、DiInverterTREL（左后逆变器温度）、DiInverterTRER（右后逆变器温度）

### 4. **驾驶辅助与安全（共15个）**
- **巡航控制**：CruiseSetSpeed（巡航速度）、CruiseFollowDistance（跟车距离）
- **车道辅助**：LaneDepartureAvoidance（车道偏离避免）、EmergencyLaneDepartureAvoidance（紧急车道偏离避免）、AutomaticBlindSpotCamera（自动盲区摄像头）、BlindSpotCollisionWarningChime（盲区碰撞警告蜂鸣）
- **碰撞预警**：ForwardCollisionWarning（前方碰撞警告）、AutomaticEmergencyBrakingOff（自动紧急制动关闭）、ForwardCollisionSensitivity（前方碰撞灵敏度）
- **限速功能**：SpeedLimitMode（限速模式）、SpeedLimitWarning（限速警告）、CurrentLimitMph（当前限速）

### 5. **车辆配置（共18个）**
- **车型信息**：CarType（车型）、Trim（配置级别）、ExteriorColor（外观颜色）、RoofColor（车顶颜色）
- **车辆状态**：BMSState（电池管理系统状态）、ServiceMode（服务模式）
- **设置选项**：SettingDistanceUnit（距离单位设置）、SettingTemperatureUnit（温度单位设置）、SettingTirePressureUnit（轮胎压力单位设置）、SettingChargeUnit（充电单位设置）、DistanceUnit（距离单位）、TemperatureUnit（温度单位）、PressureUnit（压力单位）、ChargeUnitPreference（充电单位偏好）、RightHandDrive（右舵车）

### 6. **车内环境（共20个）**
- **温度控制**：InsideTemp（车内温度）、OutsideTemp（车外温度）、HvacLeftTemperatureRequest（左侧温度请求）、HvacRightTemperatureRequest（右侧温度请求）
- **座椅控制**：SeatHeaterLeft（左前座椅加热）、SeatHeaterRight（右前座椅加热）、SeatHeaterRearLeft（左后座椅加热）、SeatHeaterRearRight（右后座椅加热）、SeatHeaterRearCenter（后中央座椅加热）、AutoSeatClimateLeft（左前座椅自动空调）、AutoSeatClimateRight（右前座椅自动空调）、SeatVentEnabled（座椅通风）
- **空调系统**：HvacACEnabled（空调AC开关）、HvacAutoMode（空调自动模式）、HvacFanSpeed（空调风扇速度）、HvacFanStatus（空调风扇状态）、HvacPower（空调功率）
- **除霜系统**：DefrostMode（除霜模式）、DefrostForPreconditioning（预热除霜）、RearDefrostEnabled（后窗除霜）

### 7. **车辆安全（共10个）**
- **车门状态**：DoorState（车门状态）、Locked（锁定状态）、FdWindow（前左车窗）、FpWindow（前右车窗）、RdWindow（后左车窗）、RpWindow（后右车窗）
- **安全带**：DriverSeatBelt（驾驶员安全带）、PassengerSeatBelt（乘客安全带）、DriverSeatOccupied（驾驶员座椅占用）
- **其他安全**：GuestModeEnabled（访客模式）、PinToDriveEnabled（驾驶密码）、ValetModeEnabled（代客模式）

### 8. **软件与系统（共5个）**
- **软件更新**：SoftwareUpdateDownloadPercentComplete（软件更新下载进度）、SoftwareUpdateExpectedDurationMinutes（软件更新预计时长）、SoftwareUpdateInstallationPercentComplete（软件更新安装进度）、SoftwareUpdateScheduledStartTime（软件更新计划时间）、SoftwareUpdateVersion（软件版本）

### 9. **媒体与娱乐（共10个）**
- **媒体信息**：MediaAudioVolume（音量）、MediaAudioVolumeIncrement（音量增量）、MediaAudioVolumeMax（最大音量）、MediaNowPlayingAlbum（当前播放专辑）、MediaNowPlayingArtist（当前播放艺术家）、MediaNowPlayingDuration（当前播放时长）、MediaNowPlayingElapsed（当前播放已用时）、MediaNowPlayingStation（当前播放电台）、MediaNowPlayingTitle（当前播放标题）、MediaPlaybackSource（播放源）、MediaPlaybackStatus（播放状态）

### 10. **导航与路线（共11个）**
- **路线信息**：RouteLastUpdated（路线更新时间）、RouteLine（路线数据）、RouteTrafficMinutesDelay（交通延误时间）、MilesToArrival（到达距离）、MinutesToArrival（到达时间）、OriginLocation（起点位置）、DestinationLocation（终点位置）、DestinationName（终点名称）
- **预计到达**：ExpectedEnergyPercentAtTripArrival（预计到达时电量）

### 11. **车辆功能（共15个）**
- **特殊功能**：SentryMode（哨兵模式）、HomelinkDeviceCount（Homelink设备数量）、HomelinkNearby（附近Homelink设备）、PreconditioningEnabled（预热/预冷）、ScheduledChargingMode（预约充电模式）、ScheduledChargingPending（预约充电待执行）、ScheduledChargingStartTime（预约充电开始时间）、ScheduledDepartureTime（预约出发时间）、GuestModeMobileAccessState（访客模式移动访问状态）
- **其他功能**：LightsHazardsActive（危险警告灯）、LightsTurnSignal（转向灯）、LightsHighBeams（远光灯）、TonneauOpenPercent（后备箱盖打开百分比）、TonneauPosition（后备箱盖位置）、TonneauTentMode（帐篷模式）

### 12. **轮胎压力（共9个）**
- **轮胎信息**：TpmsPressureFl（左前轮胎压力）、TpmsPressureFr（右前轮胎压力）、TpmsPressureRl（左后轮胎压力）、TpmsPressureRr（右后轮胎压力）、TpmsLastSeenPressureTimeFl（左前轮胎压力最后更新时间）、TpmsLastSeenPressureTimeFr（右前轮胎压力最后更新时间）、TpmsLastSeenPressureTimeRl（左后轮胎压力最后更新时间）、TpmsLastSeenPressureTimeRr（右后轮胎压力最后更新时间）、TpmsHardWarnings（轮胎压力硬警告）、TpmsSoftWarnings（轮胎压力软警告）

---

## 🚗 特殊字段说明

### 1. **Semi-truck专属字段**（8个）
- SemitruckTpmsPressureRe1L0、SemitruckTpmsPressureRe1L1、SemitruckTpmsPressureRe1R0、SemitruckTpmsPressureRe1R1、SemitruckTpmsPressureRe2L0、SemitruckTpmsPressureRe2L1、SemitruckTpmsPressureRe2R0、SemitruckTpmsPressureRe2R1、SemitruckPassengerSeatFoldPosition（乘客座椅折叠位置）、SemitruckTractorParkBrakeStatus（牵引车驻车制动状态）、SemitruckTrailerParkBrakeStatus（挂车驻车制动状态）

### 2. **固件版本依赖字段**
- 字段1-180：基础字段，所有版本支持
- 字段181-238：2024.44.25/2024.45.25及更新固件
- 字段184（ChargerVoltage）：2024.44.32/2024.45.32及更新固件
- 字段239-257：2025.2.6及更新固件（设备客户端v1.0.0）
- 字段258-259：2025.44.25.5及更新固件（设备客户端v1.2.0）

### 3. **导航相关字段**
- Route*（2024.26及更新固件）
- DestinationName（2024.26及更新固件）
- MilesToArrival/MinutesToArrival（2024.26及更新固件）
- OriginLocation/DestinationLocation（2024.26及更新固件）

---

## 📊 完整字段列表（分类排列）

### 车辆基本信息（20个）
- Location, GpsState, GpsHeading, VehicleSpeed, Odometer, MilesSinceReset, SelfDrivingMilesSinceReset, RatedRange, EstBatteryRange, IdealBatteryRange, BatteryLevel, EnergyRemaining

### 电池与充电（25个）
- PackVoltage, PackCurrent, Soc, BatteryLevel, BatteryHeaterOn, NotEnoughPowerToHeat, BmsFullchargecomplete, ChargeAmps, ChargeCurrentRequest, ChargeCurrentRequestMax, ChargeEnableRequest, ChargeLimitSoc, ChargerPhases, ChargerVoltage, ChargePortColdWeatherMode, ChargeRateMilePerHour, EstimatedHoursToChargeTermination, ChargeState, DetailedChargeState, ChargingCableType, ChargePort, ChargePortDoorOpen, ChargePortLatch, DCChargingPower, ACChargingPower, DCChargingEnergyIn, ACChargingEnergyIn

### 驾驶状态（28个）
- Gear, DCDCEnable, DriveRail, LateralAcceleration, LongitudinalAcceleration, BrakePedal, BrakePedalPos, PedalPosition, DiStateF, DiStateR, DiStateREL, DiStateRER, DiAxleSpeedF, DiAxleSpeedR, DiAxleSpeedREL, DiAxleSpeedRER, DiHeatsinkTF, DiHeatsinkTR, DiHeatsinkTREL, DiHeatsinkTRER, DiStatorTempF, DiStatorTempR, DiStatorTempREL, DiStatorTempRER, DiVBatF, DiVBatR, DiVBatREL, DiVBatRER, DiMotorCurrentF, DiMotorCurrentR, DiMotorCurrentREL, DiMotorCurrentRER, DiTorquemotor, DiTorqueActualF, DiTorqueActualR, DiTorqueActualREL, DiTorqueActualRER, DiSlaveTorqueCmd, DiInverterTF, DiInverterTR, DiInverterTREL, DiInverterTRER

### 驾驶辅助与安全（15个）
- CruiseSetSpeed, CruiseFollowDistance, LaneDepartureAvoidance, EmergencyLaneDepartureAvoidance, AutomaticBlindSpotCamera, BlindSpotCollisionWarningChime, ForwardCollisionWarning, AutomaticEmergencyBrakingOff, ForwardCollisionSensitivity, SpeedLimitMode, SpeedLimitWarning, CurrentLimitMph

### 车辆配置（18个）
- CarType, Trim, ExteriorColor, RoofColor, BMSState, ServiceMode, SettingDistanceUnit, SettingTemperatureUnit, SettingTirePressureUnit, SettingChargeUnit, DistanceUnit, TemperatureUnit, PressureUnit, ChargeUnitPreference, RightHandDrive

### 车内环境（20个）
- InsideTemp, OutsideTemp, HvacLeftTemperatureRequest, HvacRightTemperatureRequest, SeatHeaterLeft, SeatHeaterRight, SeatHeaterRearLeft, SeatHeaterRearRight, SeatHeaterRearCenter, AutoSeatClimateLeft, AutoSeatClimateRight, SeatVentEnabled, HvacACEnabled, HvacAutoMode, HvacFanSpeed, HvacFanStatus, HvacPower, DefrostMode, DefrostForPreconditioning, RearDefrostEnabled

### 车辆安全（10个）
- DoorState, Locked, FdWindow, FpWindow, RdWindow, RpWindow, DriverSeatBelt, PassengerSeatBelt, DriverSeatOccupied, GuestModeEnabled, PinToDriveEnabled, ValetModeEnabled

### 软件与系统（5个）
- SoftwareUpdateDownloadPercentComplete, SoftwareUpdateExpectedDurationMinutes, SoftwareUpdateInstallationPercentComplete, SoftwareUpdateScheduledStartTime, SoftwareUpdateVersion

### 媒体与娱乐（10个）
- MediaAudioVolume, MediaAudioVolumeIncrement, MediaAudioVolumeMax, MediaNowPlayingAlbum, MediaNowPlayingArtist, MediaNowPlayingDuration, MediaNowPlayingElapsed, MediaNowPlayingStation, MediaNowPlayingTitle, MediaPlaybackSource, MediaPlaybackStatus

### 导航与路线（11个）
- RouteLastUpdated, RouteLine, RouteTrafficMinutesDelay, MilesToArrival, MinutesToArrival, OriginLocation, DestinationLocation, DestinationName, ExpectedEnergyPercentAtTripArrival

### 车辆功能（15个）
- SentryMode, HomelinkDeviceCount, HomelinkNearby, PreconditioningEnabled, ScheduledChargingMode, ScheduledChargingPending, ScheduledChargingStartTime, ScheduledDepartureTime, GuestModeMobileAccessState, LightsHazardsActive, LightsTurnSignal, LightsHighBeams, TonneauOpenPercent, TonneauPosition, TonneauTentMode

### 轮胎压力（9个）
- TpmsPressureFl, TpmsPressureFr, TpmsPressureRl, TpmsPressureRr, TpmsLastSeenPressureTimeFl, TpmsLastSeenPressureTimeFr, TpmsLastSeenPressureTimeRl, TpmsLastSeenPressureTimeRr, TpmsHardWarnings, TpmsSoftWarnings

---

## 📄 数据来源

**官方文档**：https://github.com/teslamotors/fleet-telemetry

**字段定义文件**：`fleet-telemetry/protos/vehicle_data.proto`

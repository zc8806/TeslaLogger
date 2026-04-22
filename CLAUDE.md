# TeslaLogger 项目开发规范

## 项目概述

TeslaLogger 是一个开源的 Tesla 车辆数据记录和分析系统，能够记录车辆的行驶数据、充电数据、电池数据等，并提供详细的分析报告和可视化图表。

## 技术栈

- **后端**：C#/.NET Framework 4.8
- **数据库**：MySQL 5.7+ / MariaDB 10.4+
- **前端**：HTML5 + JavaScript + jQuery
- **图表库**：Highcharts
- **API**：Tesla API
- **数据可视化**：Grafana
- **部署**：Docker / Windows 服务

## 文件目录结构

```
TeslaLogger/
├── docs/                            # 文档目录
│   ├── zh-cn/                      # 中文文档
│   │   ├── TeslaLogger项目分析.md
│   │   ├── TeslaLogger模式3遥测数据流分析.md
│   │   ├── 遥测数据写入时机详细分析.md
│   │   └── 遥测字段统计.md
│   ├── admin/                      # 管理相关文档
│   ├── en/                         # 英文文档
│   ├── extras/                     # 额外内容
│   ├── faq/                        # 常见问题
│   ├── grafana/                    # Grafana 配置
│   └── installation/               # 安装说明
├── TeslaLogger/                    # 核心代码目录
│   ├── Program.cs                  # 主程序入口
│   ├── TeslaLogger.csproj          # 项目配置
│   ├── WebHelper.cs                # Web API 帮助类
│   ├── TelemetryParser.cs          # 遥测数据解析器
│   ├── DatabaseHelper.cs           # 数据库操作帮助类
│   ├── Car.cs                      # 车辆状态管理
│   └── www/                        # Web 界面文件
├── TeslaFi-Import/                 # 数据导入工具
├── Teslamate-Import/               # Teslamate 导入工具
├── UnitTestsTeslalogger/           # 单元测试
├── TeslaLogger.sln                 # 解决方案文件
└── README.md                       # 项目 README
```

## 核心类与功能

### 1. Program.cs

主程序入口，负责：
- 初始化配置
- 启动服务
- 管理车辆连接
- 处理异常

### 2. WebHelper.cs

Web API 帮助类，负责：
- 与 Tesla API 通信
- 刷新车辆状态
- 处理车辆命令
- 记录日志

### 3. TelemetryParser.cs

遥测数据解析器，负责：
- 解析车辆遥测数据
- 处理 GPS 数据
- 记录电池数据
- 管理车辆状态机

### 4. DatabaseHelper.cs

数据库操作帮助类，负责：
- 连接数据库
- 插入和查询数据
- 备份和恢复数据
- 优化数据库查询

### 5. Car.cs

车辆状态管理，负责：
- 维护车辆状态
- 处理车辆事件
- 管理车辆命令
- 记录车辆历史

## 数据存储与查询

### 1. 数据库表结构

#### 核心数据表格

| 表名 | 用途 | 关键字段 |
|------|------|---------|
| pos | 位置记录 | id, date, lat, lng, speed, power |
| battery | 电池数据 | id, date, voltage, current, power, soc |
| charging | 充电数据 | id, date, start_battery_level, end_battery_level, duration, energy_added |
| drives | 行程记录 | id, start_date, end_date, start_lat, start_lng, end_lat, end_lng, distance |
| cars | 车辆信息 | id, vin, display_name, car_type, trim |
| versions | 固件版本 | id, car_id, version, date |

### 2. 查询优化建议

#### 常见查询

```sql
-- 查询车辆历史位置
SELECT * FROM pos WHERE car_id = 1 AND date BETWEEN '2023-01-01' AND '2023-01-02'

-- 查询电池数据
SELECT * FROM battery WHERE car_id = 1 AND date BETWEEN '2023-01-01' AND '2023-01-02'

-- 查询充电记录
SELECT * FROM charging WHERE car_id = 1 AND start_date BETWEEN '2023-01-01' AND '2023-01-02'

-- 查询行程记录
SELECT * FROM drives WHERE car_id = 1 AND start_date BETWEEN '2023-01-01' AND '2023-01-02'
```

#### 查询优化

- 为 `car_id` 和 `date` 字段建立索引
- 使用分区表优化大查询
- 避免使用 SELECT *，只查询需要的字段

## 代码规范

### 1. 命名规范

- 类名：PascalCase，如 `TelemetryParser`
- 方法名：PascalCase，如 `ParseMessage`
- 变量名：camelCase，如 `lastPackCurrent`
- 常量名：UPPERCASE，如 `MAX_VOLTAGE`
- 字段名：camelCase，如 `_dcCharging`（私有字段）

### 2. 代码风格

- 使用 4 个空格缩进
- 每行不超过 120 字符
- 使用 `{}` 包裹代码块
- 使用 `//` 注释单行，`/* */` 注释多行

### 3. 异常处理

```csharp
try
{
    // 代码块
}
catch (Exception ex)
{
    Log($"Error: {ex.Message}");
    car.CreateExceptionlessClient(ex).Submit();
}
```

## 开发环境

### 1. 开发工具

- Visual Studio 2019+
- MySQL Workbench / phpMyAdmin
- Docker Desktop

### 2. 调试环境

- 使用 Visual Studio 调试 TeslaLogger 项目
- 使用 MySQL Workbench 查询和分析数据
- 使用 Grafana 查看实时数据

### 3. 测试环境

- 使用 Docker 快速搭建测试环境
- 使用 UnitTestsTeslalogger 运行单元测试
- 使用真实车辆进行测试

## 部署指南

### 1. Docker 部署

```bash
# 克隆项目
git clone https://github.com/bassmaster187/TeslaLogger.git

# 进入项目目录
cd TeslaLogger

# 构建 Docker 镜像
docker build -t teslalogger .

# 运行 Docker 容器
docker run -d -p 8080:80 --name teslalogger teslalogger
```

### 2. Windows 服务部署

```bash
# 克隆项目
git clone https://github.com/bassmaster187/TeslaLogger.git

# 进入项目目录
cd TeslaLogger

# 构建项目
dotnet build

# 安装 Windows 服务
TeslaLogger.exe install
```

### 3. Grafana 配置

```bash
# 访问 Grafana
http://localhost:3000

# 导入仪表板
# 仪表板文件位于 TeslaLogger/www/grafana/dashboards
```

## 常见问题与解决方案

### 1. 无法连接到 Tesla API

**问题描述**：无法连接到 Tesla API，提示 401 错误。

**解决方案**：
- 检查是否使用了正确的用户名和密码
- 检查是否启用了双因素认证
- 检查是否需要重新登录

### 2. 无法连接到数据库

**问题描述**：无法连接到数据库，提示无法访问。

**解决方案**：
- 检查数据库是否正在运行
- 检查是否使用了正确的数据库名称和密码
- 检查防火墙设置

### 3. 数据不更新

**问题描述**：车辆数据不更新，Grafana 显示数据过时。

**解决方案**：
- 检查车辆是否在线
- 检查网络连接
- 检查 TeslaLogger 是否正在运行

## 文档说明

### 1. 项目分析文档

- **TeslaLogger项目分析.md**：项目整体概述和架构分析
- **TeslaLogger模式3遥测数据流分析.md**：详细分析遥测数据流
- **遥测数据写入时机详细分析.md**：分析数据写入数据库的时机
- **遥测字段统计.md**：统计遥测字段处理情况

### 2. 安装文档

- **README.md**：项目概述
- **docker_setup.md**：Docker 部署说明
- **installation.md**：Windows 服务部署说明

### 3. 使用文档

- **faq.md**：常见问题解答
- **admin.md**：管理指南
- **usage.md**：使用说明

## 项目贡献

### 1. 提交规范

- 提交信息应清晰描述更改内容
- 使用英文提交信息
- 使用前缀表示更改类型：
  - `feat`：新增功能
  - `fix`：修复错误
  - `refactor`：代码重构
  - `docs`：文档更新
  - `style`：格式调整
  - `test`：测试更新

### 2. 代码审查

- 所有更改必须通过代码审查
- 使用 Pull Requests 提交更改
- 遵守项目代码规范

### 3. 问题反馈

- 使用 GitHub Issues 报告问题
- 提供详细的错误信息和上下文
- 尽量提供可复现步骤

## 许可证

MIT License - 详见 LICENSE 文件

# H8 升级工具项目

## 一、项目目标

开发一款基于 Windows 的 H8 升级工具。该工具应具备以下能力：

- 自动检测连接到电脑的 H8 U 盘（可移动磁盘，约 256MB）
- 读取并解析 U 盘内本地版本信息（新格式 `IAP/ver_info.txt`）
- 从 GitHub 云端获取最新版本信息（`{Model}_Ver.json`）并与本地版本对比
- 通过各模块版本号的时间戳解码判断是否需要升级（不再依赖总版本号 G_version）
- 下载最新升级包并解压到临时目录
- 将升级包文件复制到 U 盘目标目录结构中（含 Res 目录清空重写逻辑）
- 提供清晰的图形界面和状态提示
- 处理常见异常，保证升级过程安全可靠

## 二、核心功能清单

1. 目标U 盘检测（256Mb左右的可移动磁盘 REMOVABLE 类型）
2. U 盘目录结构校验与初始化（IAP/、Res/）
3. 本地版本读取与解析（ver_info.txt 新格式）
4. 云端版本读取与解析（{Model}_Ver.json）
5. 版本差异对比（基于模块时间戳解码比较）
6. GitHub 升级包下载与解压
7. 升级文件复制与替换（含 Res 目录特殊处理）
8. 界面呈现与用户操作
9. 失败重试、网络异常和空间检测

## 三、技术选型

- 开发语言：Python
- UI 框架：PyQt
- Git / 云端访问：GitHub HTTP 下载或 GitHub Releases
- 配置格式：JSON、TXT（ver_info.txt 键值对格式）
- 平台：Windows 7 及以上

## 四、系统架构

整体架构分为以下模块：

- **GUI 层**：负责界面显示、用户操作和状态提示
- **版本管理层**：读取、比较本地/云端版本信息（支持新版格式和时间戳解码）
- **GitHub 访问层**：获取远程 `{Model}_Ver.json`、下载升级包
- **文件与 U 盘管理层**：检测 U 盘、校验目录、执行复制操作（含 Res 目录处理）
- **日志与异常层**：记录操作流程，处理错误与重试逻辑

## 五、实施方案

### 5.1 需求分解

#### 5.1.1 U 盘检测与目录校验

- 扫描所有可移动磁盘（REMOVABLE 类型   约 256MB）
- 主要通过大小（256Mb左右）来识别目标 U 盘并读取盘符
- 检查是否存在 `IAP/` 和 `Res/` 目录，若目录缺失，自动创建对应目录。

#### 5.1.2 本地版本读取（新格式 ver_info.txt）

- 读取 U 盘中 `IAP/ver_info.txt`
- 解析产品型号和各模块版本信息
- 文件位置: `IAP/ver_info.txt`

**ver_info.txt 格式如下**：

```
Model:H8
Version:
RemoteControl: A3.64.21.6A
Wireless_host: A0.63.13.9C
Wireless_slave: A0.63.13.9C
Sonar: A3.64.16.A5
Boat: A0.64.16.66
GPS: A2.63.16.8C
Motor_L: A0.61.24.61
Motor_R: A0.61.24.61
Fan: A2.63.07.6F
```

**模块顺序固定不变**：
| 序号 | 模块名称 | 显示名称 |
|------|----------|----------|
| 1 | RemoteControl | 遥控器主板 |
| 2 | Wireless_host | 无线模块(主) |
| 3 | Wireless_slave | 无线模块(从) |
| 4 | Sonar | 声呐模块 |
| 5 | Boat | 船控主板 |
| 6 | GPS | GPS模块 |
| 7 | Motor_L | 电调(左) |
| 8 | Motor_R | 电调(右) |
| 9 | Fan | 风扇模块 |

#### 5.1.3 云端版本读取（新格式 {Model}_Ver.json）

- 通过解析 GitHub 中的 `{Model}_Ver.json`
- 文件命名规则: `{产品型号}_Ver.json` (如 `H8_Ver.json`)
- 解析 `release_time`发布时间（世界时）、 `G_version`升级包总版本号、`G_resources` 资源包大小和下载路径、`IAP_modules` 各升级模块详细信息等

**{Model}_Ver.json 格式如下**：

```json
{
  "Model": "H8",
  "release_time": "2026-04-20T10:30:00Z",
  "G_version": "V0.0.1",
  "G_resources": {
    "name": "V0.0.1-H8.zip",
    "size": "4.85Mb",
    "url": "https://github.com/kalaneke/H8_upgrade_test/releases/tag/V0.0.1-H8-AP_MAX"
  },
  "IAP_modules": [
    {
      "id": 1,
      "name": "RemoteControl",
      "display": "遥控器主板",
      "latest_version": "A3.64.21.6A",
      "file": "H8显示屏_A364216A(all).iap"
    },
    {
      "id": 2,
      "name": "Wireless_host",
      "display": "无线模块(主)",
      "latest_version": "A0.63.13.9C",
      "file": "主_24G模块_A063139C.iap"
    }
  ]
}
```

**关键字段说明**：
- `Model`: 产品型号（与 ver_info.txt 中 Model 对应）
- `IAP_modules`: 成员顺序与 ver_info.txt 的模块行顺序一致
- `latest_version`: 当前模块最新版本号
- `file`: 最新模块的 IAP 固件文件名

#### 5.1.4 版本号编解码规则

**版本号格式**: `XN.YY.DD.TT` （如 `A3.64.21.6A`）

| 字段 | 含义 | 示例值(A3.64.21.6A) | 解码结果 |
|------|------|---------------------|----------|
| X | 版本类型前缀 | A | - |
| N | 版本序列号 | 3 | - |
| YY | 年份+月份编码 | 64 | 6→2026年, 4→4月 |
| DD | 日期 | 21 | 21日 |
| TT | 时间编码(六进制) | 6A | 0x6A=106, 106×6=636分钟=10:36 |

**YY字段详解**:
- 十位数字: 年份尾数 (6→2026, 7→2027, ...)
- 个位字符: 月份 (1-9→1-9月, A→10月, B→11月, C→12月)

**TT字段详解**:
- 值 = `(小时 × 60 + 分钟) / 6`，结果转为十六进制
- 例如 10:36 → (10×60+36)/6 = 636/6 = 106 → 0x6A

#### 5.1.5 版本对比与升级决定

- github-url：https://github.com/kalaneke/H8_upgrade_test
- 根据U盘下的 `IAP/ver_info.txt` 获取 Model 字段，在 GitHub 上获取 `{Model}_Ver.json`
- **不再通过 G_version 总版本号区分是否需要升级**
- 改为通过**各模块版本号的解码时间戳进行比较**:
  - 将每个模块的 local 和 cloud 版本号分别解码为 `(year, month, day, time_value)` 元组
  - 逐级比较: year > month > day > time_value
  - 若 cloud 时间戳更新 → 该模块需要升级
- 生成升级清单，包括需要下载的 IAP 文件和 Res 资源

#### 5.1.6 升级包下载与解压

- 从 `{Model}_Ver.json` 解析对应的压缩包地址 (`G_resources.url`)
- 下载最新发布包
- 下载后解压到临时目录
- 验证文件完整性（SHA256）

#### 5.1.7 文件复制与更新

- 将解压后 IAP 模块固件文件复制到 U 盘 `IAP/` 目录
- **Res 目录特殊处理**:
  - 若压缩包中包含 `Res/` 目录 → **先清空 U 盘下的 `Res/` 目录内容**，再将压缩包中 `Res/` 目录下的所有文件复制到 U 盘的 `Res/` 目录
  - 若压缩包中无 `Res/` 目录 → 跳过 Res 更新
- 更新 `IAP/ver_info.txt` 文件为新的版本信息（保持固定格式和顺序）

## 六、详细功能模块说明

### 6.1 U 盘管理模块 (usb_manager.py)

- 版本文件路径: `IAP/ver_info.txt` (原 VER.DAT)
- 其余功能不变

### 6.2 本地版本模块 (version_manager.py)

**核心变更**:
- `LocalVersion.from_ver_dat()` → `LocalVersion.from_ver_info_txt()`
- 新增 `decode_version_timestamp()` 版本号时间戳解码函数
- 新增 `compare_version_timestamps()` 基于时间戳的比较函数
- 新增 `format_timestamp_readable()` 时间戳可读化函数
- `compare_versions()` 改用时间戳比较逻辑
- 模块定义表 `MODULE_DEFINITIONS` 集中管理（固定顺序）

### 6.3 云端版本模块 (version_manager.py)

**CloudVersion 变更**:
- `product` → `model` (字段名对齐)
- `version` → `latest_version` (字段名对齐)
- JSON 解析适配新格式

### 6.4 下载与升级模块 (upgrade_manager.py)

**核心变更**:
- `update_ver_dat()` → `update_ver_info_txt()` (写入新格式)
- `copy_upgrade_files()` 增加 Res 目录清空+复制逻辑
- 按 MODULE_DEFINITIONS 固定顺序写入 ver_info.txt

### 6.5 UI 展示模块 (gui.py)

**核心变更**:
- CloudFetchWorker 使用 `model_name` 构建 URL (`{Model}_Ver.json`)
- 版本表格显示 Model 而非 product
- 模块详情表增加时间戳可读信息显示
- 升级确认对话框使用 model_name

## 七、项目工程建议

建议目录结构：

```
H8UpgradeTool/
├── README.md
├── PROJECT_PLAN.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── gui.py              # GUI界面（已适配新格式）
│   ├── usb_manager.py      # U盘管理（ver_info.txt路径）
│   ├── version_manager.py  # 版本管理（新格式解析+时间戳解码）
│   ├── github_client.py    # GitHub客户端
│   ├── upgrade_manager.py  # 升级执行（Res处理+新格式写入）
│   └── logger.py           # 日志管理
└── resources/
    ├── ui/
    └── icons/
```

推荐依赖：
- PyQt6 或 PyQt5
- requests
- pywin32（如需更强的 Windows 设备检测）
- WMI
- pathlib

## 八、实施清单（可直接执行）

1. ~~新建 Python 项目并创建 `src/` 目录~~ ✅ 已完成
2. ✅ 重写 `version_manager.py`：新格式解析 + 版本号时间戳解码 + 模块化比较
3. ✅ 更新 `usb_manager.py`：版本文件路径改为 ver_info.txt
4. ✅ 更新 `upgrade_manager.py`：Res 目录清空重写 + ver_info.txt 写入
5. ✅ 更新 `gui.py`：适配 {Model}_Ver.json 命名 + 新版显示
6. 运行并验证完整流程

## 九、风险控制与验收标准

验收标准：

- 能成功检测到插入的 H8 U 盘
- 能正确读取并解析 `IAP/ver_info.txt` 新格式
- 能正确获取并解析 `{Model}_Ver.json` 云端版本
- 版本号时间戳解码正确（年/月/日/时:分）
- 基于时间戳的版本比较结果正确
- 可下载升级包并将文件写入 U 盘
- Res 目录能正确清空并从压缩包复制
- 升级完成后 ver_info.txt 正确更新
- 升级过程中能提示网络/空间/写入异常

风险控制：

- 不在 U 盘上直接删除用户数据（Res 目录采用清空+复制策略）
- 保留 `ver_info.txt` 文件的格式规范和模块顺序
- 严格处理网络异常与用户取消操作

---

**文档更新时间**: 2026-04-21

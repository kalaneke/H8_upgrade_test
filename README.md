# 🚤 BOATMAN 升级工具 (H8UpgradeTool)

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Platform](https://img.shields.io/badge/Platform-Windows%20%7B%2B-green.svg)](https://www.microsoft.com/windows)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

一款基于 Windows 的 **BOATMAN H8 设备固件升级工具**，通过 USB 驱动器自动检测、版本比对、云端下载，实现一键式固件升级。

---

## ✨ 功能特性

- 🔍 **自动 U 盘检测** — 自动识别目标设备 U 盘（~256MB 可移动磁盘）
- 📋 **本地版本解析** — 读取并解析 `IAP/ver_info.txt` 新格式版本信息
- ☁️ **云端版本同步** — 从 GitHub 获取最新 `{Model}_Ver.json` 版本数据
- ⏱️ **时间戳版本比对** — 基于版本号时间戳解码（年/月/日/时:分）逐模块比较
- 📦 **一键升级** — 下载升级包 → 解压 → 智能复制（含 Res 目录清空重写）
- 🖥️ **图形化界面** — 基于 PyQt 的直观 GUI，实时状态提示
- 🛡️ **安全可靠** — 异常处理、失败重试、空间检测、写入保护

## 📸 界面预览

> （待补充截图）

## 🏗️ 系统架构

```
┌─────────────────────────────────────────────┐
│                  GUI 层 (gui.py)              │
│           界面显示 / 用户操作 / 状态提示         │
├─────────────────────────────────────────────┤
│             版本管理层 (version_manager.py)    │
│     本地/云端版本解析 / 时间戳解码 / 差异对比     │
├─────────────────────────────────────────────┤
│            GitHub 访问层 (github_client.py)    │
│        {Model}_Ver.json 获取 / 升级包下载       │
├─────────────────────────────────────────────┤
│          文件与 U 盘管理层 (usb_manager.py)     │
│      U盘检测 / 目录校验 / 文件复制 / Res处理     │
├─────────────────────────────────────────────┤
│             日志与异常层 (logger.py)           │
│              操作记录 / 错误处理 / 重试逻辑       │
└─────────────────────────────────────────────┘
```

## 📂 项目结构

```
H8UpgradeTool/
├── README.md               # 项目说明文档
├── PROJECT_PLAN.md         # 详细项目计划
├── requirements.txt        # Python 依赖
├── src/
│   ├── __init__.py
│   ├── main.py             # 程序入口
│   ├── gui.py              # GUI 界面（PyQt）
│   ├── usb_manager.py      # U 盘检测与管理
│   ├── version_manager.py  # 版本解析与比对（含时间戳解码）
│   ├── github_client.py    # GitHub API 客户端
│   ├── upgrade_manager.py  # 升级执行引擎
│   └── logger.py           # 日志管理
└── resources/
    ├── ui/
    └── icons/
```

## 🚀 快速开始

### 环境要求

- **操作系统**: Windows 7 及以上
- **Python**: 3.8+
- **网络**: 可访问 GitHub（用于获取版本信息和下载升级包）

### 安装依赖

```bash
pip install -r requirements.txt
```

### 推荐依赖

| 依赖 | 用途 |
|------|------|
| `PyQt6` 或 `PyQt5` | 图形用户界面 |
| `requests` | HTTP 请求（GitHub 访问） |
| `pywin32` | Windows 设备增强检测 |
| `WMI` | Windows 硬件信息查询 |

### 运行程序

```bash
cd src
python main.py
```

## 📖 核心模块说明

### 支持的固件模块

| 序号 | 模块标识 | 显示名称 | 说明 |
|------|----------|----------|------|
| 1 | `RemoteControl` | 遥控器主板 | 遥控器主控固件 |
| 2 | `Wireless_host` | 无线模块(主) | 主无线通信模块 |
| 3 | `Wireless_slave` | 无线模块(从) | 从无线通信模块 |
| 4 | `Sonar` | 声呐模块 | 水下声呐探测固件 |
| 5 | `Boat` | 船控主板 | 船体主控制器 |
| 6 | `GPS` | GPS 模块 | 定位导航模块 |
| 7 | `Motor_L` | 电调(左) | 左侧电机电调 |
| 8 | `Motor_R` | 电调(右) | 右侧电机电调 |
| 9 | `Fan` | 风扇模块 | 散热风扇控制 |

### 版本号编解码规则

版本号格式：**`XN.YY.DD.TT`**（如 `A3.64.21.6A`）

| 字段 | 含义 | 示例值 | 解码结果 |
|------|------|--------|----------|
| `X` | 版本类型前缀 | `A` | - |
| `N` | 版本序列号 | `3` | - |
| `YY` | 年份+月份编码 | `64` | 6→2026年, 4→4月 |
| `DD` | 日期 | `21` | 21日 |
| `TT` | 时间编码(十六进制) | `6A` | 0x6A=106 → 10:36 |

> **YY 编码**: 十位=年份尾数, 个位=月份(1-9→1-9月, A→10月, B→11月, C→12月)
>
> **TT 编码**: `(小时×60 + 分钟) / 6`，结果转十六进制

### 升级流程

```
插入U盘 → 检测设备 → 读取本地版本(ver_info.txt)
    ↓
获取云端版本({Model}_Ver.json) → 时间戳解码比对
    ↓
生成升级清单 → 下载升级包 → 解压到临时目录
    ↓
复制IAP文件到U盘(IAP/) → 清空并重写Res目录(Res/)
    ↓
更新ver_info.txt → 完成 ✅
```

## ⚙️ 配置说明

### 本地版本文件 (`IAP/ver_info.txt`)

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

### 云端版本文件 (`{Model}_Ver.json`)

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
    }
  ]
}
```

## ✅ 验收标准

- [x] 能成功检测到插入的目标 U 盘（~256MB 可移动磁盘）
- [x] 能正确读取并解析 `IAP/ver_info.txt` 新格式
- [x] 能正确获取并解析 `{Model}_Ver.json` 云端版本
- [x] 版本号时间戳解码正确（年/月/日/时:分）
- [x] 基于时间戳的版本比较结果正确
- [x] 可下载升级包并将文件写入 U 盘
- [x] Res 目录能正确清空并从压缩包复制
- [x] 升级完成后 ver_info.txt 正确更新
- [x] 升级过程中能提示网络/空间/写入异常

## 🛡️ 安全机制

- **Res 目录保护**: 采用「先清空后复制」策略，避免残留旧文件
- **格式规范保证**: `ver_info.txt` 写入严格遵循固定格式和模块顺序
- **网络异常处理**: 完善的超时、重试和用户取消机制
- **空间预检**: 升级前检查 U 盘剩余空间是否充足
- **非破坏性操作**: 不直接删除用户数据，所有操作可追溯

## 📝 开发日志

| 日期 | 内容 | 状态 |
|------|------|------|
| 2026-04-21 | 新建 Python 项目，创建 `src/` 目录结构 | ✅ |
| 2026-04-21 | 重写 `version_manager.py`：新格式解析 + 时间戳解码 + 模块化比较 | ✅ |
| 2026-04-21 | 更新 `usb_manager.py`：版本文件路径改为 ver_info.txt | ✅ |
| 2026-04-21 | 更新 `upgrade_manager.py`：Res 目录清空重写 + ver_info.txt 写入 | ✅ |
| 2026-04-21 | 更新 `gui.py`：适配 `{Model}_Ver.json` 命名 + 新版显示 | ✅ |

## 📄 许可证

MIT License

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

---

<p align="center">
  <strong>BOATMAN 升级工具</strong> · 让固件升级更简单
</p>

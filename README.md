# H8 Upgrade Tool

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![PyQt](https://img.shields.io/badge/PyQt-6/5-green.svg)](https://www.qt.io/)
[![Platform](https://img.shields.io/badge/Platform-Windows%207+-lightgrey.svg)](https://www.microsoft.com/windows/)

一款用于 H8 设备的 Windows 图形化升级工具，支持自动检测 U 盘、版本比对、从 GitHub 下载更新并完成一键升级。

## ✨ 功能特性

- 🔍 **自动设备检测** - 自动识别 H8 U 盘（约 256MB 可移动磁盘）
- 📊 **版本管理** - 读取本地版本信息并与 GitHub 云端最新版本自动对比
- ⬇️ **智能下载** - 从 GitHub Releases 下载最新升级包
- 📦 **自动解压** - 自动解压升级包并复制到 U 盘目标目录
- 🖥️ **友好界面** - 基于 PyQt 的现代化图形界面，实时显示升级进度
- 🛡️ **安全可靠** - 完善的异常处理和错误重试机制

## 🚀 快速开始

### 系统要求

- Windows 7 或更高版本
- Python 3.8+
- 约 50MB 磁盘空间

### 安装

1. 克隆仓库到本地：

```bash
git clone https://github.com/yourusername/H8UpgradeTool.git
cd H8UpgradeTool
```

2. 安装依赖：

```bash
pip install -r requirements.txt
```

3. 运行程序：

```bash
python src/main.py
```

## 📖 使用说明

1. **插入 H8 U 盘** - 将 H8 设备 U 盘插入电脑
2. **启动程序** - 运行 `main.py` 启动升级工具
3. **自动检测** - 程序会自动检测并识别 H8 U 盘
4. **版本比对** - 查看本地版本与云端最新版本差异
5. **开始升级** - 点击"开始升级"按钮，等待升级完成

## 🏗️ 项目结构

```
H8UpgradeTool/
├── README.md                 # 项目说明文档
├── PROJECT_PLAN.md           # 项目计划文档
├── requirements.txt          # Python 依赖列表
├── src/
│   ├── main.py              # 程序入口
│   ├── app.py               # 应用主类
│   ├── gui.py               # 图形界面模块
│   ├── usb_manager.py       # U 盘管理模块
│   ├── version_manager.py   # 版本管理模块
│   ├── github_client.py     # GitHub 访问客户端
│   ├── upgrade_manager.py   # 升级逻辑模块
│   ├── logger.py            # 日志记录模块
│   └── utils.py             # 工具函数
└── resources/
    ├── ui/                  # UI 资源文件
    └── icons/               # 图标资源
```

## 🛠️ 技术栈

- **开发语言**: Python 3.8+
- **UI 框架**: PyQt6 / PyQt5
- **HTTP 客户端**: requests
- **版本控制**: Git

## 📝 版本信息格式

### 本地版本 (VER.DAT)

```
P=H8-AP_MAX              # 产品名称
T=2026-04-20T10:30:00Z   # 发布时间
G=V0.0.0                 # 总版本号
1=A3.64.18.B2            # 遥控器主板版本
2=B1.64.20.A5            # 无线模块(主)版本
4=A2.5A.12.1F            # 声呐模块版本
5=C0.6C.05.2A            # 船控主板版本
6=D5.61.30.0B            # GPS模块版本
7=E2.64.22.3C            # 电调模块版本
8=F1.63.10.1D            # 风扇模块版本
```

### 云端版本 (version.json)

```json
{
  "product": "H8-AP_MAX",
  "release_time": "2026-04-20T10:30:00Z",
  "G_version": "V0.0.1",
  "G_resources": {
    "size": "12345678",
    "url": "https://github.com/.../v2.2.0.zip"
  },
  "IAP_modules": [
    {
      "id": 1,
      "name": "remote_control",
      "display": "遥控器主板",
      "version": "A3.63.25.A1",
      "file": "H8显示屏_A36325A1(all).iap"
    }
  ]
}
```

## 🔧 开发计划

- [x] 项目规划与设计
- [ ] U 盘自动检测模块
- [ ] 本地版本解析模块
- [ ] 云端版本获取模块
- [ ] 版本比对逻辑
- [ ] 下载与解压模块
- [ ] 文件复制与升级模块
- [ ] 图形界面开发
- [ ] 异常处理与日志
- [ ] 测试与发布

## ⚠️ 注意事项

1. 升级前请确保 U 盘已正确插入并被识别
2. 升级过程中请勿拔出 U 盘
3. 确保网络连接稳定以正常下载升级包
4. 建议在升级前备份重要数据

## 🤝 贡献

欢迎提交 Issue 和 Pull Request 来改进这个项目。

## 📄 许可证

本项目采用 [MIT](LICENSE) 许可证。

---

<p align="center">Made with ❤️ for H8 Device Users</p>

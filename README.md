# WPF 视觉检测上位机平台

一个基于 WPF 的高度可扩展的视觉检测上位机平台，集成多种视觉算法和深度学习模型。

## 🎯 核心功能

- **相机管理** - 支持多品牌相机（海康、大华、USB等）
- **算法支持** - Halcon、OpenCV、YOLO、Halcon 深度学习
- **通讯协议** - Modbus、TCP/IP、串口、OPC-UA
- **数据管理** - 存储、查询、导出（Excel、CSV、PDF）
- **工作流引擎** - 流程化处理、条件判断、循环控制
- **可扩展性** - 支持自定义算法和相机驱动接入

## 🔧 技术栈

- **框架**: .NET 6.0 LTS
- **UI**: WPF + MVVM Toolkit
- **数据库**: EF Core + SQLite/SQL Server
- **图像处理**: OpenCvSharp + Halcon SDK
- **日志**: Serilog
- **深度学习**: YOLO / Halcon DL

## 📦 项目结构

```
VisionInspectionPlatform/
├── src/
│   ├── VisionPlatform.UI/               # WPF UI 层
│   ├── VisionPlatform.Business/         # 业务逻辑层
│   ├── VisionPlatform.Algorithm/        # 算法层
│   ├── VisionPlatform.Infrastructure/   # 基础设施层
│   └── VisionPlatform.Common/           # 公共模块
├── tests/
│   └── VisionPlatform.Tests/            # 单元测试
├── docs/
│   ├── PROJECT_ARCHITECTURE.md
│   └── DEVELOPMENT_GUIDE.md
└── README.md
```

## 🚀 快速开始

### 系统要求
- Windows 10/11 (x64)
- Visual Studio 2022
- .NET 6.0 LTS 或更高
- 8GB+ 内存

### 克隆仓库
```bash
git clone https://github.com/hongfei-wan/vision-inspection-platform.git
cd vision-inspection-platform
```

### 构建项目
```bash
dotnet build
```

### 运行测试
```bash
dotnet test
```

### 启动应用
```bash
dotnet run --project src/VisionPlatform.UI
```

## 📚 文档

- [开发文档](./docs/WPF_Vision_Platform_Dev_Document.md) - 完整设计文档
- [项目架构](./docs/PROJECT_ARCHITECTURE.md) - 架构设计
- [开发指南](./docs/DEVELOPMENT_GUIDE.md) - 开发指南

## 📝 许可证

MIT License

## 👤 作者

[@hongfei-wan](https://github.com/hongfei-wan)

---

**最后更新**: 2026-05-25

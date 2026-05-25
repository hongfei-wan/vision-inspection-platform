# 开发指南

## 环境设置

### 1. 安装 .NET 6.0 LTS
```bash
# Windows
winget install Microsoft.DotNet.SDK.6

# 或从官网下载
# https://dotnet.microsoft.com/download/dotnet/6.0
```

### 2. 安装 Visual Studio 2022
- 下载并安装 Visual Studio 2022 Community
- 选择 ".NET desktop development" 工作负载
- 选择 "WPF" 选项
- 选择 "Entity Framework 6 tools" (可选)

### 3. 克隆仓库
```bash
git clone https://github.com/hongfei-wan/vision-inspection-platform.git
cd vision-inspection-platform
```

### 4. 恢复 NuGet 包
```bash
dotnet restore
```

### 5. 构建项目
```bash
dotnet build
```

## 项目结构

```
VisionInspectionPlatform/
├── src/
│   ├── VisionPlatform.UI/
│   │   ├── Views/               # WPF 视图
│   │   ├── ViewModels/          # MVVM 视图模型
│   │   ├── Converters/          # 值转换器
│   │   ├── Resources/           # 资源文件
│   │   └── App.xaml
│   │
│   ├── VisionPlatform.Business/
│   │   ├── CameraModule/        # 相机管理
│   │   ├── AlgorithmModule/     # 算法管理
│   │   ├── CommunicationModule/ # 通讯管理
│   │   ├── DataModule/          # 数据管理
│   │   ├── WorkflowModule/      # 工作流引擎
│   │   └── ReportModule/        # 报表生成
│   │
│   ├── VisionPlatform.Algorithm/
│   │   ├── HalconAlgorithm/     # Halcon 算法
│   │   ├── OpenCVAlgorithm/     # OpenCV 算法
│   │   ├── YOLOAlgorithm/       # YOLO 算法
│   │   ├── HalconDLAlgorithm/   # Halcon DL 算法
│   │   └── BaseAlgorithm/       # 基础类
│   │
│   ├── VisionPlatform.Infrastructure/
│   │   ├── Database/            # 数据库层
│   │   ├── FileSystem/          # 文件系统
│   │   ├── Logger/              # 日志服务
│   │   ├── Configuration/       # 配置管理
│   │   ├── Caching/             # 缓存服务
│   │   └── Utils/               # 工具类
│   │
│   └── VisionPlatform.Common/
│       ├── Enums/               # 枚举定义
│       ├── Models/              # 数据模型
│       ├── Exceptions/          # 异常定义
│       ├── Constants/           # 常量定义
│       └── Extensions/          # 扩展方法
│
├── tests/
│   └── VisionPlatform.Tests/
│       ├── AlgorithmTests/
│       ├── BusinessTests/
│       └── IntegrationTests/
│
├── docs/
│   ├── PROJECT_ARCHITECTURE.md  # 项目架构
│   ├── WPF_Vision_Platform_Dev_Document.md  # 完整文档
│   └── DEVELOPMENT_GUIDE.md     # 本文件
│
├── VisionPlatform.sln           # 解决方案文件
├── README.md                    # 项目说明
└── appsettings.json            # 配置文件
```

## 添加新功能

### 添加新算法

1. **在 VisionPlatform.Algorithm 中创建新文件夹**
```
VisionPlatform.Algorithm/
├── MyCustomAlgorithm/
│   ├── MyCustomAlgorithm.cs
│   └── Models/
│       └── MyResult.cs
```

2. **继承 AlgorithmBase 类**
```csharp
public class MyCustomAlgorithm : AlgorithmBase
{
    public override string Name => "我的自定义算法";
    public override string Version => "1.0";
    public override AlgorithmType Type => AlgorithmType.Custom;
    
    public override List<AlgorithmParameter> GetParameters()
    {
        return new List<AlgorithmParameter> { /* 参数定义 */ };
    }
    
    public override AlgorithmResult Execute(Bitmap image, Dictionary<string, object> parameters)
    {
        // 实现算法逻辑
    }
}
```

3. **在 App.xaml.cs 中注册**
```csharp
services.AddScoped<IAlgorithm, MyCustomAlgorithm>();
```

### 添加新相机

1. **在 VisionPlatform.Business/CameraModule/Drivers 中创建驱动**
```
CameraModule/
├── Drivers/
│   └── MyCustomCameraDriver.cs
```

2. **实现相机驱动接口**
```csharp
public class MyCustomCameraDriver : ICameraDriver
{
    public async Task ConnectAsync(CameraDevice device) { }
    public async Task DisconnectAsync() { }
    public async Task StartCaptureAsync() { }
    public async Task StopCaptureAsync() { }
    public async Task<Bitmap> GetFrameAsync() { }
}
```

3. **在 CameraFactory 中添加**
```csharp
public ICamera CreateCamera(CameraType type)
{
    return type switch
    {
        CameraType.MyCustom => new MyCustomCameraDriver(),
        _ => throw new ArgumentException()
    };
}
```

### 添加新通讯协议

1. **在 VisionPlatform.Business/CommunicationModule/Protocols 中创建协议**
2. **实现 IProtocol 接口**
3. **在 ProtocolFactory 中注册**

## 运行测试

```bash
# 运行所有测试
dotnet test

# 运行特定项目的测试
dotnet test tests/VisionPlatform.Tests/

# 运行包含特定名称的测试
dotnet test --filter "ClassName"

# 生成代码覆盖率报告
dotnet test /p:CollectCoverage=true /p:CoverageFormat=opencover
```

## 代码规范

### 命名规范
- 类名: PascalCase (如 `CameraService`)
- 方法名: PascalCase (如 `GetAvailableCameras`)
- 属性名: PascalCase (如 `IsConnected`)
- 私有字段: _camelCase (如 `_logger`)
- 参数: camelCase (如 `cameraDevice`)
- 常量: UPPER_CASE (如 `MAX_BUFFER_SIZE`)

### 注释规范
```csharp
/// <summary>
/// 获取可用的相机列表
/// </summary>
/// <returns>相机设备列表</returns>
public async Task<List<CameraDevice>> GetAvailableCamerasAsync()
{
    // 实现逻辑
}
```

### 异常处理
```csharp
try
{
    // 操作
}
catch (ArgumentException ex)
{
    _logger.LogError(ex, "参数错误");
    throw;
}
catch (Exception ex)
{
    _logger.LogError(ex, "发生未知错误");
    throw new VisionException("操作失败", ex);
}
```

### 异步编程
- 异步方法以 `Async` 后缀结尾
- 使用 `async/await` 而不是 `Task.Run()`
- 不要在构造函数中进行异步操作

## 提交代码

### 分支规范
- `main` - 主分支，存放稳定版本
- `develop` - 开发分支
- `feature/*` - 功能分支 (如 `feature/add-halcon-algorithm`)
- `bugfix/*` - 修复分支 (如 `bugfix/camera-connection-issue`)

### 提交信息
```bash
git add .
git commit -m "类型: 简短描述

详细描述（可选）
- 修改了什么
- 为什么修改
- 有什么影响
"
```

**提交类型**:
- `feat:` 新功能
- `fix:` 修复
- `docs:` 文档
- `style:` 代码风格
- `refactor:` 重构
- `test:` 测试
- `chore:` 构建/工具

### 推送代码
```bash
git push origin feature/your-feature-name
# 然后在 GitHub 上创建 Pull Request
```

## 常见问题

### Q: 如何调试代码？
A: 
1. 在代码中设置断点（点击行号）
2. 按 F5 启动调试
3. 使用调试工具栏控制执行

### Q: 如何查看日志？
A: 
1. 日志存储在 `logs/` 目录
2. 修改 `appsettings.json` 中的 `LogLevel` 改变日志级别
3. 在代码中使用 `ILogger` 记录日志

### Q: 数据库如何初始化？
A: 
1. 首次运行时自动创建数据库
2. 如需手动迁移: `dotnet ef database update`
3. 使用 `DbInitializer` 类初始化数据

### Q: 如何处理 NuGet 包版本冲突？
A:
```bash
# 清除本地缓存
dotnet nuget locals all --clear

# 重新恢复包
dotnet restore
```

### Q: 如何生成 API 文档？
A:
```bash
# 使用 DocFX
docfx docfx.json
```

## 性能优化建议

1. **异步处理** - 避免阻塞 UI 线程
2. **缓存结果** - 使用 `ICacheService` 缓存频繁使用的数据
3. **数据库索引** - 为常用查询列添加索引
4. **图像压缩** - 存储缩略图而不是原始图像
5. **GPU加速** - 在 `appsettings.json` 中启用 GPU

## 调试技巧

### 输出调试信息
```csharp
_logger.LogDebug($"相机连接成功: {camera.Name}");
_logger.LogInformation($"发现 {cameras.Count} 个相机");
_logger.LogWarning($"相机配置项无效: {param}");
_logger.LogError(ex, "相机连接失败");
```

### 使用监视窗口
- 在调试时右键点击变量 → "添加监视"
- 在调试期间查看变量的值变化

### 条件断点
- 右键点击断点 → "设置筛选器"
- 设置条件表达式，仅在条件满足时中断

## 参考资源

- [.NET 官方文档](https://docs.microsoft.com/dotnet/)
- [WPF 文档](https://docs.microsoft.com/windows/wpf/)
- [Entity Framework Core](https://docs.microsoft.com/ef/core/)
- [MVVM Toolkit](https://github.com/CommunityToolkit/dotnet)
- [Serilog](https://serilog.net/)

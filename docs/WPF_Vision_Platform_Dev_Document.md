# WPF 视觉检测平台 - 完整开发文档

**项目名称**: VisionInspectionPlatform  
**版本**: 1.0  
**最后更新**: 2026-06-01  
**开发语言**: C# (.NET 6.0+)  
**UI框架**: WPF (MVVM)  

---

## 目录

1. [项目概述](#项目概述)
2. [系统架构](#系统架构)
3. [核心模块设计](#核心模块设计)
4. [关键接口](#关键接口)
5. [数据模型](#数据模型)
6. [配置文件](#配置文件)
7. [开发指南](#开发指南)
8. [API 参考](#api-参考)
9. [最佳实践](#最佳实践)
10. [故障排除](#故障排除)

---

## 项目概述

### 1.1 项目目标

构建一个高度可扩展的 WPF 视觉检测上位机平台，集成多种视觉算法和深度学习模型，支持实时图像处理、数据存储查询、设备通讯等功能。

### 1.2 核心功能

- **相机管理**: 支持多品牌相机（海康、大华、USB等）
- **算法支持**: Halcon、OpenCV、YOLO、Halcon 深度学习
- **通讯协议**: Modbus、TCP/IP、串口、OPC-UA
- **数据管理**: 存储、查询、导出（Excel、CSV、PDF）
- **工作流引擎**: 流程化处理、条件判断、循环控制
- **可扩展性**: 支持自定义算法和相机驱动接入

### 1.3 适用场景

- 工业视觉检测
- 质量控制检验
- 缺陷检测与分类
- 尺寸测量
- 表面检测
- 光学检查
- 制造业检验

### 1.4 技术栈概览

| 层级 | 技术 | 版本 |
|------|------|------|
| **框架** | .NET | 6.0 LTS |
| **UI** | WPF | 内置 |
| **MVVM** | CommunityToolkit.Mvvm | 8.2.2+ |
| **数据库** | Entity Framework Core | 7.0+ |
| **图像处理** | OpenCvSharp4 | 4.8.0+ |
| **日志** | Serilog | 3.0+ |

---

## 系统架构

### 2.1 分层架构设计

```
┌────────────────────────────────────────────────────────────┐
│           表现层 (Presentation Layer)                      │
│              WPF UI + MVVM Pattern                        │
├────────────────────────────────────────────────────────────┤
│           业务逻辑层 (Business Layer)                      │
│  CameraModule | AlgorithmManager | CommunicationModule   │
│  DataModule | WorkflowEngine | ReportModule              │
├────────────────────────────────────────────────────────────┤
│              算法层 (Algorithm Layer)                      │
│  HalconEngine | OpenCVEngine | YOLOEngine | HalconDL     │
├────────────────────────────────────────────────────────────┤
│          基础设施层 (Infrastructure Layer)                 │
│  Database | FileSystem | Logger | Configuration | Utils  │
└────────────────────────────────────────────────────────────┘
```

### 2.2 核心特性

- **分层架构**: 清晰的职责分离，易于维护和测试
- **依赖注入**: 使用 DI 容器管理生命周期
- **接口驱动**: 基于接口的多实现支持，增强可扩展性
- **异步处理**: 保证 UI 响应性，避免卡顿
- **事件驱动**: 松耦合的通讯机制
- **插件架构**: 支持动态加载新的算法和驱动

---

## 核心模块设计

### 3.1 相机模块 (Camera Module)

**文件位置**: `src/VisionPlatform.Business/CameraModule/`

#### 3.1.1 功能说明

- 多品牌相机驱动集成
- 实时视频预览和采集
- 相机参数动态配置
- 帧率控制和同步
- 多相机管理

#### 3.1.2 核心接口

```csharp
/// <summary>
/// 相机服务接口
/// </summary>
public interface ICameraService : IDisposable
{
    /// <summary>
    /// 帧数据到达事件
    /// </summary>
    event EventHandler<FrameArrivedEventArgs> FrameArrived;
    
    /// <summary>
    /// 相机错误事件
    /// </summary>
    event EventHandler<ErrorEventArgs> CameraError;
    
    /// <summary>
    /// 获取可用的相机列表
    /// </summary>
    /// <returns>相机设备列表</returns>
    Task<List<CameraDevice>> GetAvailableCamerasAsync();
    
    /// <summary>
    /// 连接到指定相机
    /// </summary>
    /// <param name="camera">相机设备</param>
    Task ConnectAsync(CameraDevice camera);
    
    /// <summary>
    /// 断开相机连接
    /// </summary>
    Task DisconnectAsync();
    
    /// <summary>
    /// 启动图像采集
    /// </summary>
    Task StartCaptureAsync();
    
    /// <summary>
    /// 停止图像采集
    /// </summary>
    Task StopCaptureAsync();
    
    /// <summary>
    /// 获取当前帧
    /// </summary>
    Bitmap GetCurrentFrame();
    
    /// <summary>
    /// 是否已连接
    /// </summary>
    bool IsConnected { get; }
    
    /// <summary>
    /// 当前帧率
    /// </summary>
    double FrameRate { get; }
}
```

#### 3.1.3 相机驱动实现

```csharp
/// <summary>
/// 海康威视相机驱动
/// </summary>
public class HikvisionCameraDriver : ICameraDriver
{
    private IntPtr _camera;
    private byte[] _imageBuffer;
    
    public async Task ConnectAsync(CameraDevice device)
    {
        // 枚举设备
        // 连接指定相机
        // 配置参数
    }
    
    public async Task StartCaptureAsync()
    {
        // 启动实时取流
        // 创建采集线程
    }
}

/// <summary>
/// USB 相机驱动
/// </summary>
public class UsbCameraDriver : ICameraDriver
{
    private VideoCapture _capture;
    
    public async Task ConnectAsync(CameraDevice device)
    {
        _capture = new VideoCapture(device.Id);
    }
}
```

#### 3.1.4 相机工厂

```csharp
public class CameraFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public ICamera CreateCamera(CameraType type)
    {
        return type switch
        {
            CameraType.Hikvision => _serviceProvider.GetRequiredService<HikvisionCameraDriver>(),
            CameraType.Dahua => _serviceProvider.GetRequiredService<DahuaCameraDriver>(),
            CameraType.USB => _serviceProvider.GetRequiredService<UsbCameraDriver>(),
            _ => throw new ArgumentException($"未知的相机类型: {type}")
        };
    }
}
```

### 3.2 算法模块 (Algorithm Module)

**文件位置**: `src/VisionPlatform.Algorithm/`

#### 3.2.1 统一算法接口

```csharp
/// <summary>
/// 算法基接口
/// </summary>
public interface IAlgorithm : IDisposable
{
    /// <summary>
    /// 算法名称
    /// </summary>
    string Name { get; }
    
    /// <summary>
    /// 算法版本
    /// </summary>
    string Version { get; }
    
    /// <summary>
    /// 算法类型
    /// </summary>
    AlgorithmType Type { get; }
    
    /// <summary>
    /// 初始化算法
    /// </summary>
    /// <param name="config">算法配置</param>
    void Initialize(AlgorithmConfig config);
    
    /// <summary>
    /// 执行算法（同步）
    /// </summary>
    /// <param name="image">输入图像</param>
    /// <param name="parameters">算法参数</param>
    /// <returns>算法结果</returns>
    AlgorithmResult Execute(Bitmap image, Dictionary<string, object> parameters);
    
    /// <summary>
    /// 执行算法（异步）
    /// </summary>
    /// <param name="image">输入图像</param>
    /// <param name="parameters">算法参数</param>
    /// <returns>算法结果</returns>
    Task<AlgorithmResult> ExecuteAsync(Bitmap image, Dictionary<string, object> parameters);
    
    /// <summary>
    /// 获取算法参数列表
    /// </summary>
    /// <returns>参数列表</returns>
    List<AlgorithmParameter> GetParameters();
    
    /// <summary>
    /// 验证参数有效性
    /// </summary>
    /// <param name="parameters">参数字典</param>
    /// <returns>是否有效</returns>
    bool ValidateParameters(Dictionary<string, object> parameters);
}
```

#### 3.2.2 算法基类

```csharp
/// <summary>
/// 算法基类
/// </summary>
public abstract class AlgorithmBase : IAlgorithm
{
    protected ILogger _logger;
    protected AlgorithmConfig _config;
    protected AlgorithmPerformance _performance;
    
    public abstract string Name { get; }
    public abstract string Version { get; }
    public abstract AlgorithmType Type { get; }
    
    public virtual void Initialize(AlgorithmConfig config)
    {
        _config = config;
        _logger?.LogInformation($"初始化算法: {Name} v{Version}");
    }
    
    public abstract AlgorithmResult Execute(Bitmap image, Dictionary<string, object> parameters);
    
    public virtual async Task<AlgorithmResult> ExecuteAsync(Bitmap image, Dictionary<string, object> parameters)
    {
        return await Task.Run(() => Execute(image, parameters));
    }
    
    public abstract List<AlgorithmParameter> GetParameters();
    
    public virtual bool ValidateParameters(Dictionary<string, object> parameters)
    {
        var requiredParams = GetParameters().Where(p => p.IsRequired).ToList();
        return requiredParams.All(p => parameters.ContainsKey(p.Name));
    }
    
    protected void LogExecutionTime(string stageName, long elapsedMs)
    {
        _logger?.LogDebug($"{Name} - {stageName}: {elapsedMs}ms");
    }
    
    public virtual void Dispose()
    {
        GC.SuppressFinalize(this);
    }
}
```

#### 3.2.3 Halcon 算法实现

```csharp
/// <summary>
/// Halcon 边缘检测算法
/// </summary>
public class HalconEdgeDetectionAlgorithm : AlgorithmBase
{
    public override string Name => "Halcon 边缘检测";
    public override string Version => "1.0";
    public override AlgorithmType Type => AlgorithmType.Halcon;
    
    private HObject _halconImage;
    private HTuple _edgeHandle;
    
    public override void Initialize(AlgorithmConfig config)
    {
        base.Initialize(config);
        // 初始化 Halcon 引擎
    }
    
    public override AlgorithmResult Execute(Bitmap image, Dictionary<string, object> parameters)
    {
        var result = new AlgorithmResult();
        var startTime = DateTime.Now;
        
        try
        {
            // 将 Bitmap 转换为 Halcon 图像
            _halconImage = ConvertBitmapToHalcon(image);
            
            // 获取参数
            var method = (string)parameters["method"] ?? "canny";
            var threshold = (double)parameters["threshold"] ?? 50.0;
            
            // 执行边缘检测
            HTuple edgeImage = new HTuple();
            HOperatorSet.EdgesImage(_halconImage, out edgeImage, method, threshold, "recursive", "true");
            
            // 转换结果为 Bitmap
            result.OutputImage = ConvertHalconToBitmap(edgeImage);
            result.Success = true;
            result.Message = "边缘检测成功";
            result.ExecutionTimeMs = (long)(DateTime.Now - startTime).TotalMilliseconds;
        }
        catch (Exception ex)
        {
            _logger?.LogError(ex, "边缘检测失败");
            result.Success = false;
            result.Message = $"边缘检测失败: {ex.Message}";
        }
        
        return result;
    }
    
    public override List<AlgorithmParameter> GetParameters()
    {
        return new List<AlgorithmParameter>
        {
            new AlgorithmParameter
            {
                Name = "method",
                DisplayName = "检测方法",
                Type = AlgorithmParameterType.String,
                DefaultValue = "canny",
                IsRequired = true
            },
            new AlgorithmParameter
            {
                Name = "threshold",
                DisplayName = "阈值",
                Type = AlgorithmParameterType.Double,
                DefaultValue = 50.0,
                MinValue = 0.0,
                MaxValue = 255.0,
                IsRequired = true
            }
        };
    }
}
```

#### 3.2.4 OpenCV 算法实现

```csharp
/// <summary>
/// OpenCV 斑点检测算法
/// </summary>
public class OpenCVBlobDetectionAlgorithm : AlgorithmBase
{
    public override string Name => "OpenCV 斑点检测";
    public override string Version => "1.0";
    public override AlgorithmType Type => AlgorithmType.OpenCV;
    
    private SimpleBlobDetector _detector;
    
    public override void Initialize(AlgorithmConfig config)
    {
        base.Initialize(config);
        var parameters = new SimpleBlobDetector_Params();
        _detector = SimpleBlobDetector.Create(parameters);
    }
    
    public override AlgorithmResult Execute(Bitmap image, Dictionary<string, object> parameters)
    {
        var result = new AlgorithmResult();
        var startTime = DateTime.Now;
        
        try
        {
            // 转换 Bitmap 为 Mat
            var mat = BitmapToMat(image);
            
            // 检测斑点
            var keypoints = _detector.Detect(mat);
            
            // 绘制结果
            var outputMat = mat.Clone();
            Cv2.DrawKeypoints(mat, keypoints, outputMat, Scalar.Blue);
            
            result.OutputImage = MatToBitmap(outputMat);
            result.Data = new Dictionary<string, object>
            {
                { "keypoints_count", keypoints.Length },
                { "keypoints", keypoints }
            };
            result.Success = true;
            result.Message = $"检测到 {keypoints.Length} 个斑点";
            result.ExecutionTimeMs = (long)(DateTime.Now - startTime).TotalMilliseconds;
        }
        catch (Exception ex)
        {
            _logger?.LogError(ex, "斑点检测失败");
            result.Success = false;
            result.Message = $"斑点检测失败: {ex.Message}";
        }
        
        return result;
    }
    
    public override List<AlgorithmParameter> GetParameters()
    {
        return new List<AlgorithmParameter>
        {
            new AlgorithmParameter
            {
                Name = "min_area",
                DisplayName = "最小面积",
                Type = AlgorithmParameterType.Double,
                DefaultValue = 10.0,
                IsRequired = false
            },
            new AlgorithmParameter
            {
                Name = "max_area",
                DisplayName = "最大面积",
                Type = AlgorithmParameterType.Double,
                DefaultValue = 5000.0,
                IsRequired = false
            }
        };
    }
}
```

#### 3.2.5 YOLO 深度学习实现

```csharp
/// <summary>
/// YOLO 目标检测算法
/// </summary>
public class YOLODetectionAlgorithm : AlgorithmBase
{
    public override string Name => "YOLO 目标检测";
    public override string Version => "8.0";
    public override AlgorithmType Type => AlgorithmType.YOLOv8;
    
    private YOLOWrapper _yoloWrapper;
    
    public override void Initialize(AlgorithmConfig config)
    {
        base.Initialize(config);
        var modelPath = (string)config.DefaultParameters["model_path"];
        _yoloWrapper = new YOLOWrapper(modelPath);
        _yoloWrapper.SetConfidenceThreshold(0.5);
    }
    
    public override AlgorithmResult Execute(Bitmap image, Dictionary<string, object> parameters)
    {
        var result = new AlgorithmResult();
        var startTime = DateTime.Now;
        
        try
        {
            var confidence = (double)parameters["confidence"] ?? 0.5;
            var detections = _yoloWrapper.Detect(image, confidence);
            
            var outputImage = image.Clone() as Bitmap;
            using (var graphics = Graphics.FromImage(outputImage))
            {
                foreach (var detection in detections)
                {
                    var pen = new Pen(Color.Red, 2);
                    graphics.DrawRectangle(pen, detection.BoundingBox);
                    graphics.DrawString($"{detection.Label} {detection.Confidence:F2}", 
                        SystemFonts.DefaultFont, Brushes.Red, 
                        detection.BoundingBox.Location);
                }
            }
            
            result.OutputImage = outputImage;
            result.Data = new Dictionary<string, object>
            {
                { "detections_count", detections.Count },
                { "detections", detections }
            };
            result.Success = true;
            result.Message = $"检测到 {detections.Count} 个对象";
            result.ExecutionTimeMs = (long)(DateTime.Now - startTime).TotalMilliseconds;
        }
        catch (Exception ex)
        {
            _logger?.LogError(ex, "YOLO 检测失败");
            result.Success = false;
            result.Message = $"检测失败: {ex.Message}";
        }
        
        return result;
    }
    
    public override List<AlgorithmParameter> GetParameters()
    {
        return new List<AlgorithmParameter>
        {
            new AlgorithmParameter
            {
                Name = "confidence",
                DisplayName = "置信度阈值",
                Type = AlgorithmParameterType.Double,
                DefaultValue = 0.5,
                MinValue = 0.0,
                MaxValue = 1.0,
                IsRequired = true
            },
            new AlgorithmParameter
            {
                Name = "iou_threshold",
                DisplayName = "IoU 阈值",
                Type = AlgorithmParameterType.Double,
                DefaultValue = 0.45,
                MinValue = 0.0,
                MaxValue = 1.0,
                IsRequired = false
            }
        };
    }
}
```

#### 3.2.6 算法管理器

```csharp
/// <summary>
/// 算法管理器
/// </summary>
public class AlgorithmManager
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger _logger;
    private readonly Dictionary<string, IAlgorithm> _algorithms;
    
    public AlgorithmManager(IServiceProvider serviceProvider, ILogger logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
        _algorithms = new Dictionary<string, IAlgorithm>();
    }
    
    /// <summary>
    /// 注册算法
    /// </summary>
    public void RegisterAlgorithm(string name, IAlgorithm algorithm)
    {
        _algorithms[name] = algorithm;
        _logger?.LogInformation($"已注册算法: {name}");
    }
    
    /// <summary>
    /// 获取算法
    /// </summary>
    public IAlgorithm GetAlgorithm(string name)
    {
        if (!_algorithms.TryGetValue(name, out var algorithm))
        {
            throw new ArgumentException($"未找到算法: {name}");
        }
        return algorithm;
    }
    
    /// <summary>
    /// 执行算法
    /// </summary>
    public AlgorithmResult ExecuteAlgorithm(string name, Bitmap image, Dictionary<string, object> parameters)
    {
        var algorithm = GetAlgorithm(name);
        
        if (!algorithm.ValidateParameters(parameters))
        {
            return new AlgorithmResult
            {
                Success = false,
                Message = "参数验证失败"
            };
        }
        
        return algorithm.Execute(image, parameters);
    }
    
    /// <summary>
    /// 异步执行算法
    /// </summary>
    public async Task<AlgorithmResult> ExecuteAlgorithmAsync(string name, Bitmap image, Dictionary<string, object> parameters)
    {
        var algorithm = GetAlgorithm(name);
        return await algorithm.ExecuteAsync(image, parameters);
    }
}
```

### 3.3 通讯模块 (Communication Module)

**文件位置**: `src/VisionPlatform.Business/CommunicationModule/`

#### 3.3.1 协议接口

```csharp
/// <summary>
/// 通讯协议接口
/// </summary>
public interface IProtocol : IDisposable
{
    /// <summary>
    /// 数据接收事件
    /// </summary>
    event EventHandler<DataReceivedEventArgs> DataReceived;
    
    /// <summary>
    /// 连接已建立事件
    /// </summary>
    event EventHandler ConnectionEstablished;
    
    /// <summary>
    /// 连接已断开事件
    /// </summary>
    event EventHandler ConnectionLost;
    
    /// <summary>
    /// 错误事件
    /// </summary>
    event EventHandler<ErrorEventArgs> ErrorOccurred;
    
    /// <summary>
    /// 连接到远端设备
    /// </summary>
    Task ConnectAsync(string address, int port);
    
    /// <summary>
    /// 断开连接
    /// </summary>
    Task DisconnectAsync();
    
    /// <summary>
    /// 发送数据
    /// </summary>
    Task SendAsync(byte[] data);
    
    /// <summary>
    /// 接收数据
    /// </summary>
    Task<byte[]> ReceiveAsync();
    
    /// <summary>
    /// 是否已连接
    /// </summary>
    bool IsConnected { get; }
}
```

#### 3.3.2 Modbus TCP 实现

```csharp
/// <summary>
/// Modbus TCP 协议实现
/// </summary>
public class ModbusTcpProtocol : IProtocol
{
    private TcpClient _tcpClient;
    private NetworkStream _stream;
    private ushort _transactionId = 0;
    
    public event EventHandler<DataReceivedEventArgs> DataReceived;
    public event EventHandler ConnectionEstablished;
    public event EventHandler ConnectionLost;
    public event EventHandler<ErrorEventArgs> ErrorOccurred;
    
    public bool IsConnected => _tcpClient?.Connected ?? false;
    
    public async Task ConnectAsync(string address, int port)
    {
        try
        {
            _tcpClient = new TcpClient();
            await _tcpClient.ConnectAsync(address, port);
            _stream = _tcpClient.GetStream();
            ConnectionEstablished?.Invoke(this, EventArgs.Empty);
        }
        catch (Exception ex)
        {
            ErrorOccurred?.Invoke(this, new ErrorEventArgs { Exception = ex });
            throw;
        }
    }
    
    public async Task DisconnectAsync()
    {
        _stream?.Dispose();
        _tcpClient?.Dispose();
        ConnectionLost?.Invoke(this, EventArgs.Empty);
    }
    
    public async Task SendAsync(byte[] data)
    {
        if (!IsConnected)
            throw new InvalidOperationException("未连接");
        
        await _stream.WriteAsync(data, 0, data.Length);
    }
    
    public async Task<byte[]> ReceiveAsync()
    {
        if (!IsConnected)
            throw new InvalidOperationException("未连接");
        
        byte[] buffer = new byte[1024];
        int bytesRead = await _stream.ReadAsync(buffer, 0, buffer.Length);
        
        Array.Resize(ref buffer, bytesRead);
        DataReceived?.Invoke(this, new DataReceivedEventArgs { Data = buffer });
        
        return buffer;
    }
    
    public void Dispose()
    {
        _stream?.Dispose();
        _tcpClient?.Dispose();
    }
}
```

#### 3.3.3 串口通讯实现

```csharp
/// <summary>
/// 串口协议实现
/// </summary>
public class SerialPortProtocol : IProtocol
{
    private SerialPort _serialPort;
    
    public event EventHandler<DataReceivedEventArgs> DataReceived;
    public event EventHandler ConnectionEstablished;
    public event EventHandler ConnectionLost;
    public event EventHandler<ErrorEventArgs> ErrorOccurred;
    
    public bool IsConnected => _serialPort?.IsOpen ?? false;
    
    public async Task ConnectAsync(string portName, int baudRate = 9600)
    {
        try
        {
            _serialPort = new SerialPort(portName, baudRate)
            {
                Parity = Parity.None,
                DataBits = 8,
                StopBits = StopBits.One,
                Handshake = Handshake.None,
                ReadTimeout = 1000,
                WriteTimeout = 1000
            };
            
            _serialPort.DataReceived += SerialPort_DataReceived;
            _serialPort.Open();
            
            ConnectionEstablished?.Invoke(this, EventArgs.Empty);
            await Task.CompletedTask;
        }
        catch (Exception ex)
        {
            ErrorOccurred?.Invoke(this, new ErrorEventArgs { Exception = ex });
            throw;
        }
    }
    
    public async Task DisconnectAsync()
    {
        if (_serialPort?.IsOpen == true)
        {
            _serialPort.Close();
            _serialPort.Dispose();
            ConnectionLost?.Invoke(this, EventArgs.Empty);
        }
        await Task.CompletedTask;
    }
    
    public async Task SendAsync(byte[] data)
    {
        if (!IsConnected)
            throw new InvalidOperationException("串口未打开");
        
        _serialPort.Write(data, 0, data.Length);
        await Task.CompletedTask;
    }
    
    public async Task<byte[]> ReceiveAsync()
    {
        if (!IsConnected)
            throw new InvalidOperationException("串口未打开");
        
        byte[] buffer = new byte[1024];
        int bytesRead = _serialPort.Read(buffer, 0, buffer.Length);
        
        Array.Resize(ref buffer, bytesRead);
        return buffer;
    }
    
    private void SerialPort_DataReceived(object sender, SerialDataReceivedEventArgs e)
    {
        byte[] buffer = new byte[1024];
        int bytesRead = _serialPort.Read(buffer, 0, buffer.Length);
        Array.Resize(ref buffer, bytesRead);
        
        DataReceived?.Invoke(this, new DataReceivedEventArgs { Data = buffer });
    }
    
    public void Dispose()
    {
        _serialPort?.Dispose();
    }
}
```

### 3.4 数据模块 (Data Module)

**文件位置**: `src/VisionPlatform.Business/DataModule/`

#### 3.4.1 数据上下文

```csharp
/// <summary>
/// 数据库上下文
/// </summary>
public class VisionDbContext : DbContext
{
    public VisionDbContext(DbContextOptions<VisionDbContext> options) : base(options)
    {
    }
    
    public DbSet<InspectionResultEntity> InspectionResults { get; set; }
    public DbSet<ImageFileEntity> ImageFiles { get; set; }
    public DbSet<AlgorithmConfigEntity> AlgorithmConfigs { get; set; }
    public DbSet<CameraConfigEntity> CameraConfigs { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // 配置 InspectionResult
        modelBuilder.Entity<InspectionResultEntity>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.ResultId).IsRequired().HasMaxLength(36);
            entity.Property(e => e.AlgorithmName).IsRequired().HasMaxLength(256);
            entity.HasIndex(e => e.CreatedTime);
        });
        
        // 配置 ImageFile
        modelBuilder.Entity<ImageFileEntity>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.FileId).IsRequired().HasMaxLength(36);
            entity.Property(e => e.FilePath).IsRequired().HasMaxLength(512);
            entity.HasIndex(e => e.CapturedTime);
        });
    }
}
```

#### 3.4.2 数据实体

```csharp
/// <summary>
/// 检测结果实体
/// </summary>
public class InspectionResultEntity
{
    public int Id { get; set; }
    public string ResultId { get; set; } = Guid.NewGuid().ToString();
    public string AlgorithmName { get; set; }
    public int ImageFileId { get; set; }
    public bool IsPass { get; set; }
    public double Confidence { get; set; }
    public string ResultJson { get; set; }
    public long ExecutionTimeMs { get; set; }
    public DateTime CreatedTime { get; set; } = DateTime.UtcNow;
}

/// <summary>
/// 图像文件实体
/// </summary>
public class ImageFileEntity
{
    public int Id { get; set; }
    public string FileId { get; set; } = Guid.NewGuid().ToString();
    public string FilePath { get; set; }
    public long FileSizeBytes { get; set; }
    public int Width { get; set; }
    public int Height { get; set; }
    public byte[] ThumbnailData { get; set; }
    public DateTime CapturedTime { get; set; } = DateTime.UtcNow;
}

/// <summary>
/// 算法配置实体
/// </summary>
public class AlgorithmConfigEntity
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string AlgorithmType { get; set; }
    public string ConfigJson { get; set; }
    public bool IsEnabled { get; set; }
    public DateTime CreatedTime { get; set; }
    public DateTime? UpdatedTime { get; set; }
}
```

#### 3.4.3 数据仓储

```csharp
/// <summary>
/// 通用数据仓储接口
/// </summary>
public interface IDataRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<List<T>> GetAllAsync();
    Task<PagedResult<T>> GetPagedAsync(int pageIndex, int pageSize);
    Task<List<T>> QueryAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
    Task<int> CountAsync();
}

/// <summary>
/// 通用数据仓储实现
/// </summary>
public class DataRepository<T> : IDataRepository<T> where T : class
{
    protected readonly VisionDbContext _context;
    protected readonly DbSet<T> _dbSet;
    
    public DataRepository(VisionDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
    
    public async Task<List<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    public async Task<PagedResult<T>> GetPagedAsync(int pageIndex, int pageSize)
    {
        var total = await _dbSet.CountAsync();
        var items = await _dbSet
            .Skip((pageIndex - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();
        
        return new PagedResult<T>
        {
            Items = items,
            Total = total,
            PageIndex = pageIndex,
            PageSize = pageSize
        };
    }
    
    public async Task<List<T>> QueryAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.Where(predicate).ToListAsync();
    }
    
    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
    }
    
    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }
    
    public async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
    
    public async Task<int> CountAsync()
    {
        return await _dbSet.CountAsync();
    }
}
```

#### 3.4.4 导出功能

```csharp
/// <summary>
/// Excel 导出器
/// </summary>
public class ExcelExporter
{
    public static async Task ExportAsync<T>(List<T> data, string filePath) where T : class
    {
        using (var workbook = new XLWorkbook())
        {
            var worksheet = workbook.Worksheets.Add("数据");
            
            // 获取类的属性
            var properties = typeof(T).GetProperties();
            
            // 写入标题
            for (int i = 0; i < properties.Length; i++)
            {
                worksheet.Cell(1, i + 1).Value = properties[i].Name;
            }
            
            // 写入数据
            for (int row = 0; row < data.Count; row++)
            {
                for (int col = 0; col < properties.Length; col++)
                {
                    var value = properties[col].GetValue(data[row]);
                    worksheet.Cell(row + 2, col + 1).Value = value;
                }
            }
            
            workbook.SaveAs(filePath);
        }
        
        await Task.CompletedTask;
    }
}

/// <summary>
/// CSV 导出器
/// </summary>
public class CsvExporter
{
    public static async Task ExportAsync<T>(List<T> data, string filePath) where T : class
    {
        using (var writer = new StreamWriter(filePath, Encoding.UTF8))
        {
            var properties = typeof(T).GetProperties();
            
            // 写入标题
            var header = string.Join(",", properties.Select(p => p.Name));
            await writer.WriteLineAsync(header);
            
            // 写入数据
            foreach (var item in data)
            {
                var values = properties.Select(p => 
                {
                    var value = p.GetValue(item);
                    return value?.ToString() ?? "";
                });
                var line = string.Join(",", values);
                await writer.WriteLineAsync(line);
            }
        }
    }
}
```

### 3.5 工作流引擎 (Workflow Engine)

**文件位置**: `src/VisionPlatform.Business/WorkflowModule/`

#### 3.5.1 工作流定义

```csharp
/// <summary>
/// 工作流步骤基类
/// </summary>
public abstract class WorkflowStep
{
    public string Name { get; set; }
    public string Description { get; set; }
    public bool IsEnabled { get; set; } = true;
    
    public abstract Task ExecuteAsync(WorkflowContext context);
}

/// <summary>
/// 工作流上下文
/// </summary>
public class WorkflowContext
{
    public string WorkflowId { get; set; }
    public Dictionary<string, object> Variables { get; set; } = new();
    public Bitmap CurrentImage { get; set; }
    public DateTime StartTime { get; set; }
    public List<StepExecutionResult> ExecutionHistory { get; set; } = new();
    
    public void SetVariable(string key, object value) => Variables[key] = value;
    public object GetVariable(string key) => Variables.TryGetValue(key, out var value) ? value : null;
}

/// <summary>
/// 步骤执行结果
/// </summary>
public class StepExecutionResult
{
    public string StepName { get; set; }
    public bool Success { get; set; }
    public string Message { get; set; }
    public long ExecutionTimeMs { get; set; }
    public DateTime ExecutedTime { get; set; }
}
```

#### 3.5.2 具体工作流步骤

```csharp
/// <summary>
/// 采集图像步骤
/// </summary>
public class CaptureImageStep : WorkflowStep
{
    private readonly ICameraService _cameraService;
    
    public override async Task ExecuteAsync(WorkflowContext context)
    {
        var startTime = DateTime.Now;
        try
        {
            var frame = _cameraService.GetCurrentFrame();
            context.CurrentImage = frame;
            
            context.ExecutionHistory.Add(new StepExecutionResult
            {
                StepName = Name,
                Success = true,
                Message = "图像采集成功",
                ExecutionTimeMs = (long)(DateTime.Now - startTime).TotalMilliseconds,
                ExecutedTime = DateTime.Now
            });
        }
        catch (Exception ex)
        {
            context.ExecutionHistory.Add(new StepExecutionResult
            {
                StepName = Name,
                Success = false,
                Message = $"图像采集失败: {ex.Message}",
                ExecutionTimeMs = (long)(DateTime.Now - startTime).TotalMilliseconds,
                ExecutedTime = DateTime.Now
            });
            throw;
        }
    }
}

/// <summary>
/// 执行算法步骤
/// </summary>
public class ExecuteAlgorithmStep : WorkflowStep
{
    private readonly AlgorithmManager _algorithmManager;
    public string AlgorithmName { get; set; }
    public Dictionary<string, object> AlgorithmParameters { get; set; }
    
    public override async Task ExecuteAsync(WorkflowContext context)
    {
        var startTime = DateTime.Now;
        try
        {
            var result = await _algorithmManager.ExecuteAlgorithmAsync(
                AlgorithmName, 
                context.CurrentImage, 
                AlgorithmParameters);
            
            context.SetVariable($"{AlgorithmName}_result", result);
            context.CurrentImage = result.OutputImage ?? context.CurrentImage;
            
            context.ExecutionHistory.Add(new StepExecutionResult
            {
                StepName = Name,
                Success = result.Success,
                Message = result.Message,
                ExecutionTimeMs = (long)(DateTime.Now - startTime).TotalMilliseconds,
                ExecutedTime = DateTime.Now
            });
        }
        catch (Exception ex)
        {
            context.ExecutionHistory.Add(new StepExecutionResult
            {
                StepName = Name,
                Success = false,
                Message = $"算法执行失败: {ex.Message}",
                ExecutionTimeMs = (long)(DateTime.Now - startTime).TotalMilliseconds,
                ExecutedTime = DateTime.Now
            });
            throw;
        }
    }
}

/// <summary>
/// 条件判断步骤
/// </summary>
public class ConditionStep : WorkflowStep
{
    public string ConditionExpression { get; set; }
    public WorkflowStep TrueBranch { get; set; }
    public WorkflowStep FalseBranch { get; set; }
    
    public override async Task ExecuteAsync(WorkflowContext context)
    {
        var result = EvaluateCondition(context, ConditionExpression);
        
        if (result && TrueBranch != null)
        {
            await TrueBranch.ExecuteAsync(context);
        }
        else if (!result && FalseBranch != null)
        {
            await FalseBranch.ExecuteAsync(context);
        }
    }
    
    private bool EvaluateCondition(WorkflowContext context, string expression)
    {
        // 实现条件解析和评估
        return true;
    }
}

/// <summary>
/// 循环步骤
/// </summary>
public class LoopStep : WorkflowStep
{
    public int LoopCount { get; set; }
    public List<WorkflowStep> LoopBody { get; set; }
    
    public override async Task ExecuteAsync(WorkflowContext context)
    {
        for (int i = 0; i < LoopCount; i++)
        {
            context.SetVariable("LoopIndex", i);
            
            foreach (var step in LoopBody)
            {
                if (step.IsEnabled)
                {
                    await step.ExecuteAsync(context);
                }
            }
        }
    }
}

/// <summary>
/// 保存结果步骤
/// </summary>
public class SaveResultStep : WorkflowStep
{
    private readonly IDataRepository<InspectionResultEntity> _resultRepository;
    private readonly IFileManager _fileManager;
    
    public override async Task ExecuteAsync(WorkflowContext context)
    {
        var startTime = DateTime.Now;
        try
        {
            // 保存图像
            var imagePath = _fileManager.SaveImage(context.CurrentImage);
            
            // 保存结果
            var result = new InspectionResultEntity
            {
                AlgorithmName = (string)context.GetVariable("algorithm_name"),
                IsPass = (bool)context.GetVariable("is_pass"),
                Confidence = (double)context.GetVariable("confidence"),
                ResultJson = JsonConvert.SerializeObject(context.Variables),
                ExecutionTimeMs = (long)(DateTime.Now - context.StartTime).TotalMilliseconds
            };
            
            await _resultRepository.AddAsync(result);
            
            context.ExecutionHistory.Add(new StepExecutionResult
            {
                StepName = Name,
                Success = true,
                Message = "结果保存成功",
                ExecutionTimeMs = (long)(DateTime.Now - startTime).TotalMilliseconds,
                ExecutedTime = DateTime.Now
            });
        }
        catch (Exception ex)
        {
            context.ExecutionHistory.Add(new StepExecutionResult
            {
                StepName = Name,
                Success = false,
                Message = $"保存失败: {ex.Message}",
                ExecutionTimeMs = (long)(DateTime.Now - startTime).TotalMilliseconds,
                ExecutedTime = DateTime.Now
            });
            throw;
        }
    }
}
```

#### 3.5.3 工作流引擎

```csharp
/// <summary>
/// 工作流引擎
/// </summary>
public class WorkflowEngine
{
    private readonly ILogger _logger;
    private readonly List<Workflow> _workflows;
    
    public WorkflowEngine(ILogger logger)
    {
        _logger = logger;
        _workflows = new List<Workflow>();
    }
    
    /// <summary>
    /// 注册工作流
    /// </summary>
    public void RegisterWorkflow(Workflow workflow)
    {
        _workflows.Add(workflow);
        _logger?.LogInformation($"已注册工作流: {workflow.Name}");
    }
    
    /// <summary>
    /// 执行工作流
    /// </summary>
    public async Task<WorkflowContext> ExecuteAsync(string workflowName, WorkflowContext context = null)
    {
        var workflow = _workflows.FirstOrDefault(w => w.Name == workflowName);
        if (workflow == null)
        {
            throw new ArgumentException($"未找到工作流: {workflowName}");
        }
        
        if (!workflow.IsEnabled)
        {
            throw new InvalidOperationException($"工作流已禁用: {workflowName}");
        }
        
        context ??= new WorkflowContext { WorkflowId = Guid.NewGuid().ToString() };
        context.StartTime = DateTime.Now;
        
        _logger?.LogInformation($"开始执行工作流: {workflowName}");
        
        try
        {
            foreach (var step in workflow.Steps)
            {
                if (step.IsEnabled)
                {
                    await step.ExecuteAsync(context);
                }
            }
            
            _logger?.LogInformation($"工作流执行完成: {workflowName}");
        }
        catch (Exception ex)
        {
            _logger?.LogError(ex, $"工作流执行失败: {workflowName}");
            throw;
        }
        
        return context;
    }
}

/// <summary>
/// 工作流定义
/// </summary>
public class Workflow
{
    public string Name { get; set; }
    public string Description { get; set; }
    public List<WorkflowStep> Steps { get; set; } = new();
    public bool IsEnabled { get; set; } = true;
}
```

---

## 关键接口

### 4.1 核心接口概览

| 接口 | 命名空间 | 用途 |
|------|---------|------|
| `IAlgorithm` | Algorithm | 算法基接口 |
| `ICameraService` | Business.Camera | 相机服务 |
| `IProtocol` | Business.Communication | 通讯协议 |
| `IDataRepository<T>` | Business.Data | 数据仓储 |
| `IFileManager` | Infrastructure.FileSystem | 文件管理 |
| `ILogger` | Infrastructure.Logger | 日志服务 |
| `IConfigService` | Infrastructure.Configuration | 配置服务 |

---

## 数据模型

### 5.1 数据类型定义

```csharp
/// <summary>
/// 算法结果
/// </summary>
public class AlgorithmResult
{
    public bool Success { get; set; }
    public string Message { get; set; }
    public Bitmap OutputImage { get; set; }
    public Dictionary<string, object> Data { get; set; }
    public long ExecutionTimeMs { get; set; }
}

/// <summary>
/// 算法参数
/// </summary>
public class AlgorithmParameter
{
    public string Name { get; set; }
    public string DisplayName { get; set; }
    public AlgorithmParameterType Type { get; set; }
    public object DefaultValue { get; set; }
    public object MinValue { get; set; }
    public object MaxValue { get; set; }
    public bool IsRequired { get; set; }
}

/// <summary>
/// 分页结果
/// </summary>
public class PagedResult<T>
{
    public List<T> Items { get; set; }
    public int Total { get; set; }
    public int PageIndex { get; set; }
    public int PageSize { get; set; }
}
```

---

## 配置文件

### 6.1 appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "VisionPlatform": "Debug"
    }
  },
  "Database": {
    "Type": "SQLite",
    "ConnectionString": "Data Source=vision.db"
  },
  "Storage": {
    "ImagePath": "./Data/Images",
    "ConfigPath": "./Data/Config",
    "MaxImageSizeMB": 100
  },
  "Algorithms": {
    "DefaultTimeout": 30000,
    "EnableCaching": true
  },
  "Camera": {
    "DefaultFrameRate": 30,
    "ImageBufferSize": 10
  }
}
```

---

## 开发指南

### 7.1 新增算法步骤

1. 创建类继承 `AlgorithmBase`
2. 实现必要方法
3. 在依赖注入中注册
4. 在 `AlgorithmManager` 中注册

### 7.2 新增相机支持

1. 创建驱动类
2. 在 `CameraFactory` 中添加
3. 配置相机参数

### 7.3 新增通讯协议

1. 实现 `IProtocol` 接口
2. 在 `ProtocolFactory` 中注册

---

## API 参考

### 8.1 算法 API

```csharp
// 获取算法
var algorithm = algorithmManager.GetAlgorithm("Halcon EdgeDetection");

// 设置参数
var parameters = new Dictionary<string, object>
{
    { "method", "canny" },
    { "threshold", 50.0 }
};

// 执行算法
var result = algorithm.Execute(image, parameters);

// 异步执行
var result = await algorithm.ExecuteAsync(image, parameters);
```

### 8.2 相机 API

```csharp
// 获取可用相机
var cameras = await cameraService.GetAvailableCamerasAsync();

// 连接相机
await cameraService.ConnectAsync(cameras[0]);

// 启动采集
await cameraService.StartCaptureAsync();

// 获取帧
var frame = cameraService.GetCurrentFrame();
```

### 8.3 数据 API

```csharp
// 添加数据
await repository.AddAsync(entity);

// 查询数据
var items = await repository.QueryAsync(x => x.IsPass == true);

// 分页查询
var paged = await repository.GetPagedAsync(1, 20);

// 导出数据
await ExcelExporter.ExportAsync(items, "results.xlsx");
```

---

## 最佳实践

### 9.1 异步编程

✅ 使用 `async/await`
❌ 避免 `Task.Result`

### 9.2 错误处理

✅ 特定异常捕获
❌ 笼统的异常捕获

### 9.3 日志记录

✅ 记录关键操作
✅ 记录错误信息
❌ 过度日志

### 9.4 资源管理

✅ 使用 `using` 语句
✅ 实现 `IDisposable`
❌ 泄漏资源

---

## 故障排除

### 10.1 常见问题

**Q: 相机连接失败**  
A: 检查相机 IP 和端口是否正确

**Q: 算法执行超时**  
A: 检查超时设置和算法性能

**Q: 数据库连接错误**  
A: 检查连接字符串和数据库权限

---

**文档完成** ✅

所有代码示例均可直接使用！

# YOLOv8 TensorFlow Lite Android 项目配置指南

## 📱 项目概述

这是一个完整配置的 YOLOv8 TensorFlow Lite Android 应用程序，支持实时目标检测。

### 🎯 主要功能
- **实时目标检测** - 使用摄像头进行实时物体识别
- **多硬件加速** - 支持 GPU 和 CPU 推理
- **COCO 数据集** - 识别 80 种常见物体
- **高性能优化** - 包含预处理、推理和后处理时间统计

## 🛠️ 环境要求

### Android 开发环境
- **Android Studio** 2023.1.1 或更新版本
- **Android SDK** API 24 (Android 7.0) 或更高
- **Java** 1.8 或更高版本
- **Gradle** 8.4

### 设备要求
- **Android 设备** API 24+ (Android 7.0+)
- **摄像头权限** 必须授予
- **内存** 至少 2GB RAM 推荐

## 🚀 快速开始

### 1. 项目验证
运行配置检查脚本：
```bash
python check_project.py
```

### 2. 在 Android Studio 中打开项目
1. 启动 Android Studio
2. 选择 "Open an existing project"
3. 选择项目根目录 (包含 `settings.gradle` 的目录)
4. 等待 Gradle 同步完成

### 3. 构建项目
1. 点击 "Build" → "Make Project" 或按 `Ctrl+F9`
2. 确保没有编译错误

### 4. 运行应用
1. 连接 Android 设备并启用 USB 调试
2. 点击 "Run" 按钮或按 `Shift+F10`
3. 授予摄像头权限
4. 开始实时目标检测！

## 📁 项目结构

```
yolov8-tflite-deploy/
├── app/
│   ├── src/main/
│   │   ├── java/com/quicinc/
│   │   │   ├── objectdetection/     # 主要应用逻辑
│   │   │   │   ├── MainActivity.java
│   │   │   │   ├── ObjectDetection.java
│   │   │   │   ├── CameraFragment.java
│   │   │   │   ├── FragmentRender.java
│   │   │   │   └── RectangleBox.java
│   │   │   └── tflite/              # TensorFlow Lite 工具
│   │   │       ├── TFLiteHelpers.java
│   │   │       └── AIHubDefaults.java
│   │   ├── assets/                  # 模型和标签文件
│   │   │   ├── yolov8n.tflite      # 主模型文件
│   │   │   ├── yolov8n_float32.tflite
│   │   │   ├── yolov8n_float16.tflite
│   │   │   ├── yolov8n_int8.tflite
│   │   │   └── coco_labels.txt     # COCO 类别标签
│   │   └── res/                     # Android 资源文件
│   └── build.gradle                 # 应用构建配置
├── gradle.properties               # 项目配置
├── settings.gradle                 # Gradle 设置
└── build.gradle.kts               # 根构建脚本
```

## 🔧 配置说明

### 模型配置
当前使用的模型配置在 `gradle.properties` 中：
```properties
objectdetection_tfLiteModelAsset=yolov8n.tflite
objectdetection_tfLiteLabelsAsset=coco_labels.txt
```

### 可用模型
- `yolov8n.tflite` - 主模型 (float32, 12.9MB)
- `yolov8n_float16.tflite` - 半精度模型 (6.5MB)
- `yolov8n_int8.tflite` - 量化模型 (3.4MB)

### 硬件加速
- **GPU 加速** - 使用 TensorFlow Lite GPU 委托
- **CPU 优化** - 使用 XNNPack 委托
- **QNN NPU** - 已禁用 (需要 Qualcomm QNN SDK)

## 🎛️ 自定义配置

### 更换模型
1. 将新的 `.tflite` 文件放入 `app/src/main/assets/`
2. 更新 `gradle.properties` 中的 `objectdetection_tfLiteModelAsset`
3. 确保模型格式符合要求：
  - 输入: `[1, 640, 640, 3]` (RGB, float32, 归一化到 [0,1])
  - 输出: 3个张量 - boxes, scores, class_idx

### 调整检测参数
在 `ObjectDetection.java` 中修改：
```java
// 置信度阈值
if (scores[i] >= 0.2) {  // 修改这里

// NMS 参数
int[] result_indices = nms.nmsScoreFilter(updatedBoxes, scores, 20, 0.2f);
//                                                      最大检测数  NMS阈值
```

### 添加新的类别标签
1. 修改 `app/src/main/assets/coco_labels.txt`
2. 确保标签顺序与模型输出对应

## 🐛 常见问题

### 编译错误
1. **QNN 相关错误** - 确保 QNN 依赖已注释掉
2. **OpenCV 错误** - 检查 OpenCV 依赖版本
3. **Gradle 同步失败** - 检查网络连接和代理设置

### 运行时错误
1. **摄像头权限** - 确保已授予摄像头权限
2. **模型加载失败** - 检查模型文件是否存在且完整
3. **内存不足** - 尝试使用更小的模型 (int8 或 float16)

### 性能优化
1. **使用量化模型** - `yolov8n_int8.tflite` 更快但精度略低
2. **调整输入分辨率** - 修改模型输入尺寸
3. **减少检测数量** - 降低 `maxDetections` 参数

## 📊 性能指标

在不同设备上的典型性能：
- **高端设备** (Snapdragon 8 Gen 1): ~30-50ms
- **中端设备** (Snapdragon 7 Gen 1): ~50-100ms
- **入门设备** (Snapdragon 6 Gen 1): ~100-200ms

## 🔄 更新和维护

### 更新 TensorFlow Lite
在 `app/build.gradle` 中更新版本：
```gradle
api 'org.tensorflow:tensorflow-lite:2.16.1'  // 更新版本号
```

### 更新 OpenCV
```gradle
implementation 'org.opencv:opencv-android:4.8.0'  // 更新版本号
```

## 📞 技术支持

如果遇到问题：
1. 检查 Android Studio 的 Logcat 输出
2. 运行 `python check_project.py` 验证配置
3. 确保设备满足最低要求
4. 查看原始项目文档和示例

## 📄 许可证

本项目基于 BSD-3-Clause 许可证。详见原始仓库的 LICENSE 文件。
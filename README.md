# yolov9_tensorRT_Cplusplus

yolov9 tensorRT C++部署。

本示例中，包含完整的代码、模型、测试图片、测试结果。

TensorRT版本：TensorRT-7.1.3.4

## 导出onnx模型

导出适配本实例的onnx模型参考[【yolov9 瑞芯微芯片rknn部署、地平线芯片Horizon部署、TensorRT部署】](https://blog.csdn.net/zhangqian_1/article/details/136321979)。


## 修改 CMakeLists.txt 对应的TensorRT位置

```powershell
  cmake_minimum_required(VERSION 3.5)

  project(yolo_trt)

  set(CMAKE_CXX_STANDARD 14)
  
  # CUDA
  find_package(CUDA REQUIRED)
  message(STATUS "Find CUDA include at ${CUDA_INCLUDE_DIRS}")
  message(STATUS "Find CUDA libraries: ${CUDA_LIBRARIES}")
  
  # TensorRT
  set(TENSORRT_ROOT /zhangqian/workspaces1/TensorRT/TensorRT-7.1.3.4/TensorRT-7.1.3.4)

```

## 编译
```powershell
cd yoloworld_tensorRT_Cplusplus
mkdir build
cd build
cmake ..
make
```

## 运行

```powershell
# 运行时如果.trt模型存在则直接加载，若不存会自动先将onnx转换成 trt 模型，并存在给定的位置，然后运行推理。
cd build
./yolo_trt
```

## 测试效果

onnx 测试效果

![test](https://github.com/cqu20160901/yolov9_tensorRT_Cplusplus/assets/22290931/0a91af3c-37b6-4827-8857-0f7ce95e7794)


tensorRT 测试效果

![image](https://github.com/cqu20160901/yoloworld-tensorRT-Cplusplus/blob/main/images/result.jpg)


## 替换模型说明

1）按照本实例给的导出onnx方式导出对应的onnx；导出的onnx模型建议simplify后再转trt模型。

2）注意修改后处理相关 postprocess.hpp 中相关的参数（类别、输入分辨率等）。

修改相关的路径
```cpp
    std::string OnnxFile = "/zhangqian/workspaces1/TensorRT/yoloworld_trt_Cplusplus/models/yolo_world_v2.onnx";
    std::string SaveTrtFilePath = "/zhangqian/workspaces1/TensorRT/yoloworld_trt_Cplusplus/models/yolo_world_v2.trt";
    cv::Mat SrcImage = cv::imread("/zhangqian/workspaces1/TensorRT/yoloworld_trt_Cplusplus/images/test.jpg");

    int img_width = SrcImage.cols;
    int img_height = SrcImage.rows;

    CNN YOLO(OnnxFile, SaveTrtFilePath, 1, 3, 640, 640, 7);
    YOLO.ModelInit();
    YOLO.Inference(SrcImage);

    for (int i = 0; i < YOLO.DetectiontRects_.size(); i += 6)
    {
        int classId = int(YOLO.DetectiontRects_[i + 0]);
        float conf = YOLO.DetectiontRects_[i + 1];
        int xmin = int(YOLO.DetectiontRects_[i + 2] * float(img_width) + 0.5);
        int ymin = int(YOLO.DetectiontRects_[i + 3] * float(img_height) + 0.5);
        int xmax = int(YOLO.DetectiontRects_[i + 4] * float(img_width) + 0.5);
        int ymax = int(YOLO.DetectiontRects_[i + 5] * float(img_height) + 0.5);

        char text1[256];
        sprintf(text1, "%d:%.2f", classId, conf);
        rectangle(SrcImage, cv::Point(xmin, ymin), cv::Point(xmax, ymax), cv::Scalar(255, 0, 0), 2);
        putText(SrcImage, text1, cv::Point(xmin, ymin + 15), cv::FONT_HERSHEY_SIMPLEX, 0.7, cv::Scalar(0, 0, 255), 2);
    }

    imwrite("/zhangqian/workspaces1/TensorRT/yoloworld_trt_Cplusplus/images/result.jpg", SrcImage);

    printf("== obj: %d \n", int(float(YOLO.DetectiontRects_.size()) / 6.0));

```

## 特别说明

本示例只是用来测试流程，模型效果并不保证，且代码整理的布局合理性没有做过多的考虑。


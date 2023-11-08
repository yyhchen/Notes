

# OpenMP 学习笔记
---

环境配置
---

记录下 OpenMP学习过程

如果装了CLion就不用装MinGW了，CLion自带环境

<br>
在win11中，使用CLion配置 `OpenMP`环境，需要在`CMakeLists.txt` 中配置如下内容：

```cmake{.line-numbers}
# openMP 配置
FIND_PACKAGE(OpenMP REQUIRED)
if (OPENMP_FOUND)
    message("OPENMP FOUND")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif ()
```


---
## 目录


1. [并行程序设计(一)](./并行程序设计（一）.md)
2. [并行程序设计(二)](./并行程序设计（二）.md)







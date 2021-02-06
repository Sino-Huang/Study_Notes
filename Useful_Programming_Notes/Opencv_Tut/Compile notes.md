# Compile Notes

## unity build

When this property is set to true, the target source files will be combined into batches for faster compilation.  This is done by creating a (set of) unity sources which `#include` the original sources, then compiling these unity sources instead of the originals.  This is known as a *Unity* or *Jumbo* build



```cmake
cmake_minimum_required(VERSION 3.17)
project(OpenCV_repo)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_UNITY_BUILD TRUE)

add_executable(OpenCV_repo opencamera.cpp helloworldtest.cpp main.cpp functions.h)

```


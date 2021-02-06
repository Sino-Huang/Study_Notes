# Clion Cmake Setting



[TOC]



## add compiler flags in cmake

```cmake
set(GCC_COVERAGE_COMPILE_FLAGS "-Wall")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GCC_COVERAGE_COMPILE_FLAGS}" )
```



## cmake file for opencv

```cmake
# somehow it is easy to get opencv in ubuntu system
include(FindPkgConfig)
find_package(OpenCV REQUIRED)

# then if the package has been found, several variables will be set 
# you cna find the full list with descriptions in the OpenCVConfig.cmake file 

# We can show the message of the OpenCV lib info
message(STATUS "OpenCV library status:")
message(STATUS "	config: ${OpenCV_DIR}")
message(STATUS "	version: ${OpenCV_VERSION}")
message(STATUS "	libraries: ${OpenCV_LIBS}")
message(STATUS "	include path: ${OpenCV_INCLUDE_DIRS}")

# include dir, the variable is defined already in cmake config file
include_directories(${OpenCV_INCLUDE_DIRS})

# this command means let the target (the first input)
# link the library
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS} fmt)

# important, when you want to use fmt library newly installed in ubuntu, you need to manually add that..
# otherwise undefined reference 
```



## make the project more structural 

```cmake
#usually you can set SOURCE_FILES separately from the add_executable line 

set(SOURCE_FILES main.cpp)

add_executable(test ${SOURCE_FILES})
```







## static library 

```cmake
cmake_minimum_required(VERSION 3.17)
project(staticlibtest)

set(CMAKE_CXX_STANDARD 20)

set(SOURCE_FILES library.cpp library.h)

# add_library is the command for building static library
add_library(staticlibtest ${SOURCE_FILES})
```



### then linking the static library in another project 



#### Notes

include folder is for h header files, 

lib folder is for libraries 



Now we first of all need to create a new directory and add the created static library into this folder. for example 

we create a "TEMP" library, then we create TEMP folder, create include and lib in it. 

Then we add TEMP.a into lib folder, TEMP.h into include folder

```bash
$ tree staticlib/
staticlib/
├── include
│   └── library.h
└── lib
    └── libstaticlibtest.a

```



Then we create a new other project (executable project)

Then 

1. create another cmake folder (this will give the instruction of the lib file and include file from the external libs)

```
├── cmake
│   └── Findstaticlib.cmake

# syntax: FindXXX XXX is the lib name
```



```cmake
# this is the cmake/Findstaticlib.cmake file

# this set the path variable of the external library
set(FIND_STATICLIB_PATHS
        ~/CLionProjects/Template/staticlib)


# this find the path of header files
# IMPORTANT PATHS not PATH
find_path(STATICLIB_INCLUDE_DIR library.h
        PATH_SUFFIXES include
        PATHS ${FIND_STATICLIB_PATHS})
       
# this find the path of static lib file, and 
# just put the name of the static lib, do not add lib suffix and do not add .a 
find_library(STATICLIB_LIBRARY
        NAMES staticlibtest
        PATH_SUFFIXES lib
        PATHS ${FIND_STATICLIB_PATHS})
```

Now go back to the original project CMakeLists.txt file 

```cmake
cmake_minimum_required(VERSION 3.17)
project(test)

set(CMAKE_CXX_STANDARD 20)

# tell cmake where to look for cmake files
# CMAKE_CURRENT_LIST_DIR is the current DIR for CMakeLists.txt file
message(STATUS "cmake current list dir is ${CMAKE_CURRENT_LIST_DIR}") # -- cmake current list dir is /home/sukai/CLionProjects/Template/test

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

set(SOURCE_FILES main.cpp)
add_executable(test ${SOURCE_FILES})

# this is to allow the find package config function work
include(FindPkgConfig)

# then you find the package
find_package(staticlib REQUIRED)


message(STATUS "staticlib include dir is ${STATICLIB_INCLUDE_DIR}")
# include dir, the variable is defined already in cmake config file
include_directories(${STATICLIB_INCLUDE_DIR})

# this command means let the target (the first input)
# link the library
target_link_libraries(${PROJECT_NAME} ${STATICLIB_LIBRARY})
```



## Shared library 

similar to static lib

```cmake
cmake_minimum_required(VERSION 3.17)
project(sharedlib)

set(CMAKE_CXX_STANDARD 20)

add_library(sharedlib SHARED library.cpp library.h)
```

### Setting shared library for another project 

```cmake
# this is Findsharedlib.cmake config file

# set variable for PATHS
set(FIND_SHAREDLIB_PATHS
        ~/CLionProjects/Template/sharedlib)

# set the include dir configuration
find_path(SHAREDLIB_INCLUDE_DIR library.h
        PATH_SUFFIXES include
        PATHS ${FIND_SHAREDLIB_PATHS})

# set the lib configuration
# note that NAMES does not include suffix lib and the extension .so or .a
find_library(SHAREDLIB_LIBRARY
        NAMES sharedlib
        PATH_SUFFIXES lib
        PATHS ${FIND_SHAREDLIB_PATHS})
```

now move to the project's CMakeLists.txt 

```cmake
cmake_minimum_required(VERSION 3.17)
project(test)

set(CMAKE_CXX_STANDARD 20)

# tell cmake where to look for cmake files
# CMAKE_CURRENT_LIST_DIR is the current DIR for CMakeLists.txt file
message(STATUS "cmake current list dir is ${CMAKE_CURRENT_LIST_DIR}") # -- cmake current list dir is /home/sukai/CLionProjects/Template/test

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

set(SOURCE_FILES main.cpp)
add_executable(test ${SOURCE_FILES})

# this is to allow the find package config function work
include(FindPkgConfig)

# then you find the package
find_package(sharedlib REQUIRED)


message(STATUS "staticlib include dir is ${SHAREDLIB_INCLUDE_DIR}")
# include dir, the variable is defined already in cmake config file
include_directories(${SHAREDLIB_INCLUDE_DIR})

# this command means let the target (the first input)
# link the library
target_link_libraries(${PROJECT_NAME} ${SHAREDLIB_LIBRARY})
```

#### use ldd command to actually see the linked shared library of an executable 

```bash
$ ldd cmake-build-debug/test
        linux-vdso.so.1 (0x00007ffd92590000)
        libsharedlib.so => /home/sukai/CLionProjects/Template/sharedlib/lib/libsharedlib.so (0x00007f9bb3fd3000)
        libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f9bb3dd0000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9bb3be6000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f9bb3a97000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f9bb3fdf000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f9bb3a7c000)

```

## Add files in the current folder easily

```cmake
  # searches for all .cpp and .h files in the current directory and add them 
  # to the current project
  FILE(GLOB folder_source *.cpp)
  FILE(GLOB folder_header *.h)
  SOURCE_GROUP("Source Files" FILES ${folder_source})
  SOURCE_GROUP("Header Files" FILES ${folder_header})
  
  # create the project
  ADD_EXECUTABLE(your_project ${folder_source} ${folder_header})
```


```cmake
cmake_minimum_required(VERSION 3.20)

project(cmake_study)

# 定义SRC变量，变量的值为 add.cpp sub.cpp mult.cpp div.cpp main.cpp
# set(SRC add.cpp sub.cpp mult.cpp div.cpp main.cpp)

# 搜索源文件目录，不用上面自定义SRC变量然后一个一个的把cpp文件写入
# aux_source_directory(${PROJECT_SOURCE_DIR}/src SRC)

# 搜索源文件
file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)

# 指定生成可执行文件的目录
# set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/build)

# 指定自定义头文件的目录
include_directories(${PROJECT_SOURCE_DIR}/include)

# 指定C++编译器的版本 C++ 11版本
set(CMAKE_CXX_STANDARD 11)

# 指定生成可执行文件的名字，以及要编译的源文件目录
# add_executable(hello ${SRC})

# 指定库文件的生成目录
set(LIBRARY_OUTPUT_PATH ~/source/lib)

# calc生成库文件的名字 SHARED/STATIC:动态库/静态库 
# 动态库会生成 libcalc.so文件
add_library(calc SHARED ${SRC})
# 静态库会生成libcal.a文件
# add_library(calc STATIC ${SRC})
```

- 使用静态库文件

```cmake
cmake_minimum_required(VERSION 3.20)
project(cmake_study)
# 搜索源文件目录，不用上面自定义SRC变量然后一个一个的把cpp文件写入
# aux_source_directory(${PROJECT_SOURCE_DIR}/src SRC)
# 搜索源文件
file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 指定生成可执行文件的目录
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/build)
# 指定自定义头文件的目录
include_directories(${PROJECT_SOURCE_DIR}/include)

# 指定库文件的目录
link_directories(~/source/lib)
# 链接静态库
link_libraries(calc)

# 指定生成可执行文件的名字，以及要编译的源文件目录
add_executable(hello ${SRC})
```

- 静态库会在生成可执行程序的链接阶段打包到可执行程序中，所以可执行程序启动，静态库就要加载到内存中了
- 动态库在生成可执行程序的链接阶段不会被打包到可执行程序中，当可执行程序被启动并且调用了动态库中的函数的时候，动态库才会被加载到内存
- 使用动态库文件

```cmake
cmake_minimum_required(VERSION 3.20)
project(cmake_study)
# 搜索源文件
file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 指定生成可执行文件的目录
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/build)
# 指定自定义头文件的目录
include_directories(${PROJECT_SOURCE_DIR}/include)
# 指定库文件的目录
link_directories(~/source/lib)
# 指定生成可执行文件的名字，以及要编译的源文件目录
add_executable(hello ${SRC})

# target_link_libraries命令要放在add_executable命令之后，先生成可执行文件
# hello可执行文件的名字 calc动态库的文件名
target_link_libraries(hello calc)
```

- message打印日志

```cmake
# message打印日志
# (无)：重要消息
# STATUS: 非重要消息
# WARNING: CMake警告，会继续执行
# AUHTOR_WARNING: CMake警告(dev)，会继续执行
# SEND_ERROR: CMake错误，继续执行，但是会跳过生成的步骤
# FATAL_ERROR: CMake错误，种植所有处理过程
message("hello ${PROJECT_SOURCE_DIR}")
message(STATUS "hello ${PROJECT_SOURCE_DIR}")
message(WARNING "hello ${PROJECT_SOURCE_DIR}")
message(AUHTOR_WARNING "hello ${PROJECT_SOURCE_DIR}")
message(SEND_ERROR "hello ${PROJECT_SOURCE_DIR}")
```


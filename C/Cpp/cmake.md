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
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/build)

# 指定自定义头文件的目录
include_directories(${PROJECT_SOURCE_DIR}/include)

# 指定C++编译器的版本 C++ 11版本
set(CMAKE_CXX_STANDARD 11)

# 指定生成可执行文件的名字，以及要编译的源文件目录
add_executable(hello ${SRC})
```


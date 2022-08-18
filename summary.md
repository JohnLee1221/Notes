#       CMakeLists.txt

## 常用命令

### CMAKE_MINIMUM_REQUIRED()

\#指定cmake最小版本要求

CMAKE_MINIMUM_REQUIRED(VERSION 3.4.1)

### ADD_COMPILE_OPTIONS()

\#添加编译选项

ADD_COMPILE_OPTIONS(-std=c++11 -Wall -Won-error=old-style-cast -Wno-error=useless-cast -Wold-style-cast -Wuseless-cast -Wno-error=useless-cast)

### PROJECT()

\#设置项目名称

PROJECT(DEMO)

自动引入两个变量：DEMO_SOURCE_DIR   DEMO_BINARY_DIR，等价cmake自动定义的两个变量：PROJECT_SOURCE_DIR    PROJECT_BINARY_DIR

### MESSAGE()

\#输出消息

MESSAGE(STATUS "THIS IS A DEMO PROGRAM")

### OPTION()

\#给变量赋值

OPTION(VAR_NAME "comment" VAR_VALUE)		#comment是变量注释

### SET()

\#设置变量

SET(SRC_LIST demo.cpp)

### ADD_LIBRARY()

ADD_LIBRARY(common1 STATIC ${PROJECT_SOURCE_DIR}/src/common1.cpp)#指定生成静态库--libcommon1.a

ADD_LIBRARY(common2 SHARED ${PROJECT_SOURCE_DIR}/src/common2.cpp)#指定生成动态库--libcommon2.so

### SET_TARGET_PROPERTIES()

\#对库的名称重新定义

SET_TARGET_PROPERTIES(common1 PROPERTIES OUTPUT_NAME "common")

SET_TARGET_PROPERTIES(common2 PROPERTIES OUTPUT_NAME "common")

SET(LIBRARY_OUTPUT_PATH ${SRC_LIST}/lib)#设置库文件生成路径

### ADD_EXECUTABLE()

\#指定可执行文件的源文件

ADD_EXECUTABLE(demo ${SRC_LIST})

### INCLUDE_DIRECTOIES()

\#定义头文件的包含目录

INCLUDE_DIRECTOIES(${PROJECT_SOURCE_DIR}/include}

SET(EXECUTABLE_OUTPUT_PATH ${SRC_LIST}/bin)#设置执行文件生成路径

### FIND_LIBRARY()

\#指定库目录

FIND_LIBRARY(DEMO_LIB common HINTS ${PROJECT_SOURCE_DIR}/lib)#在指定目录下查找指定库，并把绝对路径放置到变量里

### TARGET_LINK_LIBRARIES()

TARGET_LINK_LIBRARIES(demo ${DEMO_LIB})#把目标文件与库文件进行链接

### AUX_SOURCE_DIRECTORY()

AUX_SOURCE_DIRECTORY(${PROJECT_SOOURCE_DIR}/ SRC_LIST}#搜索项目主目录下的所有源文件

ADD_EXECUTABLE(demo ${SRC_LIST})

### FIND_PACKAGE()

```cmake
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...]
             [NO_POLICY_SCOPE])
```

```cmake
# 查找CURL库
find_package(CURL)
add_executable(curltest curltest.cc)
if(CURL_FOUND)
    target_include_directories(clib PRIVATE ${CURL_INCLUDE_DIR})
    target_link_libraries(curltest ${CURL_LIBRARY})
else(CURL_FOUND)
    message(FATAL_ERROR ”CURL library not found”)
endif(CURL_FOUND)
```

对于系统预定义的 `Find<LibaryName>.cmake` 模块，使用方法一般如上例所示。

每一个模块都会定义以下几个变量 

```
<LibaryName>_FOUND
<LibaryName>_INCLUDE_DIR or <LibaryName>_INCLUDES
<LibaryName>_LIBRARY or <LibaryName>_LIBRARIES
```

你可以通过`<LibaryName>_FOUND` 来判断模块是否被找到，如果没有找到，按照工程的需要关闭 某些特性、给出提醒或者中止编译，上面的例子就是报出致命错误并终止构建。 如果`<LibaryName>_FOUND` 为真，则将`<LibaryName>_INCLUDE_DIR` 加入 INCLUDE_DIRECTORIES





```cmake
cmake_minimum_required(VERSION 3.5)
project(arm_core)

set(BUILD_TEST TRUE)
set(LIB_NAME libarmcore)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(yaml-cpp REQUIRED)
find_package(spdlog REQUIRED)

include_directories(
  include/
)

add_library(libarmcore SHARED
   #pose
   src/pose/Pose.cpp
)

target_link_libraries(libarmcore
  jsoncpp
  yaml-cpp
  spdlog
)

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  TARGETS libarmcore
  EXPORT libarmcore
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

# add lib.config
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
        libarmcoreConfig.cmake
        VERSION 1.0
        COMPATIBILITY AnyNewerVersion 
        )

install(EXPORT libarmcore
        FILE libarmcoreConfig.cmake
        DESTINATION lib/cmake/libarmcore
        )
```

------





## 预定义变量

### PROJECT_SOURCE_DIR

工程的根目录

### PROJECT_BINARY_DIR

运行 cmake 命令的目录，通常是 ${PROJECT_SOURCE_DIR}/build

### PROJECT_NAME

返回通过 project 命令定义的项目名称

### CMAKE_CURRENT_SOURCE_DIR

当前处理的 CMakeLists.txt 所在的路径

### CMAKE_CURRENT_BINARY_DIR

target 编译目录

### CMAKE_CURRENT_LIST_DIR

CMakeLists.txt 的完整路径

### CMAKE_CURRENT_LIST_LINE

当前所在的行

### CMAKE_MODULE_PATH

定义自己的 cmake 模块所在的路径，SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)，然后可以用INCLUDE命令来调用自己的模块

### EXECUTABLE_OUTPUT_PATH

重新定义目标二进制可执行文件的存放位置

### LIBRARY_OUTPUT_PATH

重新定义目标链接库文件的存放位置



## 条件控制

### if…elseif…else…endif

#### 逻辑判断和比较

if (expression)：expression 不为空（0,N,NO,OFF,FALSE,NOTFOUND）时为真

if (not exp)：与上面相反

if (var1 AND var2)

if (var1 OR var2)

if (COMMAND cmd)：如果 cmd 确实是命令并可调用为真

if (EXISTS dir) if (EXISTS file)：如果目录或文件存在为真

if (file1 IS_NEWER_THAN file2)：当 file1 比 file2 新，或 file1/file2 中有一个不存在时为真，文件名需使用全路径

if (IS_DIRECTORY dir)：当 dir 是目录时为真

if (DEFINED var)：如果变量被定义为真

if (var MATCHES regex)：给定的变量或者字符串能够匹配正则表达式 regex 时为真，此处 var 可以用 var 名，也可以用 ${var}

if (string MATCHES regex)

#### 数字比较

if (variable LESS number)：LESS 小于

if (string LESS number)

if (variable GREATER number)：GREATER 大于

if (string GREATER number)

if (variable EQUAL number)：EQUAL 等于

if (string EQUAL number)

#### 字母表顺序比较

if (variable STRLESS string)

if (string STRLESS string)

if (variable STRGREATER string)

if (string STRGREATER string)

if (variable STREQUAL string)

if (string STREQUAL string)

#### 示例

if(MSVC)
set(LINK_LIBS common)
else()
set(boost_thread boost_log.a boost_system.a)
endif()
target_link_libraries(demo ${LINK_LIBS})

或者

if(UNIX)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fpermissive -g")
else()
add_definitions(-D_SCL_SECURE_NO_WARNINGS
D_CRT_SECURE_NO_WARNINGS
-D_WIN32_WINNT=0x601
-D_WINSOCK_DEPRECATED_NO_WARNINGS)
endif()

if(${CMAKE_BUILD_TYPE} MATCHES "debug")
...
else()
...
endif()

### while…endwhile

while(condition)

...

endwhile()

### foreach…endforeach

foreach(loop_var RANGE start stop [step])

...

endforeach(loop_var)



# cmake

## CMAKE_PREFIX_PATH

`CMAKE_PREFIX_PATH`是一个分号分隔的路径列表，用来指明软件/库安装路径前缀，以供`find_package()`，`find_program()`，`find_library()`，`find_file()`和`find_path()`命令搜索使用，这样就方便搜索可执行文件、头文件及库文件等。初始为空，由用户设定。



cmake .. -DCMAKE_PREFIX_PATH=/usr/local/lib



或者在CMakeLists.txt中：

list(APPEND CMAKE_PREFIX_PATH "/usr/local/lib")



## CMAKE_MODULE_PATH

`CMAKE_MODULE_PATH`是以分号分隔的列表，供`include()`或 `find_package()`使用。初始为空，由用户设定。



## CMAKE_INSTALL_PREFIX

```bash
cmake -DCMAKE_INSTALL_PREFIX=/usr ..
```



修改cmake文件，加入：

```sql
SET(CMAKE_INSTALL_PREFIX /usr/local)
INSTALL(TARGETS demo DESTINATION bin)		#将demo安装在/usr/local/bin目录下
```



# package.xml

## 定义

package.xml文件定义了package的属性。（例如：包名，版本号，作者，一来等等），相当于一个包的自我描述。

解析catkin_make的参数同时遍历整个工作空间把所有的有package.xml的文件夹添加进软件包列表里面。对每个软件包执行add_subdirectory。

解析package.xml文件，载入对应的参数。根据依赖参数，载入对应的软件包参数。根据载入参数生成当前软件包的配置文件。

## 释义

​    b8u yuytvh m opgc ytyt gd gceuyx yst xety                                    

## 模板

```
<?xml version="1.0"?>
<packageformat="2">
<name>test</name>
<version>0.0.0</version>
<description>The test package</description>

<!-- One maintainer tag required, multiple allowed, one person per tag -->
<!-- Example:  -->
<!-- <maintainer email="jane.doe@example.com">Jane Doe</maintainer> -->

<maintaineremail="lab212@todo.todo">lab212</maintainer>

<!-- One license tag required, multiple allowed, one license per tag -->
<!-- Commonly used license strings: -->
<!--   BSD, MIT, Boost Software License, GPLv2, GPLv3, LGPLv2.1, LGPLv3 -->

<license>TODO</license>
<!-- Url tags are optional, but multiple are allowed, one per tag -->
<!-- Optional attribute type can be: website, bugtracker, or repository -->
<!-- Example: -->
<!-- <url type="website">http://wiki.ros.org/test</url> -->

<!-- Author tags are optional, multiple are allowed, one per tag -->
<!-- Authors do not have to be maintainers, but could be -->
<!-- Example: -->
<!-- <author email="jane.doe@example.com">Jane Doe</author> -->

<!-- The *depend tags are used to specify dependencies -->
<!-- Dependencies can be catkin packages or system dependencies -->
<!-- Examples: -->
<!-- Use depend as a shortcut for packages that are both build and exec dependencies -->

<!--   <depend>roscpp</depend> -->
<!--   Note that this is equivalent to the following: -->
<!--   <build_depend>roscpp</build_depend> -->
<!--   <exec_depend>roscpp</exec_depend> -->
<!-- Use build_depend for packages you need at compile time: -->
<!--   <build_depend>message_generation</build_depend> -->
<!-- Use build_export_depend for packages you need in order to build against this package: -->
<!--   <build_export_depend>message_generation</build_export_depend> -->
<!-- Use buildtool_depend for build tool packages: -->
<!--   <buildtool_depend>catkin</buildtool_depend> -->
<!-- Use exec_depend for packages you need at runtime: -->
<!--   <exec_depend>message_runtime</exec_depend> -->
<!-- Use test_depend for packages you need only for testing: -->
<!--   <test_depend>gtest</test_depend> -->
<!-- Use doc_depend for packages you need only for building documentation: -->
<!--   <doc_depend>doxygen</doc_depend> -->
<buildtool_depend>catkin</buildtool_depend>
<build_depend>roscpp</build_depend>
<build_depend>rospy</build_depend>
<build_depend>std_msgs</build_depend>
<build_export_depend>roscpp</build_export_depend>
<build_export_depend>rospy</build_export_depend>
<build_export_depend>std_msgs</build_export_depend>
<exec_depend>roscpp</exec_depend>
<exec_depend>rospy</exec_depend>
<exec_depend>std_msgs</exec_depend>
<!-- The export tag contains other, unspecified, tags -->
<export>
<!-- Other tools can request additional information be placed here -->
</export>
</package>
```











# git

git init

git add 

git commit -m ""

git 









# rclcpp

## create_wall_timer()

```
typename rclcpp::WallTimer<CallbackT>::SharedPtr
create_wall_timer(
  std::chrono::duration<DurationRepT, DurationT> period,					//设置timer的时间间隔
  CallbackT callback,														//回调函数
  rclcpp::CallbackGroup::SharedPtr group,									//执行此他的回调
  node_interfaces::NodeBaseInterface * node_base,
  node_interfaces::NodeTimersInterface * node_timers)
```

